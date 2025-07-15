# Authentication and Authorization Overview

This document provides a comprehensive overview of the authentication and authorization system implemented in the Super-Deals application. It covers the core concepts, AWS services used, and the implementation patterns that secure our application.

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [AWS Services](#aws-services)
3. [Token Types](#token-types)
4. [OAuth 2.0 Implementation](#oauth-20-implementation)
5. [Authorization Flow](#authorization-flow)
6. [Direct S3 Upload Pattern](#direct-s3-upload-pattern)
7. [Security Considerations](#security-considerations)

## Core Concepts

### Authentication vs. Authorization

**Authentication** and **authorization** are two fundamental security concepts that work together to secure our application:

- **Authentication (AuthN)**: Verifies the identity of a user or system. It answers the question, "Who are you?"
  - Implemented via Cognito User Pools
  - Handles user registration, login, and verification
  - Issues tokens that prove a user's identity

- **Authorization (AuthZ)**: Determines what actions an authenticated user is allowed to perform. It answers the question, "What are you allowed to do?"
  - Implemented via multiple layers:
    - Cognito User Groups (e.g., Merchants)
    - IAM Roles and Policies
    - OAuth 2.0 Scopes
    - API Gateway Authorizers

### Identity-Based vs. Resource-Based Authorization

Our system implements both types of authorization:

- **Identity-Based Authorization**: Permissions attached to identities (users, groups, roles)
  - Example: Merchant group members receive the merchant IAM role
  - Managed through Cognito groups and IAM roles

- **Resource-Based Authorization**: Permissions attached to resources
  - Example: S3 bucket policies restricting access to specific paths
  - Managed through AWS resource policies

## AWS Services

Our authentication and authorization system leverages several AWS services:

### Amazon Cognito User Pools

- **Purpose**: User directory and authentication service
- **Implementation**: `backend/lib/auth/user-pool/stack.js`
- **Features**:
  - User registration and sign-in
  - Multi-factor authentication
  - Email verification
  - Password policies and account recovery
  - Custom attributes for user metadata
  - User groups for role-based access control

### Amazon Cognito Identity Pools

- **Purpose**: Issues temporary AWS credentials based on user identity
- **Implementation**: `backend/lib/auth/identity-pool/stack.js`
- **Features**:
  - Maps authenticated users to IAM roles
  - Provides temporary, limited-privilege AWS credentials
  - Enables direct access to AWS services from client applications
  - Supports rule-based role mapping based on user attributes or group membership

### AWS Identity and Access Management (IAM)

- **Purpose**: Manages access to AWS resources
- **Implementation**: `backend/lib/iam/stack.js` and `backend/lib/iam/roles/stack.js`
- **Features**:
  - Role definitions for different user types
  - Policy attachments for resource access
  - Fine-grained permissions for AWS services
  - Role assumption via web identity federation

### Amazon API Gateway

- **Purpose**: Manages and secures API endpoints
- **Implementation**: `backend/lib/api/http/stack.js`
- **Features**:
  - Request validation
  - Cognito authorizer integration
  - OAuth scope enforcement
  - Method-level authorization

### Amazon S3

- **Purpose**: Secure object storage
- **Implementation**: Integrated with IAM for access control
- **Features**:
  - Direct uploads from client applications
  - Path-based access control
  - Temporary credential support

## Token Types

Our authentication system uses several types of tokens:

### ID Token

- **Purpose**: Contains user identity information
- **Format**: JWT (JSON Web Token)
- **Contents**:
  - User attributes (email, name, etc.)
  - Group memberships
  - Custom claims
- **Usage**: Identifying the user in the application
- **Validity**: Typically 1 hour (configurable)

### Access Token

- **Purpose**: Grants access to resources
- **Format**: JWT (JSON Web Token)
- **Contents**:
  - User identity
  - OAuth scopes (permissions)
  - Expiration time
- **Usage**: Accessing protected API endpoints
- **Validity**: Typically 1 hour (configurable)

### Refresh Token

- **Purpose**: Obtains new ID and access tokens without re-authentication
- **Format**: Opaque token
- **Usage**: Refreshing expired tokens
- **Validity**: Typically 30 days (configurable)

### Temporary AWS Credentials

- **Purpose**: Allows direct access to AWS services
- **Contents**:
  - Access key ID
  - Secret access key
  - Session token
- **Usage**: Direct S3 uploads, other AWS service access
- **Validity**: Typically 1 hour

## OAuth 2.0 Implementation

Our application implements OAuth 2.0 for API authorization:

### Resource Servers

Resource servers represent protected APIs in our system:

```javascript
// Create Resource Server
this.resourceServer = new UserPoolResourceServer(this, 'ResourceServer', {
  userPool,
  identifier: `deals-${envName}`,
  scopes: this.scopes,
});
```

- **Implementation**: `backend/lib/auth/user-pool/resource-servers/deals/construct.js`
- **Purpose**: Define protected APIs and their scopes

### Scopes

Scopes define fine-grained permissions for API access:

```javascript
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

- **Implementation**: `backend/lib/permissions/oauth-permissions/deals/construct.js`
- **Purpose**: Define granular permissions for API operations

### Authorization Code Flow

Our application uses the OAuth 2.0 Authorization Code flow:

1. User authenticates with Cognito User Pool
2. User is redirected to the application with an authorization code
3. Application exchanges code for tokens
4. Application uses tokens to access protected resources

This flow is configured in the User Pool Client:

```javascript
this.poolClient = this.pool.addClient(`UserPoolClient`, {
  generateSecret: true,
  authFlows: {
    userPassword: true,
    adminUserPassword: true,
  },
  oAuth: {
    flows: {
      authorizationCodeGrant: true,
    },
    scopes: [OAuthScope.OPENID, OAuthScope.EMAIL, OAuthScope.PROFILE],
    callbackUrls: ["http://localhost:5173"],
  },
});
```

## Authorization Flow

The complete authorization flow in our application works as follows:

1. **User Registration**:
   - User registers via Cognito User Pool
   - User is assigned to appropriate groups (e.g., Merchants)
   - Email verification is required

2. **Authentication**:
   - User logs in with username/password
   - Cognito issues ID, access, and refresh tokens
   - Access token contains OAuth scopes based on user groups

3. **Identity Federation**:
   - Tokens are exchanged for temporary AWS credentials via Cognito Identity Pool
   - Credentials are associated with appropriate IAM role based on user groups

4. **API Authorization**:
   - API requests include access token in Authorization header
   - API Gateway validates token and checks required scopes
   - Request is authorized or denied based on token scopes

5. **Direct AWS Service Access**:
   - Client uses temporary credentials to access AWS services directly
   - IAM policies on assumed role control permitted actions

## Direct S3 Upload Pattern

Our application implements a direct-to-S3 upload pattern for efficient file handling:

### Benefits

- **Performance**: Bypasses API Gateway limits for large files
- **Cost Efficiency**: Reduces Lambda execution time and API Gateway data transfer
- **Security**: Uses temporary credentials with limited permissions
- **Scalability**: Offloads file handling to S3

### Implementation

The pattern is implemented as follows:

1. **Frontend Requests Credentials**:
   ```javascript
   const { credentials } = await fetchAuthSession();
   ```

2. **Frontend Creates S3 Client**:
   ```javascript
   const s3Client = new S3Client({
     region: 'us-east-1',
     credentials: {
       accessKeyId: credentials.accessKeyId,
       secretAccessKey: credentials.secretAccessKey,
       sessionToken: credentials.sessionToken
     }
   });
   ```

3. **Frontend Uploads Directly to S3**:
   ```javascript
   await s3Client.send(new PutObjectCommand({
     Bucket: s3BucketName,
     Key: fileKey,
     Body: file,
     ContentType: file.type
   }));
   ```

4. **Frontend Sends File Reference to API**:
   ```javascript
   formData.set('logoFileKey', fileKey);
   ```

5. **Backend Stores File Reference**:
   - Lambda function receives the file reference (S3 key)
   - Stores the reference in the database
   - No binary data handling required

### Security Considerations

The direct S3 upload pattern is secured through:

- **Temporary Credentials**: Short-lived and scoped to specific actions
- **Path-Based Restrictions**: Users can only access their own paths
- **Action Limitations**: Typically limited to PutObject operations
- **Content Validation**: File type and size validation before upload

## Security Considerations

Our authentication and authorization system implements several security best practices:

### Defense in Depth

Multiple layers of security protect our application:

- **Authentication**: Cognito User Pools with MFA support
- **Authorization**: Multiple authorization mechanisms (IAM, OAuth, API Gateway)
- **Validation**: Request validation at API Gateway and Lambda levels
- **Encryption**: Data encryption in transit and at rest

### Principle of Least Privilege

Users and systems receive only the permissions they need:

- **Fine-Grained Scopes**: API permissions limited to specific operations
- **Path-Based Access**: S3 access limited to specific paths
- **Temporary Credentials**: Short-lived credentials with limited scope
- **Role-Based Access**: Permissions assigned based on user roles

### Token Security

Token security is maintained through:

- **Short Lifetimes**: Access tokens typically valid for 1 hour
- **Secure Storage**: Tokens stored in HTTP-only cookies
- **CSRF Protection**: State parameters in OAuth flows
- **Signature Validation**: JWT signature validation

### Monitoring and Auditing

Security monitoring is implemented through:

- **CloudWatch Logs**: API and Lambda function logging
- **CloudTrail**: AWS API call auditing
- **API Gateway Access Logs**: API request logging
- **Cognito Event Logging**: Authentication event tracking
