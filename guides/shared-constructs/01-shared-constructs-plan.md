# Shared CDK Constructs Plan

**Author:** Thiru AI Labs  
**Created:** November 30, 2025  
**Status:** Draft - Under Discussion

---

## 1. Problem Statement

### Current Issues

1. **Template Drift**: `svc-ai-template` and `svc-users-template` have diverged in implementation patterns (e.g., bindings endpoint at `lib/api/endpoints/bindings` vs `lib/api/endpoints/users/well-known`)

2. **Maintenance Burden**: Updates to common patterns require changes across multiple repositories

3. **Consistency Risk**: Teams implementing new services may pick up outdated patterns from older templates

4. **Documentation Duplication**: Extensive implementation guides exist in each template, leading to potential inconsistencies

### Business Context

From the technology decisions documentation: AI agentic applications will also use regular CRUD-type backend services - not everything requires AI/agents. This means:

- Multiple service types will share common infrastructure patterns
- Auth constructs are needed by the users service (and consumed by others)
- AI constructs are needed only by AI-enabled services
- General constructs are needed by all services

---

## 2. Proposed Solution

### Package Structure

Create a single monorepo package with internal organization:

```
@thiru-ai-labs/cdk-constructs/
├── src/
│   ├── core/           # General constructs (API Gateway, Lambda, DynamoDB, monitoring)
│   ├── auth/           # Auth constructs (Cognito, IAM roles, identity pools)
│   └── ai/             # AI constructs (Bedrock, vector DBs, agent patterns)
├── docs/               # Consolidated implementation documentation
├── test/               # Construct tests
└── package.json
```

### Why Single Package (Monorepo Style)?

- **Simpler for solo development**: One repo to manage, one version to track
- **Easier internal refactoring**: Can move code between modules without breaking consumers
- **Shared dependencies**: CDK libs, constructs, etc. managed in one place
- **Can split later**: If needed, can extract into separate packages when boundaries are clearer

### Related Package: infra-contracts

The existing `infra-contracts` package serves a different purpose:

- **Types-only**: Defines TypeScript interfaces for SSM parameter contracts
- **Inter-service communication**: Allows services to share compile-time shapes
- **No CDK constructs**: Just type definitions

Both packages work together:

- `@thiru-ai-labs/cdk-constructs` - Infrastructure patterns (CDK constructs)
- `@thiru-ai-labs/infra-contracts` - Inter-service contracts (types only)

---

## 3. What Goes Where

### Extraction Approach

The shared constructs are **direct copies** of working constructs from `svc-merchants/lib/`, preserving the internal structure and sub-constructs. This ensures:

1. **Minimal changes to service-stack** - Import paths change, but instantiation remains the same
2. **Preserved patterns** - Sub-constructs (e.g., `api/stage/`, `monitor/ses/`) remain intact
3. **Working code** - No need to rewrite or "generalize" - just fix import paths

### Shared Constructs Package

