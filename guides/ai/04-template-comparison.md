# Template Comparison: svc-merchants vs microservice-template

**Purpose:** Document differences between the two template sources to inform the AI agentic template strategy.

**Status:** Draft  
**Created:** November 29, 2025  
**Last Updated:** November 29, 2025

---

## Overview

We have two template sources:

| Template                  | Location                     | Purpose                           |
| ------------------------- | ---------------------------- | --------------------------------- |
| **svc-merchants**         | `svc-users-template/` (copy) | Users/auth service with Cognito   |
| **microservice-template** | `svc-ai-template/` (copy)    | Generic microservice without auth |

---

## Architecture Comparison

### svc-merchants (Users Template)

```
ServiceStack
├── SsmBindingsConstruct (external config bindings)
├── MonitorConstruct (CloudWatch, SNS)
├── DatabaseConstruct (DynamoDB - Faux-SQL OR Single-Table)
├── AuthConstruct ⭐ UNIQUE
│   ├── UserPoolConstruct (Cognito User Pool)
│   ├── IdentityPoolConstruct (Cognito Identity Pool)
│   └── UserGroupsConstruct (merchant, customer groups)
├── IamConstruct ⭐ UNIQUE (roles for authenticated users)
├── PermissionsConstruct (OAuth scopes)
├── ApiConstruct (API Gateway + Lambda)
└── SsmPublicationsConstruct (publish service configs)
```

### microservice-template (Generic Template)

```
ServiceStack
├── SsmBindingsConstruct (external config bindings)
├── MonitorConstruct (CloudWatch, SNS) ⭐ ENHANCED
├── DatabaseConstruct (DynamoDB - Single-Table only)
├── StorageConstruct ⭐ UNIQUE (S3 bucket)
├── QueueConstruct ⭐ UNIQUE (SQS + Lambda consumer)
├── StreamConsumerConstruct ⭐ UNIQUE (DynamoDB Streams)
├── SchedulerConstruct ⭐ UNIQUE (EventBridge scheduled tasks)
├── SecretsConstruct ⭐ UNIQUE (Secrets Manager)
├── PermissionsConstruct (OAuth scopes)
└── ApiConstruct (API Gateway + Lambda)
```

---

## Feature Comparison

| Feature                   | svc-merchants | microservice-template | Notes                          |
| ------------------------- | ------------- | --------------------- | ------------------------------ |
| **Cognito User Pool**     | ✅            | ❌                    | Auth service only              |
| **Cognito Identity Pool** | ✅            | ❌                    | AWS credentials for users      |
| **User Groups**           | ✅            | ❌                    | Role-based access              |
| **IAM Roles for Users**   | ✅            | ❌                    | Authenticated user permissions |
| **S3 Storage**            | ❌            | ✅                    | Asset storage                  |
| **SQS Queues**            | ❌            | ✅                    | Async processing               |
| **DynamoDB Streams**      | ❌            | ✅                    | Change data capture            |
| **EventBridge Scheduler** | ❌            | ✅                    | Scheduled tasks                |
| **Secrets Manager**       | ❌            | ✅                    | Secret provisioning            |
| **Faux-SQL Database**     | ✅            | ❌                    | Multiple tables option         |
| **Single-Table Database** | ✅            | ✅                    | Both support                   |
| **SSM Publications**      | ✅            | ⚠️ Commented          | Service discovery              |
| **Lambda Monitoring**     | Basic         | ✅ Enhanced           | Per-function alarms            |
| **DynamoDB Monitoring**   | Basic         | ✅ Enhanced           | Throttle alarms                |
| **S3 Monitoring**         | ❌            | ✅                    | Failure alarms                 |

---

## Configuration Comparison

### svc-merchants Config Structure

```typescript
// config/default.ts aggregates from:
// - config/service.ts
// - config/database.ts
// - config/api.ts
// - config/resources.ts
// - config/features.ts
// - config/github.ts
// - config/aws.ts
// - config/schema.ts (Zod validation)

interface IConfig {
  envName: string;
  accountId: string;
  region: string;
  service: IServiceConfig;
  database: IDatabaseConfig; // Detailed DB config
  api: IApiConfig; // Detailed API config
  resources: IResourcesConfig;
  features?: IFeaturesConfig;
  github?: IGitHubConfig;
  aws?: IAwsConfig;
}
```

### microservice-template Config Structure

```typescript
// config/default.ts - Single file with inline Zod schema

interface IConfig {
  envName: string;
  accountId: string;
  region: string;
  service: { name: string; displayName: string };
  github?: { repo: string; branch: string; codestarConnectionId: string };
  aws?: { region: string; profile?: string };
  endpoints?: {
    /* LocalStack endpoints */
  };
  resources: { tablePrefix; bucketPrefix; functionPrefix; apiPrefix };
  features?: {
    permissionsEnabled?: boolean;
    queuesEnabled?: boolean;
    monitoringLambdaErrorsEnabled?: boolean;
    monitoringDynamoThrottlesEnabled?: boolean;
    monitoringS3FailuresEnabled?: boolean;
    dynamodbStreamsEnabled?: boolean;
    schedulerEnabled?: boolean;
    secretsManagerEnabled?: boolean;
  };
  development?: {
    /* dev settings */
  };
}
```

