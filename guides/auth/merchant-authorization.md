# Merchant Authorization

This document details the authorization mechanisms specific to merchant users in the Super-Deals application. It covers the complete flow from sign-up to resource access, focusing on the permissions and access patterns available to merchants.

## Table of Contents

1. [Merchant User Journey](#merchant-user-journey)
2. [Merchant-Specific Permissions](#merchant-specific-permissions)
3. [Resource Access Patterns](#resource-access-patterns)
4. [API Scopes for Merchants](#api-scopes-for-merchants)
5. [Deal Creation Flow](#deal-creation-flow)
6. [Security Considerations](#security-considerations)

## Merchant User Journey

### Sign-Up and Verification Flow

The merchant sign-up process follows these steps:

1. **Registration**:
   - Merchant completes the multi-step registration form
   - Form collects account, business, and contact information
   - Data is validated on both client and server sides
   - Request is sent to `/merchants/account/signup` endpoint

2. **Account Creation**:
   - Backend creates a Cognito user with merchant attributes
   - User is added to the "Merchants" group
   - Account status is set to UNCONFIRMED

3. **Email Verification**:
   - Custom email is sent with verification code
   - Merchant-specific email template is used
   - User is redirected to verification page
   - Countdown timer shows code expiration

4. **Verification Completion**:
   - Merchant enters verification code
   - Account status changes to CONFIRMED
   - Follow-up email is sent with next steps

5. **First Login**:
   - Merchant logs in with verified credentials
   - Tokens are issued with appropriate scopes
   - Merchant is redirected to dashboard

### Implementation Details

The merchant sign-up flow is implemented in these key files:

- **Frontend**:
  - `sveltekit/src/routes/merchants/sign-up/+page.svelte`: Multi-step form UI
  - `sveltekit/src/routes/merchants/sign-up/+page.server.js`: Server-side validation and API calls
  - `sveltekit/src/routes/auth/verification-sent/+page.svelte`: Verification page UI

- **Backend**:
  - `backend/src/lambda/merchants/account/sign-up/handler.js`: Sign-up Lambda handler
  - `backend/src/lambda/auth/custom-message/handler.js`: Custom email Lambda trigger
  - `backend/lib/auth/user-pool/stack.js`: Cognito User Pool configuration

## Merchant-Specific Permissions

Merchants receive specific permissions through multiple mechanisms:

### Cognito Group Membership

Merchants are added to the "Merchants" group in Cognito:

```javascript
// Add user to Merchants group
await cognitoClient.send(
  new AdminAddUserToGroupCommand({
    UserPoolId: userPoolId,
    Username: username,
    GroupName: 'Merchants'
  })
);
```

This group membership:
- Identifies the user as a merchant
- Is included in the ID token as a claim
- Is used for role mapping in the Identity Pool

### IAM Role Assignment

Based on group membership, merchants receive the merchant IAM role:

```javascript
// Role mapping rule
{
  claim: 'cognito:groups',
  matchType: 'Contains',
  value: 'Merchants',
  roleArn: this.merchant.roleArn
}
```

This role provides:
- Access to specific AWS resources
- Temporary credentials for direct service access
- Permission boundaries for security

### OAuth Scopes

Merchants receive specific OAuth scopes in their access tokens:

- `deals:read`: Permission to view deals
- `deals:write`: Permission to create and update deals
- `deals:delete`: Permission to delete deals

These scopes are enforced at the API Gateway level.

## Resource Access Patterns

Merchants have access to specific resources through controlled patterns:

### S3 Access Pattern

Merchants can upload files directly to S3 with path-based restrictions:

```javascript
// S3 policy for merchants
const merchantS3Policy = new PolicyStatement({
  effect: Effect.ALLOW,
  actions: ["s3:PutObject"],
  resources: [`${storage.s3Bucket.bucketArn}/merchants/*`],
});
```

Key characteristics:
- Access is limited to the `/merchants/` path prefix
- Only `PutObject` action is allowed (no delete or list)
- Each merchant can only access their own subdirectory
- Path pattern: `/merchants/${userId}/deals/DEAL-${dealId}/logo/${filename}`

### API Access Pattern

Merchants access APIs through protected endpoints:

- **Authentication**: Bearer token in Authorization header
- **Authorization**: OAuth scopes in access token
- **Validation**: Request validation at API Gateway
- **Endpoint Pattern**: `/merchants/*` for merchant-specific operations

### Database Access Pattern

Merchants access database resources indirectly through APIs:

- No direct database access from client applications
- Lambda functions mediate all database operations
- Access is controlled by Lambda IAM roles
- Data is filtered based on merchant identity

## API Scopes for Merchants

Merchants have access to specific API scopes defined in the OAuth configuration:

### Available Scopes

| Scope | Description | Endpoints | Actions |
|-------|-------------|-----------|---------|
| `deals:read` | Read access to deals | `GET /merchants/{userId}/deals` | List, view deals |
| `deals:write` | Write access to deals | `POST /merchants/{userId}/deals` | Create, update deals |
| `deals:delete` | Delete access to deals | `DELETE /merchants/{userId}/deals/{dealId}` | Delete deals |

### Scope Implementation

Scopes are defined in the resource server configuration:

```javascript
// Define scopes for Deals API
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
```

And enforced at the API Gateway level:

```javascript
dealsResource.addMethod(
  "POST",
  new LambdaIntegration(lambda.merchants.deals.create.function),
  {
    ...http.optionsWithAuth.writeDealsAuth, // Use write scope for deal creation
  }
);
```

### Scope Assignment

Scopes are assigned based on user group membership:
- All merchants receive `deals:read` and `deals:write` scopes
- Additional scopes may be added based on merchant status or role

## Deal Creation Flow

The deal creation flow demonstrates the complete authorization process for merchants:

### Frontend Implementation

1. **User Authentication**:
   ```javascript
   // Check authentication in +page.server.js
   const idToken = cookies.get('idToken');
   if (!idToken) {
     throw error(401, 'Not authenticated');
   }
   ```

2. **Permission Verification**:
   ```javascript
   // Check if user has merchant permissions and can write deals
   if (!Utils.auth.isMerchant(session) || !Utils.auth.canWriteDeals(session)) {
     throw error(403, 'Insufficient permissions to create deals');
   }
   ```

3. **Credential Acquisition**:
   ```javascript
   // Get temporary AWS credentials
   const { credentials } = await fetchAuthSession();
   ```

4. **Direct S3 Upload**:
   ```javascript
   // Create S3 client with credentials
   const s3Client = new S3Client({
     region: 'us-east-1',
     credentials: {
       accessKeyId: credentials.accessKeyId,
       secretAccessKey: credentials.secretAccessKey,
       sessionToken: credentials.sessionToken
     }
   });

   // Upload file directly to S3
   await s3Client.send(new PutObjectCommand({
     Bucket: s3BucketName,
     Key: fileKey,
     Body: file,
     ContentType: file.type
   }));
   ```

5. **API Call with Authorization**:
   ```javascript
   // Send deal data to API with authorization header
   const response = await fetch('/api/merchants/${userId}/deals', {
     method: 'POST',
     headers: {
       'Authorization': `Bearer ${accessToken}`,
       'Content-Type': 'application/json'
     },
     body: JSON.stringify({
       title,
       description,
       logoFileKey: fileKey,
       // other deal properties
     })
   });
   ```

### Backend Authorization

1. **API Gateway Authorization**:
   - Validates access token signature
   - Checks for required scope (`deals:write`)
   - Rejects request if scope is missing

2. **Lambda Authorization**:
   - Verifies user identity from token claims
   - Ensures user ID in path matches token subject
   - Validates business rules (e.g., deal limits)

3. **Database Operation**:
   - Creates deal record with merchant ID
   - Associates S3 file reference with deal
   - Returns success response

### Complete Flow Diagram

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│             │      │             │      │             │      │             │
│   Browser   │──1──▶│   Cognito   │──2──▶│  Identity   │──3──▶│     S3      │
│             │◀─────│             │◀─────│    Pool     │◀─────│             │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
       │                                                               ▲
       │                                                               │
       │                                                               4
       │                                                               │
       5                                                               │
       │                                                               │
       ▼                                                               │
┌─────────────┐      ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│             │      │             │      │             │      │             │
│     API     │──6──▶│   Lambda    │──7──▶│  DynamoDB   │      │  Generated  │
│   Gateway   │◀─────│             │◀─────│             │      │   Image     │
└─────────────┘      └─────────────┘      └─────────────┘      └─────────────┘
```

1. User authenticates and receives tokens
2. Tokens are exchanged for temporary credentials
3. Credentials are used to upload file to S3
4. File is stored in merchant's S3 path
5. Deal data with file reference is sent to API
6. API validates token and forwards to Lambda
7. Lambda creates deal record in database

## Security Considerations

Merchant authorization implements several security measures:

### Principle of Least Privilege

- Merchants can only access their own resources
- S3 access is limited to specific paths and operations
- API access requires specific scopes
- Database access is mediated through Lambda functions

### Secure Token Handling

- Access tokens have short lifetimes (1 hour)
- Refresh tokens are stored securely
- Tokens are validated on every request
- CSRF protection is implemented

### Audit Trail

- All merchant actions are logged
- S3 object uploads are recorded
- API calls are logged in CloudWatch
- Authentication events are tracked

### Data Isolation

- Merchant data is isolated by user ID
- S3 paths include user ID for separation
- Database queries filter by merchant ID
- Cross-merchant access is prevented