| Module    | Contents                                                                                                                                                                                               |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **core/** | `ApiConstruct`, `DatabaseConstruct`, `MonitorConstruct`, `SsmBindingsConstruct`, `SsmPublicationsConstruct`, `PermissionsConstruct`, `EventsConstruct`, `PipelineConstruct`, plus utilities (`utils/`) |
| **auth/** | `AuthConstruct` (in `cognito/`), `IamConstruct` (in `iam/`)                                                                                                                                            |
| **ai/**   | `AgentsConstruct`, `VectorDbConstruct` (Pinecone, OpenSearch)                                                                                                                                          |

### What Stays in Service Templates

| Stays in Template             | Reason                                                |
| ----------------------------- | ----------------------------------------------------- |
| `config/` directory           | Service-specific configuration values                 |
| `bin/app.ts`                  | Service entry point                                   |
| `lib/service-stack.ts`        | Orchestrates constructs (imports from cdk-constructs) |
| `lib/api/endpoints/`          | Service-specific API endpoints and handlers           |
| `src/` (handlers, DAL, types) | Business logic, data access, domain types             |

### API Endpoints Pattern

The `ApiConstruct` in `cdk-constructs` creates the RestApi, stages, authorization, and CORS configuration. However, **endpoints are service-specific** and stay in the service template.

The `ApiConstruct` exposes `apiProps` which the service uses to create its endpoints:

```typescript
// In service-stack.ts
const api = new ApiConstruct(this, "ApiConstruct", {
  config,
  auth,
  permissions,
});

// Service-specific endpoints (stays in service template)
new MerchantsEndpointsConstruct(this, "EndpointsConstruct", {
  apiProps: api.apiProps, // { restApi, optionsWithCors, optionsWithAuth }
  config,
  db,
  auth,
});
```

### Documentation Split

| Location                    | Content                                                                    |
| --------------------------- | -------------------------------------------------------------------------- |
| **Shared constructs docs/** | How to use constructs, patterns, best practices, architecture guides       |
| **Template docs/**          | Service-specific setup, business logic guides, deployment for that service |

---

## 4. Configuration Management

### Challenge

Constructs need configuration, but configuration values are service-specific. Not all services use all constructs.

### Approach: Construct Owns Its Config Interface

Each construct in `cdk-constructs` defines the configuration interface it requires. This provides:

1. **Clear contract**: Construct explicitly declares what config it needs
2. **Type safety**: Service developer gets compile-time validation
3. **Self-documenting**: Props interface shows exactly what's required
4. **Flexible composition**: Service only includes config for constructs it uses

### Config Categories

Based on analysis of `svc-merchants/config/`:

**Category 1: Base Config (Every Service Needs)**

| Config         | Interface          | Purpose                          |
| -------------- | ------------------ | -------------------------------- |
| `service.ts`   | `IServiceConfig`   | Service name, display name       |
| `resources.ts` | `IResourcesConfig` | Resource naming prefixes         |
| Base fields    | `IBaseConfig`      | `envName`, `accountId`, `region` |

**Category 2: Construct-Specific Config**

| Config        | Interface         | Related Construct      |
| ------------- | ----------------- | ---------------------- |
| `database.ts` | `IDatabaseConfig` | `DatabaseConstruct`    |
| `api.ts`      | `IApiConfig`      | `ApiConstruct`         |
| `features.ts` | `IFeaturesConfig` | `PermissionsConstruct` |

**Category 3: Environment/Pipeline Config (Deferred)**

| Config                        | Interface            | Related Construct   |
| ----------------------------- | -------------------- | ------------------- |
| `github.ts`                   | `IGitHubConfig`      | `PipelineConstruct` |
| `localstack.ts`               | `IEnvironmentConfig` | `PipelineConstruct` |
| `staging.ts`, `production.ts` | `IEnvironmentConfig` | `PipelineConstruct` |

### Configuration Structure

```
cdk-constructs/src/
├── types.ts                    # IBaseConfig, IServiceConfig, IResourcesConfig
├── core/
│   ├── types.ts                # Re-exports from sub-modules
│   ├── db/
│   │   └── types.ts            # IDatabaseConfig
│   ├── api/
│   │   └── types.ts            # IApiConfig
│   ├── monitor/
│   │   └── types.ts            # IMonitorConfig
│   └── ...
├── auth/
│   ├── types.ts                # Re-exports
│   ├── cognito/
│   │   └── types.ts            # IAuthConfig
│   └── iam/
│       └── types.ts            # IIamConfig
└── pipeline/                   # DEFERRED
    └── types.ts                # IGitHubConfig, IEnvironmentConfig
```

### Base Config Interface

```typescript
// @thiru-ai-labs/cdk-constructs/src/types.ts

/** Service metadata - every service needs this */
export interface IServiceConfig {
  readonly name: string;
  readonly displayName: string;
}

/** Resource naming prefixes - every service needs this */
export interface IResourcesConfig {
  readonly tablePrefix: string;
  readonly bucketPrefix: string;
  readonly functionPrefix: string;
  readonly apiPrefix: string;
}

/** Base configuration required by all constructs */
export interface IBaseConfig {
  readonly envName: string;
  readonly accountId: string;
  readonly region: string;
  readonly service: IServiceConfig;
  readonly resources: IResourcesConfig;
}
```

### Construct-Specific Config

```typescript
// @thiru-ai-labs/cdk-constructs/src/core/db/types.ts

export interface IDatabaseConfig {
  readonly approach: "faux-sql" | "single-table";
  readonly fauxSql?: IFauxSqlConfig;
  readonly singleTable?: ISingleTableConfig;
}

