# Authentication and Authorization Implementation Patterns

This document details the key implementation patterns used in the Super-Deals application for authentication and authorization. It provides practical guidance on how these patterns are implemented and how they can be extended for future requirements.

## Table of Contents

1. [Direct S3 Upload Pattern](#direct-s3-upload-pattern)
2. [OAuth Scope Enforcement Pattern](#oauth-scope-enforcement-pattern)
3. [Role-Based Access Control Pattern](#role-based-access-control-pattern)
4. [API Gateway Authorization Pattern](#api-gateway-authorization-pattern)
5. [Multi-Step Form with Auth State Pattern](#multi-step-form-with-auth-state-pattern)
6. [Custom Email Templates Pattern](#custom-email-templates-pattern)
7. [Extending the Authorization System](#extending-the-authorization-system)

## Direct S3 Upload Pattern

### Overview

The Direct S3 Upload pattern allows client applications to upload files directly to S3 without proxying through API Gateway or Lambda functions. This pattern is used for uploading deal images and other binary content.

### Implementation

#### 1. IAM Role Configuration

The merchant IAM role is configured with specific S3 permissions:

```javascript
// In backend/lib/permissions/stack.js
const merchantS3Policy = new PolicyStatement({
  effect: Effect.ALLOW,
  actions: ["s3:PutObject"],
  resources: [`${storage.s3Bucket.bucketArn}/merchants/*`],
});

iam.roles.merchant.addToPolicy(merchantS3Policy);
```

#### 2. Frontend Credential Acquisition

The frontend acquires temporary AWS credentials:

```javascript
// In +page.server.js
export async function load({ cookies, locals }) {
  // Verify authentication and authorization
  if (!Utils.auth.isMerchant(session) || !Utils.auth.canWriteDeals(session)) {
    throw error(403, 'Insufficient permissions to create deals');
  }

  // Get credentials from Amplify
  const { credentials } = await fetchAuthSession();
  
  return {
    userId: user.userId,
    credentials,
    dealId: KSUID.randomSync(new Date()).string,
    s3BucketName: backendOutputs.S3BucketName
  };
}
```

#### 3. Frontend File Upload

The frontend uploads files directly to S3:

```javascript
// In +page.svelte
const s3Client = new S3Client({
  region: 'us-east-1',
  credentials: {
    accessKeyId: credentials.accessKeyId,
    secretAccessKey: credentials.secretAccessKey,
    sessionToken: credentials.sessionToken
  }
});

// Generate a unique key for the file
const fileKey = `merchants/${userId}/deals/DEAL-${dealId}/logo/${file.name}`;

await s3Client.send(new PutObjectCommand({
  Bucket: s3BucketName,
  Key: fileKey,
  Body: file,
  ContentType: file.type
}));

// Send only the file reference to the API
formData.delete('logo');
formData.set('logoFileKey', fileKey);
```

#### 4. Backend Reference Storage

The backend stores only the file reference:

```javascript
// In Lambda handler
const { logoFileKey, ...dealData } = event.body;

// Store deal with file reference
const deal = {
  id: dealId,
  merchantId: userId,
  logoUrl: logoFileKey,
  ...dealData,
  createdAt: new Date().toISOString()
};

await dynamoDb.put({
  TableName: process.env.DEALS_TABLE,
  Item: deal
});
```

### Benefits

- **Performance**: Bypasses API Gateway limits for large files
- **Cost Efficiency**: Reduces Lambda execution time and data transfer costs
- **Scalability**: Offloads file handling to S3
- **Security**: Uses temporary credentials with limited permissions

### Best Practices

1. **Always validate files** on the client side before upload
2. **Use path-based access control** with user IDs in the path
3. **Generate unique file keys** to prevent overwrites
4. **Set appropriate content types** for browser rendering
5. **Implement CORS configuration** on the S3 bucket

## OAuth Scope Enforcement Pattern

### Overview

The OAuth Scope Enforcement pattern uses OAuth 2.0 scopes to control access to API endpoints. This pattern is implemented through Cognito Resource Servers and API Gateway authorizers.

### Implementation

#### 1. Resource Server Definition

Resource servers define available scopes:

```javascript
// In backend/lib/auth/user-pool/resource-servers/deals/construct.js
this.scopes = [
  new ResourceServerScope({
    scopeName: 'read',
    scopeDescription: 'Read access to deals'
  }),
  new ResourceServerScope({
    scopeName: 'write',
    scopeDescription: 'Write access to deals'
  }),
  new ResourceServerScope({
    scopeName: 'delete',
    scopeDescription: 'Delete access to deals'
  })
];

this.resourceServer = new UserPoolResourceServer(this, 'ResourceServer', {
  userPool,
  identifier: `deals-${envName}`,
  scopes: this.scopes,
});
```

#### 2. Authorization Options

Authorization options are created for different operations:

```javascript
// In backend/lib/permissions/oauth-permissions/deals/construct.js
getAuthOptions(authorizerId) {
  const scopeNames = this.getScopeNames();
  const baseAuth = {
    authorizationType: 'COGNITO_USER_POOLS',
    authorizer: { authorizerId }
  };

  return {
    readDealsAuth: {
      ...baseAuth,
      authorizationScopes: [scopeNames.find(scope => scope === 'deals:read')]
    },
    writeDealsAuth: {
      ...baseAuth,
      authorizationScopes: [scopeNames.find(scope => scope === 'deals:write')]
    },
    deleteDealsAuth: {
      ...baseAuth,
      authorizationScopes: [scopeNames.find(scope => scope === 'deals:delete')]
    }
  };
}
```

#### 3. API Method Authorization

API methods are configured with required scopes:

```javascript
// In backend/lib/api/http/endpoints/merchants/deals/create/construct.js
dealsResource.addMethod(
  "POST",
  new LambdaIntegration(lambda.merchants.deals.create.function),
  {
    operationName: "CreateDeal",
    requestValidator,
    requestModels: {
      "application/json": model,
    },
    ...http.optionsWithAuth.writeDealsAuth, // Use write scope for deal creation
  }
);
```

#### 4. Token Validation

API Gateway validates tokens and scopes:

```javascript
// In backend/lib/api/http/authorization/construct.js
this.authorizer = new CognitoUserPoolsAuthorizer(this, "CognitoUserPoolsAuthorizer", {
  cognitoUserPools: [auth.userPool.pool],
  identitySource: "method.request.header.Authorization",
});
```

### Benefits

- **Fine-grained Access Control**: Control access at the operation level
- **Declarative Authorization**: Define permissions in infrastructure code
- **Standardized Approach**: Follow OAuth 2.0 standards
- **Reduced Lambda Code**: Authorization happens before Lambda execution

### Best Practices

1. **Use descriptive scope names** that clearly indicate the permission
2. **Implement scope hierarchies** where appropriate (e.g., admin includes all scopes)
3. **Validate scopes in Lambda functions** as a defense-in-depth measure
4. **Document available scopes** for developers and integrators

## Role-Based Access Control Pattern

### Overview

The Role-Based Access Control (RBAC) pattern assigns permissions based on user roles. In our application, this is implemented through Cognito User Groups and IAM roles.

### Implementation

#### 1. Cognito User Groups

User groups are defined in Cognito:

```javascript
// In backend/lib/auth/user-groups/stack.js
new CfnUserPoolGroup(this, 'MerchantsGroup', {
  userPoolId: userPool.pool.userPoolId,
  groupName: 'Merchants',
  description: 'Group for merchant users',
  precedence: 1
});
```

#### 2. IAM Role Mapping

IAM roles are mapped to Cognito groups:

```javascript
// In backend/lib/iam/roles/stack.js
new CfnIdentityPoolRoleAttachment(this, 'IdentityPoolRoleAttachment', {
  identityPoolId: auth.identityPool.pool.ref,
  roles: {
    authenticated: this.authenticated.roleArn,
    unauthenticated: this.unAuthenticated.roleArn,
  },
  roleMappings: {
    [roleMappingsKey.ref]: {
      type: 'Rules',
      ambiguousRoleResolution: 'Deny',
      identityProvider: `${auth.userPool.pool.userPoolProviderName}:${auth.userPool.poolClient.userPoolClientId}`,
      rulesConfiguration: {
        rules: [
          {
            claim: 'cognito:groups',
            matchType: 'Contains',
            value: 'Merchants',
            roleArn: this.merchant.roleArn
          }
        ]
      }
    }
  }
});
```

#### 3. Group Assignment

Users are assigned to groups during registration:

```javascript
// In backend/src/lambda/merchants/account/sign-up/handler.js
await cognitoClient.send(
  new AdminAddUserToGroupCommand({
    UserPoolId: userPoolId,
    Username: username,
    GroupName: 'Merchants'
  })
);
```

#### 4. Role-Based Permissions

Permissions are attached to roles:

```javascript
// In backend/lib/permissions/stack.js
const merchantS3Policy = new PolicyStatement({
  effect: Effect.ALLOW,
  actions: ["s3:PutObject"],
  resources: [`${storage.s3Bucket.bucketArn}/merchants/*`],
});

iam.roles.merchant.addToPolicy(merchantS3Policy);
```

### Benefits

- **Simplified Permission Management**: Manage permissions by role, not individual users
- **Hierarchical Structure**: Organize permissions in a logical hierarchy
- **Scalability**: Easily add new users to existing roles
- **Auditability**: Clearly see which roles have which permissions

### Best Practices

1. **Follow principle of least privilege** when assigning permissions to roles
2. **Create role hierarchies** that match your organization structure
3. **Document role responsibilities** for clarity
4. **Regularly audit role permissions** to ensure they remain appropriate

## API Gateway Authorization Pattern

### Overview

The API Gateway Authorization pattern secures API endpoints using Cognito User Pools as an authorizer. This pattern validates tokens and enforces scope-based permissions.

### Implementation

#### 1. Authorizer Configuration

A Cognito authorizer is created:

```javascript
// In backend/lib/api/http/authorization/construct.js
this.authorizer = new CognitoUserPoolsAuthorizer(this, "CognitoUserPoolsAuthorizer", {
  cognitoUserPools: [auth.userPool.pool],
  identitySource: "method.request.header.Authorization",
});
this.authorizer._attachToApi(restApi);
```

#### 2. Method Authorization

API methods are configured with the authorizer:

```javascript
// In backend/lib/api/http/endpoints/merchants/deals/create/construct.js
dealsResource.addMethod(
  "POST",
  new LambdaIntegration(lambda.merchants.deals.create.function),
  {
    operationName: "CreateDeal",
    requestValidator,
    requestModels: {
      "application/json": model,
    },
    ...http.optionsWithAuth.writeDealsAuth,
  }
);
```

#### 3. Token Validation

API Gateway validates tokens automatically:
- Verifies token signature
- Checks token expiration
- Validates required scopes
- Extracts claims for Lambda context

#### 4. Lambda Authorization

Lambda functions can perform additional authorization:

```javascript
// In Lambda handler
const userId = event.requestContext.authorizer.claims.sub;
const merchantId = event.pathParameters.userId;

// Ensure user can only access their own resources
if (userId !== merchantId) {
  return {
    statusCode: 403,
    body: JSON.stringify({
      message: "Forbidden: Cannot access another merchant's resources"
    })
  };
}
```

### Benefits

- **Token Validation**: Automatic validation of JWT tokens
- **Scope Enforcement**: Declarative scope requirements
- **Reduced Boilerplate**: No token validation code in Lambda functions
- **Standardized Approach**: Consistent authorization across endpoints

### Best Practices

1. **Always use HTTPS** for API endpoints
2. **Implement defense-in-depth** with additional checks in Lambda functions
3. **Use request validators** to ensure request format is valid
4. **Configure appropriate caching** for authorizers to improve performance

## Multi-Step Form with Auth State Pattern

### Overview

The Multi-Step Form with Auth State pattern manages user state during multi-step registration processes. This pattern is used in the merchant sign-up flow.

### Implementation

#### 1. State Storage

Form state is stored in cookies:

```javascript
// In +page.server.js
cookies.set(`signup_${key}`, value, {
  path: '/',
  maxAge: 60 * 30, // 30 minutes
  httpOnly: false
});
```

#### 2. Step Validation

Each step is validated independently:

```javascript
// In +page.server.js
const formData = await request.formData();
const step = formData.get('step');

if (step === '1') {
  const result = step1Schema.safeParse({
    businessName: formData.get('businessName'),
    email: formData.get('email'),
    password: formData.get('password'),
    confirmPassword: formData.get('confirmPassword')
  });
  
  if (!result.success) {
    return fail(400, { errors: result.error.flatten() });
  }
  
  // Store validated data in cookies
  Object.entries(result.data).forEach(([key, value]) => {
    cookies.set(`signup_${key}`, value, {
      path: '/',
      maxAge: 60 * 30,
      httpOnly: false
    });
  });
}
```

#### 3. Final Submission

On final step, all data is combined and submitted:

```javascript
// In +page.server.js
if (step === '3') {
  // Validate step 3
  if (!result.success) {
    return fail(400, { errors: result.error.flatten() });
  }
  
  // Combine all steps from cookies
  const signupData = {
    businessName: cookies.get('signup_businessName'),
    email: cookies.get('signup_email'),
    password: cookies.get('signup_password'),
    // Add step 2 and 3 data
    ...result.data
  };
  
  // Submit to API
  const response = await merchantService.signUp(signupData);
  
  if (response.success) {
    // Clear cookies and redirect to verification page
    Object.keys(signupData).forEach(key => {
      cookies.delete(`signup_${key}`);
    });
    
    cookies.set('pendingConfirmation', signupData.email, {
      path: '/',
      maxAge: 60 * 30,
      httpOnly: true
    });
    
    cookies.set('userType', 'merchant', {
      path: '/',
      maxAge: 60 * 30,
      httpOnly: true
    });
    
    return { success: true, redirect: '/auth/verification-sent' };
  }
}
```

### Benefits

- **Improved User Experience**: Break complex forms into manageable steps
- **Progressive Validation**: Validate each step independently
- **State Persistence**: Maintain state across page refreshes
- **Security**: Validate on both client and server sides

### Best Practices

1. **Set appropriate cookie expiration** to balance security and user experience
2. **Validate on both client and server** for best user experience and security
3. **Clear sensitive data** after successful submission
4. **Use HTTPS-only cookies** for sensitive information
5. **Implement CSRF protection** for form submissions

## Custom Email Templates Pattern

### Overview

The Custom Email Templates pattern customizes Cognito email templates based on user type. This pattern is used to provide different verification emails for merchants and customers.

### Implementation

#### 1. Lambda Trigger Configuration

A custom message Lambda trigger is configured:

```javascript
// In backend/lib/auth/user-pool/stack.js
lambdaTriggers: {
  customMessage: customMessageLambda.function,
},
```

#### 2. User Type Detection

The Lambda detects user type from attributes:

```javascript
// In backend/src/lambda/auth/custom-message/handler.js
// Determine if the user is a merchant
const userAttributes = event.request.userAttributes;
const isMerchant = userAttributes['custom:userGroup'] === 'merchant';
```

#### 3. Template Selection

Different templates are used based on user type:

```javascript
// In backend/src/lambda/auth/custom-message/handler.js
if (isMerchant) {
  event.response.emailSubject = 'Verify your Super Deals Merchant Account';
  event.response.emailMessage = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        /* Merchant-specific styling */
      </style>
    </head>
    <body>
      <h1>Welcome to Super Deals Merchant Portal!</h1>
      <p>Thank you for registering as a merchant. Please verify your email using the code below:</p>
      <div class="verification-code">${event.request.codeParameter}</div>
      <!-- Merchant-specific content -->
    </body>
    </html>
  `;
} else {
  event.response.emailSubject = 'Verify your Super Deals Account';
  event.response.emailMessage = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        /* Customer-specific styling */
      </style>
    </head>
    <body>
      <h1>Welcome to Super Deals!</h1>
      <p>Thank you for registering. Please verify your email using the code below:</p>
      <div class="verification-code">${event.request.codeParameter}</div>
      <!-- Customer-specific content -->
    </body>
    </html>
  `;
}
```

### Benefits

- **Personalized Experience**: Tailored messaging for different user types
- **Branded Communication**: Consistent branding in all communications
- **HTML Formatting**: Rich email formatting with CSS
- **Dynamic Content**: Include user-specific information in emails

### Best Practices

1. **Test emails in multiple clients** to ensure compatibility
2. **Use inline CSS** for maximum compatibility
3. **Include plain text alternatives** for accessibility
4. **Keep templates maintainable** by using a modular approach
5. **Include all necessary information** in the email body

## Extending the Authorization System

### Adding New User Types

To add new user types (e.g., Admins, Customers):

1. **Create User Group**:
   ```javascript
   new CfnUserPoolGroup(this, 'AdminsGroup', {
     userPoolId: userPool.pool.userPoolId,
     groupName: 'Admins',
     description: 'Group for admin users',
     precedence: 0 // Lower precedence = higher priority
   });
   ```

2. **Create IAM Role**:
   ```javascript
   this.admin = new Role(this, 'CognitoAdminRole', {
     assumedBy: new FederatedPrincipal('cognito-identity.amazonaws.com', {
       StringEquals: {
         'cognito-identity.amazonaws.com:aud': auth.identityPool.pool.ref
       },
       'ForAnyValue:StringLike': {
         'cognito-identity.amazonaws.com:amr': 'authenticated'
       }
     },
       'sts:AssumeRoleWithWebIdentity'
     )
   });
   ```

3. **Add Role Mapping**:
   ```javascript
   rules: [
     {
       claim: 'cognito:groups',
       matchType: 'Contains',
       value: 'Admins',
       roleArn: this.admin.roleArn
     },
     // Existing rules...
   ]
   ```

4. **Attach Permissions**:
   ```javascript
   const adminPolicy = new PolicyStatement({
     effect: Effect.ALLOW,
     actions: [
       "s3:PutObject",
       "s3:GetObject",
       "s3:ListBucket"
     ],
     resources: [
       `${storage.s3Bucket.bucketArn}/*`,
     ],
   });
   
   iam.roles.admin.addToPolicy(adminPolicy);
   ```

5. **Update Email Templates**:
   ```javascript
   if (userAttributes['custom:userGroup'] === 'admin') {
     // Admin-specific email template
   } else if (isMerchant) {
     // Merchant-specific email template
   } else {
     // Customer-specific email template
   }
   ```

### Adding New Resource Servers and Scopes

To add new protected APIs (e.g., Orders API):

1. **Create Resource Server**:
   ```javascript
   // In a new file: backend/lib/auth/user-pool/resource-servers/orders/construct.js
   this.scopes = [
     new ResourceServerScope({
       scopeName: 'read',
       scopeDescription: 'Read access to orders'
     }),
     new ResourceServerScope({
       scopeName: 'write',
       scopeDescription: 'Write access to orders'
     }),
     // More scopes...
   ];
   
   this.resourceServer = new UserPoolResourceServer(this, 'ResourceServer', {
     userPool,
     identifier: `orders-${envName}`,
     scopes: this.scopes,
   });
   ```

2. **Create OAuth Permissions**:
   ```javascript
   // In a new file: backend/lib/permissions/oauth-permissions/orders/construct.js
   getAuthOptions(authorizerId) {
     const scopeNames = this.getScopeNames();
     const baseAuth = {
       authorizationType: 'COGNITO_USER_POOLS',
       authorizer: { authorizerId }
     };
   
     return {
       readOrdersAuth: {
         ...baseAuth,
         authorizationScopes: [scopeNames.find(scope => scope === 'orders:read')]
       },
       // More auth options...
     };
   }
   ```

3. **Update Main OAuth Construct**:
   ```javascript
   // In backend/lib/permissions/oauth-permissions/construct.js
   this.orders = new OrdersOAuthPermissionsConstruct(this, "Orders", {
     resourceServer: auth.userPool.resourceServers.orders.resourceServer,
   });
   ```

4. **Apply to API Endpoints**:
   ```javascript
   ordersResource.addMethod(
     "GET",
     new LambdaIntegration(lambda.merchants.orders.list.function),
     {
       ...http.optionsWithAuth.readOrdersAuth,
     }
   );
   ```

### Adding New API Resources

To add new API resources (e.g., Orders API):

1. **Create API Resource**:
   ```javascript
   // In a new file: backend/lib/api/http/endpoints/merchants/orders/construct.js
   const ordersResource = merchantsResource.addResource('orders');
   
   // List orders endpoint
   ordersResource.addMethod(
     "GET",
     new LambdaIntegration(lambda.merchants.orders.list.function),
     {
       operationName: "ListOrders",
       ...http.optionsWithAuth.readOrdersAuth,
     }
   );
   
   // Get order endpoint
   const orderResource = ordersResource.addResource('{orderId}');
   orderResource.addMethod(
     "GET",
     new LambdaIntegration(lambda.merchants.orders.get.function),
     {
       operationName: "GetOrder",
       ...http.optionsWithAuth.readOrdersAuth,
     }
   );
   ```

2. **Update Endpoints Construct**:
   ```javascript
   // In backend/lib/api/http/endpoints/construct.js
   new MerchantsOrdersConstruct(this, "MerchantsOrders", {
     lambda,
     http,
     merchantsResource
   });
   ```

### Best Practices for Extension

1. **Follow existing patterns** for consistency
2. **Document new components** thoroughly
3. **Update authorization documentation** when adding new permissions
4. **Test authorization rules** comprehensively
5. **Maintain separation of concerns** between different authorization mechanisms
6. **Use descriptive names** for roles, groups, and scopes
