# SSM Contracts and the Generic Bindings Pattern

This document explains how we publish and consume cross-service contract values via SSM Parameter Store using:

- infra-contracts (shared, strongly-typed interfaces)
- SSM publications (producer code that writes params)
- An ergonomic, generic `BindingsConstruct<T>` and small `ssm.ts` helpers (consumer code that reads params)

The goal is to eliminate ad-hoc SSM reads, provide a single, ergonomic API for consumers, and keep type-safety via shared contracts.

## Table of Contents

- [Quick Start](#quick-start)
- [Contracts (infra-contracts)](#contracts-infra-contracts)
- [Publishing (Producer → SSM)](#publishing-producer--ssm)
- [Consuming (Consumer → BindingsConstruct)](#consuming-consumer--bindingsconstruct)
  - [Keys variant (simple suffix = key name)](#keys-variant-simple-suffix--key-name)
  - [Spec variant (key → SSM suffix mapping)](#spec-variant-key--ssm-suffix-mapping)
  - [Producer vs Consumer serviceName](#producer-vs-consumer-servicename)
  - [Why two reader styles?](#why-two-reader-styles)
  - [Design principles](#design-principles)
- [Versioning and Rollout Checklist](#versioning-and-rollout-checklist)

## Quick Start

- Producer (publisher)
  - Define a minimal interface in `infra-contracts` (no optional fields).
  - Publish only the required parameters under `/super-deals/<env>/<producer-service>/public/...`.
    ```ts
    // users-ms/lib/ssm-publications/auth/construct.ts
    publishStringParameters(this, basePath, {
      "auth/userPoolId": auth.userPool.pool.userPoolId,
    });
    ```

- Consumer (reader)
  - Use the ergonomic `BindingsConstruct<T>`.
  - If suffix == key name, use the keys variant; otherwise use the spec mapping variant.
  - Always set `producerServiceName` to the service that PUBLISHES the values.
    ```ts
    // Keys variant example (website)
    const website = new BindingsConstruct<IWebsiteBindings>(this, "WebsiteBindings", {
      envName: config.envName,
      producerServiceName: "website-ms",
      visibility: "public",
      keys: ["websiteUrl", "sourceEmail"] as const,
    });
    ```
    ```ts
    // Spec variant example (auth)
    const authSpec = { userPoolId: "auth/userPoolId" } as const;
    const auth = new BindingsConstruct<IAuthBindings>(this, "UsersMsAuthBindings", {
      envName: config.envName,
      producerServiceName: "users-ms",
      visibility: "public",
      spec: authSpec,
    });
    ```

- Use the values
  - `bindings.values.<key>` returns a strongly-typed string for use in your constructs.
  - Example: `UserPool.fromUserPoolId(this, "ImportedUserPool", auth.values.userPoolId)`

## Contracts (infra-contracts)

Expose only fields that consumers actually use; keep them required (no optionals):

```ts
// @super-deals/infra-contracts/src/users-ms/types.ts
export interface IAuthBindings {
  userPoolId: string;
}

export interface IIamBindings {
  merchantRoleArn: string;
}

export interface IWebsiteBindings {
  websiteUrl: string;
  sourceEmail: string;
}
```

## Publishing (Producer → SSM)

The producer service (e.g., `users-ms`) writes parameters under a base path:

```
/super-deals/<env>/<producer-service>/<visibility>/<key>
```

Examples:

```ts
// users-ms/lib/ssm-publications/construct.ts
// builds basePath like /super-deals/dev/users-ms/public
new AuthBindingsConstruct(this, "AuthBindingsConstruct", { basePath, region, auth });
new IamBindingsConstruct(this, "IamBindingsConstruct", { basePath, iam });
```

```ts
// users-ms/lib/ssm-publications/auth/construct.ts
publishStringParameters(this, basePath, {
  "auth/userPoolId": auth.userPool.pool.userPoolId,
});
```

```ts
// users-ms/lib/ssm-publications/iam/construct.ts
publishStringParameters(this, basePath, {
  "iam/roles/merchant/arn": iam.roles.merchant.roleArn,
});
```

Notes

- Visibility: Prefer `public` for cross-repo consumption.
- Minimal surface: Publish only the keys that consumers use.

## Consuming (Consumer → BindingsConstruct)

Each repo provides small SSM helpers in `src/helpers/ssm.ts`:

- `buildSsmPublicPath`, `buildSsmPrivatePath`
- `readParam`
- `readBindingsByKeys(scope, basePath, keys)`
- `readBindings(scope, basePath, spec)`

The generic `BindingsConstruct<T>` uses the helpers internally and supports two styles:

### Keys variant (simple suffix = key name)

```ts
// users-ms/lib/website/construct.ts
import BindingsConstruct from "#lib/utils/bindings/construct";
import type { IWebsiteBindings } from "@super-deals/infra-contracts";

const bindings = new BindingsConstruct<IWebsiteBindings>(this, "WebsiteBindings", {
  envName: config.envName,
  producerServiceName: "website-ms",
  visibility: "public",
  keys: ["websiteUrl", "sourceEmail"] as const,
});

this.websiteUrl = bindings.values.websiteUrl;
this.sourceEmail = bindings.values.sourceEmail;
```

### Spec variant (key → SSM suffix mapping)

```ts
// deals-ms/lib/auth/construct.ts
import BindingsConstruct from "#lib/utils/bindings/construct";
import type { IAuthBindings } from "@super-deals/infra-contracts";

const authSpec = { userPoolId: "auth/userPoolId" } as const;

const authBindings = new BindingsConstruct<IAuthBindings>(this, "UsersMsAuthBindings", {
  envName: config.envName,
  producerServiceName: "users-ms",
  visibility: "public",
  spec: authSpec,
});

const userPool = UserPool.fromUserPoolId(this, "ImportedUserPool", authBindings.values.userPoolId);
```

```ts
// deals-ms/lib/iam/construct.ts
import BindingsConstruct from "#lib/utils/bindings/construct";
import type { IIamBindings } from "@super-deals/infra-contracts";

const iamSpec = { merchantRoleArn: "iam/roles/merchant/arn" } as const;

const iamBindings = new BindingsConstruct<IIamBindings>(this, "UsersMsIamBindings", {
  envName: config.envName,
  producerServiceName: "users-ms",
  visibility: "public",
  spec: iamSpec,
});

const merchantRole = Role.fromRoleArn(this, "ImportedMerchantRole", iamBindings.values.merchantRoleArn);
```

### Producer vs Consumer serviceName

- Always point the consumer to the producer by setting `producerServiceName` to the service that publishes the parameters (e.g., `"users-ms"`, `"website-ms"`).
- Avoid using `config.service.name` in consumers unless the consumer and producer are the same service.

### Why two reader styles?

- Use the keys variant when your SSM key suffixes match your interface keys (simpler and more concise).
- Use the spec variant when you need to map logical keys to different SSM suffixes (e.g., nested paths like `iam/roles/merchant/arn`).

### Design principles

- Minimal contracts: Contracts expose only fields used by consumers.
- No optional values: Shared contracts should not contain optional fields.
- Strong typing end-to-end: `infra-contracts` types drive both the producer and consumer shapes, reducing errors.
- Single, ergonomic construct: `BindingsConstruct<T>` + helpers encapsulate environment, base paths, and reading logic.

## Versioning and Rollout Checklist

- Add new keys additively; do not break consumers.
- Publish changes to `infra-contracts` with semver and update consumers.
- If you must change SSM paths, publish under a new prefix and deprecate the old one.
- Keep docs + CHANGELOG in sync with contract changes.