// What DatabaseConstruct requires
export type TDatabaseConstructConfig = IBaseConfig & {
  readonly database: IDatabaseConfig;
};
```

```typescript
// @thiru-ai-labs/cdk-constructs/src/core/db/construct.ts

interface IDatabaseConstructProps {
  readonly config: TDatabaseConstructConfig;
}
```

### Service Implementation

The service composes its `IConfig` based on which constructs it uses:

```typescript
// svc-merchants/config/types.ts
import type { IBaseConfig } from "@thiru-ai-labs/cdk-constructs";
import type {
  IDatabaseConfig,
  IApiConfig,
} from "@thiru-ai-labs/cdk-constructs/core";
import type { IAuthConfig } from "@thiru-ai-labs/cdk-constructs/auth";

export interface IConfig extends IBaseConfig {
  readonly database: IDatabaseConfig;
  readonly api: IApiConfig;
  // Only include what this service uses
}
```

```typescript
// svc-merchants/config/database.ts
import type { IDatabaseConfig } from "@thiru-ai-labs/cdk-constructs/core";

export const databaseConfig: IDatabaseConfig = {
  approach: "faux-sql",
  fauxSql: {
    tables: [
      /* ... */
    ],
  },
};
```

### Subpath Exports

```json
{
  "exports": {
    ".": "./dist/index.js",
    "./core": "./dist/core/index.js",
    "./auth": "./dist/auth/index.js",
    "./ai": "./dist/ai/index.js"
  }
}
```

### Benefits

- **Construct creator defines contract**: Each construct specifies exactly what config it needs
- **Compile-time validation**: TypeScript catches missing or incorrect config at build time
- **Flexible composition**: Services only include config for constructs they use
- **Clear ownership**: Construct author responsible for defining and validating config needs

### Note: Deferred Pipeline Config

Environment configs (`localstack.ts`, `staging.ts`, `production.ts`) and `github.ts` relate to `PipelineConstruct` which is deferred. When implemented, these types will live in `cdk-constructs/src/pipeline/types.ts`.

---

## 5. Construct Design Principles

### 1. Composition over Inheritance

```typescript
// Good: Composable constructs
class ApiEndpointConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IApiEndpointProps) {
    // Uses LambdaConstruct internally
    const lambda = new LambdaConstruct(this, "Lambda", props.lambdaConfig);
    // Composes with API Gateway
  }
}
```

### 2. Sensible Defaults with Override Capability

```typescript
interface ILambdaConstructProps {
  readonly entry: string;
  readonly handler?: string; // Default: 'handler'
  readonly runtime?: Runtime; // Default: NODEJS_20_X
  readonly memorySize?: number; // Default: 512
  readonly timeout?: Duration; // Default: 30 seconds
}
```

### 3. Props Interfaces for All Constructs

```typescript
// Every construct has a props interface
export interface IMyConstructProps {
  readonly config: IServiceConfig;
  readonly requiredProp: string;
  readonly optionalProp?: number;
}
```

### 4. Expose What Consumers Need

```typescript
class DatabaseConstruct extends Construct {
  // Expose resources consumers might need
  public readonly table: Table;
  public readonly tableArn: string;
  public readonly tableName: string;

