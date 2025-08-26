# Service Discovery and BFF (Backend for Frontend)

This document describes the current service discovery approach and the BFF plan used by Super Deals.

## Phase 1: BFF proxy and static config

- The web app exposes `/api/*` endpoints as a thin proxy to microservices.
- The web layout fetches config from `web/src/routes/api/config/+server.ts` during server-load and provides it to all pages.
- No client-stored secrets are required; public bindings only.

## Phase 2: Per-service service discovery

Each microservice exposes a well-known endpoint:
- Path: `/.well-known/bindings`
- Method: `GET`
- Purpose: return public, non-sensitive runtime bindings for that service.

### Binding sources

- Primary: SSM Parameter Store under `{APP_BASE_PATH}/{env}/{service}/public`.
- Fallback: environment variables if SSM path is not set or empty.

#### Base path resolution (standard)

To avoid hardcoding, the SSM base path is resolved with the following precedence:

1. `APP_BASE_PATH` environment variable (recommended)
2. `parameterStorePrefix` from service config (legacy/back-compat)
3. Fallback: `/super-deals`

The final public path is constructed as:

```
{APP_BASE_PATH}/{ENV_NAME}/{SERVICE_NAME}/public
```

Notes:
- `SERVICE_NAME` should be set per service (e.g., `deals-ms`, `users-ms`).
- Private bindings must use the matching `.../private` path and are never exposed by discovery endpoints.

### SSM layout (public)

- users-ms: `{APP_BASE_PATH}/{env}/users-ms/public`
  - `auth/userPoolId`
  - `auth/userPoolClientId`
  - `auth/identityPoolId`
  - `auth/cognitoDomain`
  - `auth/oauthAuthorizeUrl`
  - `auth/oauthTokenUrl`
  - `region`

- deals-ms: `{APP_BASE_PATH}/{env}/deals-ms/public`
  - `api/baseUrl` (published by API Stage construct)
  - `region`

Private bindings (e.g., secrets) should be stored under `{APP_BASE_PATH}/{env}/{service}/private` and are not exposed by the `/.well-known/bindings` endpoint.

## CDK implementation summary

- For each service:
  - A Lambda handler `src/services/services-discovery/lambda-handler.ts` reads SSM via `@aws-sdk/client-ssm` (`GetParametersByPath`) with `SSM_PUBLIC_PATH`.
  - `SSM_PUBLIC_PATH` is injected from CDK using the resolved base path formula above.
  - IAM permission is added to the Lambda for `ssm:GetParametersByPath` on `arn:aws:ssm:*:*:parameter{SSM_PUBLIC_PATH}*`.
  - An endpoint construct wires `GET /.well-known/bindings` to the discovery Lambda.
  - An SSM publication helper publishes public values on synth:
    - users-ms: `lib/services/services-discovery/ssm-publication.ts` used by `lib/services/construct.ts`.
    - deals-ms: same helper under `lib/services/services-discovery/ssm-publication.ts` used by `lib/services/construct.ts`.
  - deals-ms additionally publishes `api/baseUrl` from `lib/api/stage/construct.ts` to SSM so clients can discover the REST endpoint.

## Web usage

- The web app can either:
  1. Continue using the BFF `/api/*` proxy for all calls (Phase 1), or
  2. Query each service's `/.well-known/bindings` when needed to dynamically configure SDK clients (Phase 2). For example, the auth service (users-ms) bindings provide Cognito IDs and OAuth URLs.

## Notes

- CORS is configured per root resource using `optionsWithCors` in API constructs.
- No secrets are exposed in public bindings. Keep private values under the `private` SSM path and never return them from the discovery Lambda.
- Deployments: ensure environments define `envName` consistently (e.g., `dev`, `staging`).