### Key Config Differences

| Aspect                    | svc-merchants               | microservice-template         |
| ------------------------- | --------------------------- | ----------------------------- |
| **Structure**             | Modular (separate files)    | Single file                   |
| **Zod Schema**            | Separate schema.ts          | Inline in default.ts          |
| **Database Config**       | Detailed (approach, tables) | Simple (prefix only)          |
| **API Config**            | Detailed (CORS, stages)     | Minimal                       |
| **Feature Flags**         | Basic                       | More granular                 |
| **Template Placeholders** | Implicit                    | Explicit (`{{SERVICE_NAME}}`) |

---

## Feature Flags Comparison

### svc-merchants Features

```typescript
features?: {
  permissionsEnabled?: boolean;
}
```

### microservice-template Features

```typescript
features?: {
  permissionsEnabled?: boolean;
  queuesEnabled?: boolean;
  monitoringLambdaErrorsEnabled?: boolean;
  monitoringDynamoThrottlesEnabled?: boolean;
  monitoringS3FailuresEnabled?: boolean;
  dynamodbStreamsEnabled?: boolean;
  schedulerEnabled?: boolean;
  secretsManagerEnabled?: boolean;
}
```

---

## Documentation Comparison

### svc-merchants Docs

```
docs/
├── implementation/
│   ├── README.md
│   ├── adding-endpoints-part-1-lambda-handlers.md
│   ├── adding-endpoints-part-2-api-gateway.md
│   ├── authentication.md
│   ├── authorization.md
│   ├── configuration-management/
│   │   ├── README.md
│   │   ├── environment-configuration.md
│   │   ├── service-configuration.md
│   │   └── ... (9 files)
│   ├── data-access.md
│   ├── database-setup.md
│   ├── deployment.md
│   └── ... (17 files total)
├── testing/
│   ├── handler-testing-guide.md
│   ├── cdk-template-testing-guide.md
│   └── ... (8 files)
└── DOCUMENTATION-MAP.md
```

### microservice-template Docs

```
docs/
├── guides/
│   ├── README.md
│   ├── async-processing.md
│   ├── monitoring.md
│   ├── streams.md
│   ├── scheduler.md
│   └── secrets.md
└── ... (19 items total)
```

---

## Merge Strategy for AI Template

### What to Keep from svc-merchants (→ svc-users-template)

1. **Cognito constructs** - Essential for auth service
2. **IAM constructs** - User role management
3. **Modular config structure** - Better organization
4. **Comprehensive documentation** - More complete guides
5. **Faux-SQL database option** - Flexibility

### What to Add from microservice-template (→ svc-ai-template)

1. **StorageConstruct** - S3 for agent artifacts
2. **QueueConstruct** - Async agent processing
3. **StreamConsumerConstruct** - Event-driven patterns
4. **SchedulerConstruct** - Scheduled agent runs
5. **SecretsConstruct** - API key management
6. **Enhanced monitoring** - Per-resource alarms
7. **Granular feature flags** - More control
8. **Template placeholders** - Explicit `{{PLACEHOLDER}}` pattern

### What to Add for AI Agents (Both Templates)

1. **AgentsConstruct** - LangGraph execution
2. **OrchestrationConstruct** - Step Functions
3. **PaymentsConstruct** - Stripe integration
4. **VectorDbConstruct** - Pinecone/OpenSearch

---

## Recommended Template Structure

### svc-users-template (Auth Service)

Keep as-is with minor updates:

- Add explicit template placeholders
- Update docs to be generic
- Remove merchant-specific code

### svc-ai-template (Generic AI Microservice)

Merge best of both:

```
ServiceStack
├── SsmBindingsConstruct
├── MonitorConstruct (enhanced from microservice-template)
├── DatabaseConstruct (add Faux-SQL option from svc-merchants)
├── StorageConstruct (from microservice-template)
├── QueueConstruct (from microservice-template)
├── StreamConsumerConstruct (from microservice-template)
├── SchedulerConstruct (from microservice-template)
├── SecretsConstruct (from microservice-template)
├── PermissionsConstruct
├── ApiConstruct
├── AgentsConstruct ⭐ NEW
├── OrchestrationConstruct ⭐ NEW
├── PaymentsConstruct ⭐ NEW
└── VectorDbConstruct ⭐ NEW
```

Config structure:

- Use modular config from svc-merchants
- Add feature flags from microservice-template
- Add explicit template placeholders

Documentation:

- Use comprehensive structure from svc-merchants
- Add guides for new constructs from microservice-template
- Add AI-specific guides

---

## Next Steps

1. **svc-users-template:**

   - Remove merchant-specific code
   - Add template placeholders
   - Update documentation

2. **svc-ai-template:**
   - Add Faux-SQL database option
   - Add modular config structure
   - Add comprehensive documentation
   - Add AI-specific constructs
   - Add template placeholders

---

## Related Documents

- [Backend Infrastructure Adoption](./01-backend-infrastructure-adoption.md)
- [Template Conversion Plan](./02-template-conversion-plan.md)
- [Template Conversion Progress](./03-template-conversion-progress.md)