  // Don't expose internal implementation details
  private readonly internalHelper: SomeHelper;
}
```

---

## 6. Development Workflow

### For Development (Current Phase)

```
smw/
├── cdk-constructs/           # NEW: Shared constructs package
├── infra-contracts/          # Existing: Types-only contracts
├── svc-users-template-v2/    # Copy of users template for testing
├── svc-ai-template-v2/       # Copy of AI template for testing
└── ...
```

### Local Development Import

```json
// svc-users-template-v2/package.json
{
  "devDependencies": {
    "@thiru-ai-labs/cdk-constructs": "file:../cdk-constructs",
    "@thiru-ai-labs/infra-contracts": "file:../infra-contracts"
  }
}
```

### For Production (Future)

```json
// package.json
{
  "devDependencies": {
    "@thiru-ai-labs/cdk-constructs": "^1.0.0",
    "@thiru-ai-labs/infra-contracts": "^1.0.0"
  }
}
```

---

## 7. Migration Strategy

### Phase 1: Setup (Current)

1. Create `cdk-constructs` project directory
2. Set up package structure and build configuration
3. Create copies of templates for testing (`svc-users-template-v2`, `svc-ai-template-v2`)

### Phase 2: Extract Core Constructs

1. Identify common constructs between templates
2. Extract to `cdk-constructs/src/core/`
3. Define configuration interfaces
4. Update template copies to import from shared package

### Phase 3: Extract Auth Constructs

1. Extract auth-specific constructs from users template
2. Move to `cdk-constructs/src/auth/`
3. Update users template to consume

### Phase 4: Extract AI Constructs

1. Extract AI-specific constructs from AI template
2. Move to `cdk-constructs/src/ai/`
3. Update AI template to consume

### Phase 5: Documentation Consolidation

1. Move implementation guides to shared package
2. Update template docs to reference shared docs
3. Remove duplicated documentation

### Phase 6: Validation & Cleanup

1. Test both templates with shared constructs
2. Ensure all patterns work correctly
3. Remove original template copies (keep v2 versions)
4. Update infra-contracts dependency in templates

---

## 8. Package Dependencies

### cdk-constructs/package.json

```json
{
  "name": "@thiru-ai-labs/cdk-constructs",
  "version": "0.1.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "peerDependencies": {
    "aws-cdk-lib": "^2.200.0",
    "constructs": "^10.0.0"
  },
  "devDependencies": {
    "aws-cdk-lib": "^2.206.0",
    "constructs": "^10.0.0",
    "typescript": "~5.6.3",
    "@types/node": "^22.0.0"
  }
}
```

### Why Peer Dependencies for CDK?

- Avoids duplicate CDK versions across consumers
- Consumer controls the CDK version
- Prevents version conflicts at synth time

---

## 9. Testing Strategy

### Construct Unit Tests

```typescript
// cdk-constructs/test/core/lambda.test.ts
import { App, Stack } from "aws-cdk-lib";
import { Template } from "aws-cdk-lib/assertions";
import { LambdaConstruct } from "../../src/core";

describe("LambdaConstruct", () => {
  test("creates Lambda with defaults", () => {
    const app = new App();
    const stack = new Stack(app, "TestStack");

    new LambdaConstruct(stack, "TestLambda", {
      entry: "handler.ts",
    });

    const template = Template.fromStack(stack);
    template.hasResourceProperties("AWS::Lambda::Function", {
      Runtime: "nodejs20.x",
      MemorySize: 512,
    });
  });
});
```

### Integration Tests in Templates

Templates should have integration tests that verify constructs work together correctly in their specific context.

---

## 10. Versioning Strategy

### Semantic Versioning

- **MAJOR**: Breaking changes to construct props or behavior
- **MINOR**: New constructs or additive features
- **PATCH**: Bug fixes, documentation updates

### Deprecation Policy

1. Mark deprecated with `@deprecated` JSDoc
2. Log warning when deprecated construct is used
3. Remove in next major version

### Changelog

Maintain `CHANGELOG.md` with:

- Version number and date
- Breaking changes (with migration guide)
- New features
- Bug fixes

---

## 11. Design Decisions

The following questions were discussed and decisions made:

### 1. Construct Granularity

**Decision: Both L2 and L3 style constructs**

| Level                   | Purpose                                    | Example                                             |
| ----------------------- | ------------------------------------------ | --------------------------------------------------- |
| **L2 (Resource-level)** | Single AWS resource with sensible defaults | `LambdaConstruct`, `DynamoTableConstruct`           |
| **L3 (Pattern-level)**  | Combines multiple resources for a use case | `ApiEndpointConstruct` (Lambda + API Gateway + IAM) |

**Rationale:**

- L2 constructs give flexibility when fine-grained control is needed
- L3 constructs reduce boilerplate for common patterns
- Services can choose the level of abstraction they need
- L3 constructs internally compose L2 constructs (composition over inheritance)

**Example:**

```typescript
// L2: Fine-grained control
const lambda = new LambdaConstruct(this, 'MyLambda', { entry: 'handler.ts' });
const api = new ApiGatewayConstruct(this, 'MyApi', { ... });

