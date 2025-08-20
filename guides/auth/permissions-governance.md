# Permissions Governance: Ownership, Guardrails, Discoverability, Versioning

This guide describes how we design, own, discover, and evolve identity-based permissions and contracts across Super Deals microservices.

> Scope
>
> - Identity-facing policies: Permissions attached to human/user identities via Cognito groups → IAM roles.
> - Execution-role policies: Permissions attached to service identities (e.g., Lambda roles). These remain local to the owning service.

## Current Status

- Implemented
  - Centralized user-facing permissions attached to shared roles from `users-ms`, with service-specific statements added in consumer services (e.g., `deals-ms/lib/permissions/construct.ts`).
  - SSM publication of cross-service contract values and consumption via `infra-contracts`.
  - CHANGELOG and PR checklist introduced in `infra-contracts` (see repository for details).
- Deferred (to address later)
  - Permission boundaries on shared roles and SCP alignment as guardrails.
  - Tagging IAM roles/policies and adopting managed policy descriptions for console discoverability.
  - Formalized SSM contract versioning strategy (e.g., versioned prefixes) and documenting breaking changes across services.

## Ownership

- **Centralized user permissions**
  - Owned by `users-ms` team (Identity domain). They curate shared IAM roles assumed by application users via the Identity Pool.
  - Microservice teams propose resource-specific permissions to be granted to those shared roles.
- **Service execution permissions**
  - Owned locally by each service (e.g., `deals-ms`). Inline policies on Lambda execution roles are defined and reviewed by that service.
- **Change requests**
  - Consumers propose policy additions via PRs that touch: `deals-ms/lib/permissions/construct.ts` (consumer-proposed statements) and `users-ms` role definitions.
  - The Identity team reviews impact, overlaps, and least-privilege adherence.

## Guardrails

- **Least privilege**
  - Require explicit resource ARNs and actions; avoid wildcards. Use path-level constraints (e.g., `s3:PutObject` for `${BucketArn}/merchants/*`).
- **Separation of concerns**
  - User-facing permissions live on shared roles. Service runtime permissions live on Lambda roles. Do not mix.
- **Permission boundaries (optional)**
  - If needed, attach permission boundaries to shared roles to limit the maximum allowed actions regardless of attached policies.
  - Status: Deferred.
- **SCP alignment (org-level)**
  - If deploying to AWS Organizations, ensure policies do not rely on actions that are denied by SCPs; validate in lower environments.
  - Status: Deferred.
- **Trust policies**
  - For web identity roles, restrict `AssumeRoleWithWebIdentity` principals to the specific Cognito Identity Pool and audience conditions.
- **Policy review checklist**
  - Explicit resources, explicit actions, environment scoping via SSM/SSO parameters, no data-plane overreach, defense-in-depth with resource policies where applicable (e.g., S3 bucket policy).

## Discoverability

- **Naming conventions**
  - Roles: `app-<env>-<audience>-<capability>-role` (e.g., `app-dev-merchant-core-role`).
  - Policies: `app-<env>-<service>-<purpose>-policy`.
  - SSM params (contracts): `/super-deals/<env>/<domain>/<name>`.
- **Tagging**
  - Tag roles and policies with: `Service`, `Owner`, `Environment`, `DataDomain`, `Purpose`.
  - Status: Deferred.
- **Contracts registry**
  - Use SSM Parameter Store to publish discoverable ARNs and paths needed cross-repo. Backed by the `infra-contracts` package for typed access.
- **Documentation hooks**
  - Each consumer service documents which shared roles/scopes it relies on and which permissions it adds for users in `docs/guides/auth/*` and service READMEs.

## Versioning and Contracts

- **Semver for contracts**
  - Version the `infra-contracts` package using semver. Breaking policy/contract changes require a major bump.
  - Status: Deferred (policy and SSM contract semver discipline).
- **SSM path versioning**
  - For breaking changes to parameter shapes or locations, publish under a new versioned prefix (e.g., `/super-deals/v2/...`) and deprecate `/v1` after migration.
  - Status: Deferred.
- **Change management**
  - Every permission change must:
    - Link to a ticket with risk assessment and blast radius.
    - Include tests or deployment verification steps.
    - Update relevant guides and `CHANGELOG.md` in `infra-contracts`.
  - Status: CHANGELOG + PR checklist implemented; process discipline ongoing.
- **Rollout strategy**
  - Introduce new policies behind additive changes first. Validate in `dev` and `stage`. Remove deprecated permissions only after producers and consumers have migrated and monitoring shows no denials.

## Practical Patterns

- **Centralized role attachments**
  - Add user-facing permissions in `deals-ms/lib/permissions/construct.ts` by attaching `PolicyStatement`s to imported roles from `users-ms`.
- **Local execution permissions**
  - Keep Lambda permissions inline in service constructs (e.g., `deals-ms/lib/services/*/construct.ts`). Prefer explicit action/resource statements over grants when clarity matters.
- **Grants vs. inline policies**
  - `grant*` helpers are convenience wrappers but may obscure cross-cutting ownership. Prefer explicit `PolicyStatement`s for user-facing permissions. Grants are fine for execution roles local to the service.

## Examples

- SSM contract paths
  - Identity endpoints: `/super-deals/<env>/identity/issuerUrl`, `/super-deals/<env>/identity/userPoolId`.
  - Shared role ARNs: `/super-deals/<env>/iam/roles/merchant`.
- Policy statement (user-facing S3 put)
  ```ts
  new PolicyStatement({
    effect: Effect.ALLOW,
    actions: ["s3:PutObject"],
    resources: [`${storage.s3Bucket.bucketArn}/merchants/*`],
  });
  ```

## Responsibilities Matrix (RACI)

- Users/Identity team: Accountable for shared roles and cross-service policy hygiene.
- Consumer service teams: Responsible for proposing resource-specific user permissions and for their execution-role policies.
- Platform/Infra: Consulted on guardrails, SCPs, and permission boundaries.
- Security: Informed and involved in reviews for sensitive expansions.
