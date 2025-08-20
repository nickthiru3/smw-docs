# End-to-End Auth Model (users-ms → deals-ms)

This guide documents how authentication and authorization are provisioned in users-ms and consumed by deals-ms. It covers the constructs, SSM/contract bindings, Cognito resource server and scopes, API Gateway authorizer, and IAM policies.

## High-level flow

1. users-ms provisions core Auth and IAM (User Pool, Identity Pool, Roles, Groups).
2. users-ms publishes binding parameters (userPoolId, role ARNs) via the infra-contracts package backed by SSM.
3. deals-ms imports those bindings to reference the existing User Pool and IAM roles.
4. deals-ms defines a Cognito Resource Server and OAuth scopes for its API under the imported User Pool.
5. deals-ms creates a Cognito User Pools authorizer and attaches method-level authorizationScopes.
6. deals-ms attaches an S3 PutObject policy to the imported merchant role for controlled direct uploads.

---

## users-ms (producer of auth/IAM)

- `lib/auth/construct.ts`
  - Creates the Cognito User Pool and client, Identity Pool, and user groups.
  - Exposes these via child constructs:
    - `user-pool/construct.ts`
    - `identity-pool/construct.ts`
    - `user-groups/construct.ts`

- `lib/iam/construct.ts`
  - Creates roles in `iam/roles/construct.ts` (e.g., Merchant role).
  - Identity Pool role mapping typically routes merchants (via `cognito:groups`) to the merchant role.

- Bindings contract (via `@super-deals/infra-contracts`)
  - Publishes SSM parameters for:
    - `userPoolId`
    - `merchantRoleArn` (and optionally `authenticatedRoleArn` and `unauthenticatedRoleArn`)
  - Consumer stacks (deals-ms) retrieve these via the same package.

---

## deals-ms (consumer and API owner)

### 1) Import Auth and IAM bindings

- `lib/auth/construct.ts`
  - Imports User Pool via SSM bindings:
    - `const auth = getAuthBindings(this, envName)`
    - `this.userPool = UserPool.fromUserPoolId(..., auth.userPoolId)`

- `lib/iam/construct.ts`
  - Imports IAM roles via SSM bindings:
    - `const iamB = getIamBindings(this, envName)`
    - `merchant: iam.Role.fromRoleArn(..., iamB.merchantRoleArn)`

### 2) Define resource server and scopes

- `lib/permissions/resource-server/construct.ts`
  - Creates `UserPoolResourceServer` on the imported pool (`auth.userPool`).
  - Identifier: `deals-${envName}`.
  - Scopes: `read`, `write`, `delete`.
  - Exposes slash-form scopes for API Gateway via `getOAuthScopes()`:
    - `${identifier}/${scopeName}` e.g., `deals-dev/write`.

### 3) OAuth helper to supply API Gateway method options

- `lib/permissions/oauth/construct.ts`
  - Accepts `ResourceServerConstruct` wrapper.
  - `getAuthOptions(authorizerId)` returns three method configs:
    - `readDealsAuth | writeDealsAuth | deleteDealsAuth`
  - Each has:
    - `authorizationType: "COGNITO_USER_POOLS"`
    - `authorizer: { authorizerId }`
    - `authorizationScopes: string[]` (slash-form scopes filtered for read/write/delete)

### 4) Permissions and S3 policy attachment

- `lib/permissions/construct.ts`
  - Composes resource server and OAuth constructs.
  - Attaches S3 PutObject policy for merchant uploads to imported role:
    - `iam.roles.merchant.addToPrincipalPolicy(new PolicyStatement({
         actions: ["s3:PutObject"],
         resources: [`${storage.s3Bucket.bucketArn}/merchants/*`],
       }))`

### 5) API Gateway authorizer and endpoints

- `lib/api/authorization/construct.ts`
  - Creates `CognitoUserPoolsAuthorizer` with the imported `IUserPool`.
  - Calls `permissions.oauth.getAuthOptions(authorizerId)` to produce per-scope method options:
    - `this.authOptions.deals.{readDealsAuth, writeDealsAuth, deleteDealsAuth}`.

- `lib/api/construct.ts`
  - Instantiates `AuthorizationConstruct` and passes its options down to endpoints as `http.optionsWithAuth`.

- `lib/api/endpoints/post/construct.ts`
  - Adds the POST method for deals creation and spreads the write scope config:
    - `...http.optionsWithAuth.writeDealsAuth`

---

## Token and access flow

1. User authenticates against Cognito (users-ms User Pool) and receives tokens.
2. Access token includes scopes per resource servers enabled for the app client.
3. Client calls deals-ms API with `Authorization: Bearer <access_token>`.
4. API Gateway (deals-ms) validates token via Cognito authorizer and enforces required scope(s) on the method.
5. Lambda executes if scope present; otherwise, API Gateway rejects.
6. For S3 uploads: client exchanges tokens for temporary AWS creds via Identity Pool (users-ms), assumes merchant role, and uploads only to allowed `/merchants/*` paths.

---

## Key conventions

- Slash-form OAuth scopes for API Gateway:
  - `${resourceServerIdentifier}/${scopeName}` (e.g., `deals-dev/write`).
- Keep `IUserPool` references in consumer stacks (don’t re-create pools).
- Import IAM roles via ARNs and attach narrowly-scoped policies in the consumer.

---

## Current state vs. legacy docs

- Existing docs (`docs/guides/auth/overview.md`, `merchant-authorization.md`, `implementation-patterns.md`) include older examples using colon-form scopes and monorepo-internal constructs.
- The current implementation uses cross-repo SSM bindings via `@super-deals/infra-contracts` and slash-form scopes for API Gateway.
- Update those docs over time to reflect:
  - SSM contract usage for userPoolId and role ARNs.
  - Deals resource server identifier pattern and slash-form scopes.
  - Method-level `authorizationScopes` enforcement and authorizer wiring.

---

## Follow-ups

- Ensure users-ms app clients are configured to allow the deals resource server scopes.
- Decide token handling (cookie/header) in BFF and add verification.
- Add structured auth logs, correlation IDs, and service-level defense-in-depth checks.
- Update OpenAPI and sequence diagrams for the new flow.