// L3: Pattern-level (uses L2 internally)
const endpoint = new ApiEndpointConstruct(this, 'GetUsers', {
  method: 'GET',
  path: '/users',
  handlerEntry: 'handler.ts',
});
```

### 2. Feature Flags

**Decision: Yes, use feature flags for optional capabilities**

```typescript
interface ILambdaConstructProps {
  readonly entry: string;
  // Feature flags with sensible defaults
  readonly enableTracing?: boolean; // Default: true in prod, false in dev
  readonly enableMonitoring?: boolean; // Default: true
  readonly enableAlarms?: boolean; // Default: true in prod, false in dev
}
```

**Rationale:**

- Reduces boilerplate for common cases (defaults handle 80% of use cases)
- Allows opt-out when needed (testing, cost savings in dev)
- Makes constructs self-documenting (props show what's configurable)
- Avoids creating multiple construct variants

**Pattern:**

```typescript
constructor(scope: Construct, id: string, props: ILambdaConstructProps) {
  const enableMonitoring = props.enableMonitoring ?? true;

  if (enableMonitoring) {
    new MonitoringConstruct(this, 'Monitoring', { target: this.lambda });
  }
}
```

### 3. Environment-Specific Behavior

**Decision: Props-based configuration, NOT environment detection**

**Rationale (why props-based):**

- Explicit is better than implicit
- Easier to test (no environment mocking needed)
- No magic behavior changes based on hidden state
- Consumer controls the behavior, not the construct

**Rationale (why NOT environment detection):**

- Hidden behavior is hard to debug
- "Works in dev, breaks in prod" scenarios
- Violates principle of least surprise
- Makes constructs harder to test

**Pattern:**

```typescript
// Good: Explicit props
interface ILambdaConstructProps {
  readonly memorySize?: number; // Consumer decides
  readonly reservedConcurrency?: number; // Consumer decides
  readonly logRetentionDays?: number; // Consumer decides
}

// Service config handles environment differences
// svc-users-template/src/config/dev.ts
export const config: IAuthServiceConfig = {
  lambda: {
    memorySize: 256, // Smaller in dev
    logRetentionDays: 7, // Shorter in dev
  },
};

// svc-users-template/src/config/prod.ts
export const config: IAuthServiceConfig = {
  lambda: {
    memorySize: 1024, // Larger in prod
    logRetentionDays: 90, // Longer in prod
  },
};
```

**Exception:** Some defaults can be environment-aware as a convenience, but should be overridable:

```typescript
const defaultLogRetention =
  props.logRetentionDays ?? (props.config.envName === "prod" ? 90 : 7);
```

### 4. Error Handling

**Decision: Fail fast with clear, actionable error messages**

**Pattern:**

```typescript
constructor(scope: Construct, id: string, props: IMyConstructProps) {
  // Validate required props immediately
  if (!props.tableName) {
    throw new Error(
      `[${id}] tableName is required. ` +
      `Provide it via props or ensure config.database.tableName is set.`
    );
  }

  // Validate prop combinations
  if (props.enableEncryption && !props.kmsKeyArn) {
    throw new Error(
      `[${id}] kmsKeyArn is required when enableEncryption is true.`
    );
  }

  // Validate prop values
  if (props.memorySize && (props.memorySize < 128 || props.memorySize > 10240)) {
    throw new Error(
      `[${id}] memorySize must be between 128 and 10240 MB. Got: ${props.memorySize}`
    );
  }
}
```

**Rationale:**

- Fail at synth time, not deploy time
- Include construct ID in error for easy debugging
- Explain what's wrong AND how to fix it
- Validate early, before creating any resources

**Optional validation helper:**

```typescript
// cdk-constructs/src/core/validation.ts
export function requireProp<T>(
  constructId: string,
  propName: string,
  value: T | undefined
): T {
  if (value === undefined || value === null) {
    throw new Error(`[${constructId}] ${propName} is required.`);
  }
  return value;
}
```

### 5. Naming Conventions

**Decision: Consistent suffixes and prefixes**

| Type              | Convention           | Example                                  |
| ----------------- | -------------------- | ---------------------------------------- |
| Construct classes | `XxxConstruct`       | `LambdaConstruct`, `ApiGatewayConstruct` |
| Props interfaces  | `IXxxConstructProps` | `ILambdaConstructProps`                  |
| Config interfaces | `IXxxConfig`         | `IServiceConfig`, `IAuthConfig`          |
| Type aliases      | `TXxx`               | `TEnvironment`, `TUserRole`              |
| Internal helpers  | No suffix            | `validateProps`, `createDefaultPolicy`   |

**File naming:**

```
construct.ts     # The construct class
types.ts         # Interfaces and types for this construct
helpers.ts       # Internal helper functions (if needed)
index.ts         # Exports
```

**Rationale:**

- Matches existing codebase conventions (from templates)
- `I` prefix for interfaces is TypeScript convention in CDK ecosystem
- `Construct` suffix makes it clear what the class is
- Distinguishes props (construct-specific) from config (service-wide)

---

## 12. Success Criteria

### Immediate Goals

- [ ] Single source of truth for infrastructure patterns
- [ ] Templates consume shared constructs successfully
- [ ] No code duplication between templates
- [ ] Clear separation of concerns

### Long-term Goals

- [ ] Easy to add new constructs
- [ ] Easy to update existing constructs
- [ ] Clear documentation for all patterns
- [ ] Version-controlled with proper semver

---

## 13. Next Steps

1. **Review this plan** - Discuss and refine approach
2. **Create cdk-constructs project** - Set up directory structure
3. **Create template copies** - `svc-users-template-v2`, `svc-ai-template-v2`
4. **Begin extraction** - Start with core constructs
5. **Iterate** - Extract, test, refine

---

## Appendix A: Directory Structure (Proposed)

```
cdk-constructs/
├── src/
│   ├── index.ts                    # Main exports
│   ├── core/
│   │   ├── index.ts                # Core exports
│   │   ├── types.ts                # Core interfaces
│   │   ├── api-gateway/
│   │   │   ├── construct.ts
│   │   │   └── types.ts
│   │   ├── lambda/
│   │   │   ├── construct.ts
│   │   │   └── types.ts
│   │   ├── dynamodb/
│   │   │   ├── construct.ts
│   │   │   ├── single-table/
│   │   │   └── faux-sql/
│   │   ├── monitoring/
│   │   ├── queues/
│   │   ├── scheduler/
│   │   ├── storage/
│   │   ├── secrets/
│   │   ├── events/
│   │   └── pipeline/
│   ├── auth/
│   │   ├── index.ts
│   │   ├── types.ts
│   │   ├── cognito/
│   │   │   ├── user-pool/
│   │   │   └── identity-pool/
│   │   ├── iam-roles/
│   │   ├── authorizer/
│   │   └── well-known/
│   └── ai/
│       ├── index.ts
│       ├── types.ts
│       ├── bedrock/
│       ├── vector-db/
│       ├── agents/
│       └── knowledge-base/
├── docs/
│   ├── README.md
│   ├── getting-started.md
│   ├── core/
│   ├── auth/
│   └── ai/
├── test/
│   ├── core/
│   ├── auth/
│   └── ai/
├── package.json
├── tsconfig.json
├── CHANGELOG.md
└── README.md
```

---

## Appendix B: infra-contracts Integration

The `infra-contracts` package defines the SSM parameter contracts:

```typescript
// infra-contracts/src/users-ms/types.ts
export interface AuthBindings {
  userPoolId: string;
  userPoolArn?: string;
  userPoolProviderName?: string;
  userPoolClientId?: string;
  identityPoolId?: string;
}

export interface IamBindings {
  merchantRoleArn: string; // TODO: Update to userRoleArn
  authenticatedRoleArn?: string;
  unauthenticatedRoleArn?: string;
}
```

### How They Work Together

1. **Users service** (using `cdk-constructs/auth`) creates Cognito resources
2. **Users service** publishes SSM parameters (using `cdk-constructs/core/ssm-publications`)
3. **Other services** read SSM parameters (using `cdk-constructs/core/ssm-bindings`)
4. **Cross-service type safety** provided by `infra-contracts` types

### Template Dependencies

```json
{
  "devDependencies": {
    "@thiru-ai-labs/cdk-constructs": "file:../cdk-constructs",
    "@thiru-ai-labs/infra-contracts": "file:../infra-contracts"
  }
}
```
