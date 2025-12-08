# Shared Constructs Implementation Tracker

**Last Updated:** December 7, 2025  
**Status:** Config Strategy Defined - Construct Extraction In Progress

---

## Overview

This document tracks the implementation progress of the shared CDK constructs initiative. It serves as a reference for what has been done and what remains to be done.

---

## Phase Status

| Phase                  | Status         | Notes                                 |
| ---------------------- | -------------- | ------------------------------------- |
| Phase 1: Setup         | ✅ Complete    | Project structure created             |
| Phase 2: Extract Core  | 🔄 In Progress | Direct copy approach - fixing imports |
| Phase 3: Extract Auth  | 🔄 In Progress | Direct copy approach - fixing imports |
| Phase 4: Extract AI    | 🔄 In Progress | Refactored structure                  |
| Phase 5: Documentation | 🔄 In Progress | Updating for corrected approach       |
| Phase 6: Validation    | 🔲 Not Started | Test with svc-merchants-v2            |

### Corrected Extraction Approach

The shared constructs are **direct copies** of working constructs from `svc-merchants/lib/`, preserving internal structure and sub-constructs:

```
cdk-constructs/src/
├── core/
│   ├── api/              # ApiConstruct + stage/, authorization/ (NOT endpoints/)
│   ├── db/               # DatabaseConstruct + faux-sql/, single-table/
│   ├── monitor/          # MonitorConstruct + ses/, api/
│   ├── ssm-bindings/     # SsmBindingsConstruct + website/, monitor/
│   ├── ssm-publications/ # SsmPublicationsConstruct + auth/, iam/
│   ├── permissions/      # PermissionsConstruct + oauth/, policies/, resource-server/
│   ├── events/           # EventsConstruct
│   ├── pipeline/         # PipelineConstruct
│   └── utils/            # Shared utilities (ssm-bindings/, topic/)
├── auth/
│   ├── cognito/          # AuthConstruct + user-pool/, identity-pool/, user-groups/, roles/
│   └── iam/              # IamConstruct + roles/
└── ai/
    ├── agents/           # AgentsConstruct
    └── vector-db/        # Pinecone, OpenSearch constructs
```

**Key principle**: Service-specific code (like `api/endpoints/`) stays in the service template. The `ApiConstruct` exposes `apiProps` for services to create their own endpoints.

### Remaining Work (Phase 2-4)

| Task                                             | Status | Notes                                                       |
| ------------------------------------------------ | ------ | ----------------------------------------------------------- |
| Remove `api/endpoints/` from cdk-constructs      | 🔲     | Service-specific, stays in templates                        |
| Modify `ApiConstruct` to expose `apiProps`       | 🔲     | Remove EndpointsConstruct instantiation                     |
| Fix import paths in all constructs               | 🔄     | Replace `#lib/*`, `#config/*` with relative - IN PROGRESS   |
| Define config interfaces per construct           | 🔄     | Created `IBaseConfig`, `IDatabaseConfig` - MORE NEEDED      |
| Move types to separate `types.ts` files          | 🔄     | Created `src/types.ts`, `core/db/types.ts` - MORE NEEDED    |
| Update construct props to use typed config       | 🔄     | Updated `DatabaseConstruct` - MORE NEEDED                   |
| Update `index.ts` aggregators                    | 🔄     | Updated `core/index.ts` for db - BROKEN IMPORTS TO FIX      |
| Test build of cdk-constructs                     | 🔲     | `npm run build` - blocked by import fixes                   |
| Update svc-merchants-v2 to use shared constructs | ✅     | Rewrote to match svc-merchants pattern (uses local for now) |

**Legend:** ✅ Complete | 🔄 In Progress | 🔲 Not Started | ⏸️ Blocked

### Current Session Progress (Dec 5-7, 2025)

**Completed:**

1. Updated `01-shared-constructs-plan.md` Section 4 with comprehensive config handling strategy
2. Created `cdk-constructs/src/types.ts` with `IBaseConfig`, `IServiceConfig`, `IResourcesConfig`
3. Created `cdk-constructs/src/core/db/types.ts` with `IDatabaseConfig`, `TDatabaseConstructConfig`
4. Updated `DatabaseConstruct` and sub-constructs to use new types
5. Rewrote `svc-merchants-v2/lib/service-stack.ts` to match original `svc-merchants` pattern
6. Validated `svc-merchants-v2` main source compiles (only test file issues)
7. Extracted missing constructs from `microservice-template/lib/` to `cdk-constructs/src/core/`:
   - `storage/` - StorageConstruct (S3 bucket)
   - `queues/` - QueueConstruct (SQS with DLQ)
   - `scheduler/` - SchedulerConstruct (EventBridge scheduled Lambda)
   - `secrets/` - SecretsConstruct (Secrets Manager)
   - `streams/` - DynamoDbStreamConsumerConstruct (DynamoDB Streams)

**In Progress:**

- Creating `index.ts` files for new constructs (streams/ needs index.ts)
- Fixing broken imports in `cdk-constructs/src/core/index.ts`

**Next Steps:**

1. Create `streams/index.ts`
2. Fix `core/index.ts` to export only existing modules with correct paths
3. Create types files for remaining constructs (monitor, api, ssm-bindings, etc.)
4. Update remaining constructs to use typed config pattern
5. Test `cdk-constructs` build

---

## Pre-Implementation Tasks

### Discussion & Planning

- [x] Identify drift between templates (bindings endpoint location)
- [x] Discuss shared constructs approach
- [x] Create planning document (`01-shared-constructs-plan.md`)
- [x] Create tracking document (this file)
- [x] Discuss configuration interface locations (in respective modules)
- [x] Make design decisions (granularity, feature flags, env behavior, error handling, naming)
- [x] Final review before implementation
- [x] Identify all constructs to extract

### Preparation

- [x] Create `cdk-constructs` project directory
- [x] Create `svc-users-template-v2` (copy for testing)
- [x] Create `svc-ai-template-v2` (copy for testing)
- [x] Set up GitHub repo for `cdk-constructs`

---

## Phase 1: Setup

### Tasks

| Task                               | Status | Notes                      |
| ---------------------------------- | ------ | -------------------------- |
| Create `cdk-constructs/` directory | ✅     | Done                       |
| Initialize package.json            | ✅     | With subpath exports       |
| Set up TypeScript config           | ✅     | NodeNext module            |
| Create directory structure         | ✅     | src/core, src/auth, src/ai |
| Set up exports (index.ts files)    | ✅     | All modules have index.ts  |
| Add build scripts                  | ✅     | build, clean, test         |
| Create README.md                   | ✅     | With usage examples        |
| Create CHANGELOG.md                | ✅     | Initial version            |
| Install dependencies               | ✅     | npm install successful     |
| Build package                      | ✅     | tsc builds to dist/        |
| Create svc-users-template-v2       | ✅     | Copy with shared deps      |
| Create svc-ai-template-v2          | ✅     | Copy with shared deps      |
| Initialize git repo                | ✅     | Done                       |
| Create GitHub repo                 | ✅     | Done                       |

### Files to Create

```
cdk-constructs/
├── src/
│   ├── index.ts
│   ├── core/index.ts
│   ├── auth/index.ts
│   └── ai/index.ts
├── package.json
├── tsconfig.json
├── README.md
└── CHANGELOG.md
```

---

## Phase 2: Extract Core Constructs

### Constructs to Extract

| Construct                       | Source             | Status | Notes             |
| ------------------------------- | ------------------ | ------ | ----------------- |
| SingleTableConstruct            | svc-users-template | ✅     | core/dynamodb     |
| FauxSqlConstruct                | svc-users-template | ✅     | core/dynamodb     |
| SSM Helpers                     | Both templates     | ✅     | core/ssm          |
| LambdaErrorAlarmsConstruct      | Both templates     | ✅     | core/monitoring   |
| DynamoDbThrottleAlarmsConstruct | Both templates     | ✅     | core/monitoring   |
| StorageConstruct                | Both templates     | ✅     | core/storage      |
| SecretsConstruct                | Both templates     | ✅     | core/secrets      |
| QueueConstruct                  | Both templates     | ✅     | core/queues       |
| SchedulerConstruct              | Both templates     | ✅     | core/scheduler    |
| DynamoDbStreamConsumerConstruct | svc-users-template | ✅     | core/streams      |
| ApiStageConstruct               | Both templates     | ✅     | core/api          |
| ApiAlarmsConstruct              | Both templates     | ✅     | core/api          |
| TopicConstruct                  | Both templates     | ✅     | core/events       |
| PipelineConstruct               | Both templates     | 🔲     | Complex, deferred |

### Configuration Interfaces

| Interface         | Status | Notes      |
| ----------------- | ------ | ---------- |
| IServiceConfig    | ✅     | core/types |
| IAwsConfig        | ✅     | core/types |
| ILambdaConfig     | ✅     | core/types |
| IDynamoDbConfig   | ✅     | core/types |
| IMonitoringConfig | ✅     | core/types |

### Deferred Constructs

| Construct         | Reason for Deferral                                                                                                                                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PipelineConstruct | Complex CI/CD construct tightly coupled to service-specific configuration (GitHub repo, CodeStar connection, stage definitions). Requires significant refactoring to make generic. Consider extracting common pipeline patterns later. |

---

## Phase 3: Extract Auth Constructs

### Constructs to Extract

| Construct                  | Source             | Status | Notes            |
| -------------------------- | ------------------ | ------ | ---------------- |
| UserPoolConstruct          | svc-users-template | ✅     | auth/cognito     |
| IdentityPoolConstruct      | svc-users-template | ✅     | auth/cognito     |
| UserGroupsConstruct        | svc-users-template | ✅     | auth/cognito     |
| CognitoAuthorizerConstruct | svc-users-template | ✅     | auth/cognito     |
| IdentityPoolRolesConstruct | svc-users-template | ✅     | auth/iam         |
| WellKnownBindingsConstruct | svc-users-template | 🔲     | Service-specific |
| PermissionsConstruct       | svc-users-template | 🔲     | Service-specific |

### Configuration Interfaces

| Interface            | Status | Notes              |
| -------------------- | ------ | ------------------ |
| IAuthServiceConfig   | ✅     | auth/types         |
| ICognitoConfig       | ✅     | auth/types         |
| IPasswordPolicy      | ✅     | auth/cognito/types |
| IOAuthConfig         | ✅     | auth/cognito/types |
| IUserGroupDefinition | ✅     | auth/cognito/types |

### Deferred Constructs

| Construct                  | Reason for Deferral                                                                                                                                                                                                                                              |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WellKnownBindingsConstruct | Tightly coupled to service-specific SSM parameter paths and external service discovery patterns. The SSM path structure varies per service and environment. Consider extracting if a common pattern emerges across multiple services.                            |
| PermissionsConstruct       | Defines service-specific OAuth scopes and authorization rules (e.g., users/read, users/write). Each service has unique permission requirements. The `CognitoAuthorizerConstruct` provides the generic authorizer; permissions remain service-specific by design. |

---

## Phase 4: Extract AI Constructs

### Constructs to Extract

| Construct              | Source          | Status | Notes             |
| ---------------------- | --------------- | ------ | ----------------- |
| AgentsConstruct        | svc-ai-template | ✅     | ai/agents         |
| PineconeConstruct      | svc-ai-template | ✅     | ai/vector-db      |
| OpenSearchConstruct    | svc-ai-template | ✅     | ai/vector-db      |
| KnowledgeBaseConstruct | svc-ai-template | 🔲     | Complex, deferred |

### Configuration Interfaces

| Interface         | Status | Notes              |
| ----------------- | ------ | ------------------ |
| IAiServiceConfig  | ✅     | ai/types           |
| IBedrockConfig    | ✅     | ai/types           |
| IVectorDbConfig   | ✅     | ai/types           |
| TVectorDbProvider | ✅     | ai/vector-db/types |
| TLlmProvider      | ✅     | ai/agents          |

### Deferred Constructs

| Construct              | Reason for Deferral                                                                                                                                                                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| KnowledgeBaseConstruct | AWS Bedrock Knowledge Base is a complex managed service with many configuration options (data sources, chunking strategies, embeddings). The current template doesn't have a full implementation. Consider adding when Knowledge Base patterns mature. |

---

## Phase 5: Documentation Consolidation

### Documentation to Move

| Document              | Source         | Target               | Status | Notes                               |
| --------------------- | -------------- | -------------------- | ------ | ----------------------------------- |
| Implementation guides | Both templates | cdk-constructs/docs/ | ⏸️     | Keep in templates, reference shared |
| Testing guides        | Both templates | cdk-constructs/docs/ | ⏸️     | Keep in templates, reference shared |
| Architecture docs     | Both templates | cdk-constructs/docs/ | ⏸️     | Keep in templates, reference shared |
| Configuration guides  | Both templates | cdk-constructs/docs/ | ⏸️     | Keep in templates, reference shared |

**Note:** Template-specific docs remain in templates. Shared construct docs are in `cdk-constructs/docs/`.

### Documentation Created

| Document              | Status | Location                         |
| --------------------- | ------ | -------------------------------- |
| README                | ✅     | `docs/README.md`                 |
| Getting Started       | ✅     | `docs/getting-started.md`        |
| Core Constructs Guide | ✅     | `docs/guides/core-constructs.md` |
| Auth Constructs Guide | ✅     | `docs/guides/auth-constructs.md` |
| AI Constructs Guide   | ✅     | `docs/guides/ai-constructs.md`   |
| Migration Guide       | ✅     | `docs/guides/migration-guide.md` |

---

## Phase 6: Validation & Cleanup

### Validation Tasks

| Task                                              | Status | Notes |
| ------------------------------------------------- | ------ | ----- |
| Test svc-users-template-v2 with shared constructs | 🔲     |       |
| Test svc-ai-template-v2 with shared constructs    | 🔲     |       |
| Run all unit tests                                | 🔲     |       |
| Run CDK synth                                     | 🔲     |       |
| Deploy to dev environment                         | 🔲     |       |

### Cleanup Tasks

| Task                                     | Status | Notes |
| ---------------------------------------- | ------ | ----- |
| Remove duplicate code from templates     | 🔲     |       |
| Update template documentation            | 🔲     |       |
| Rename v2 templates to replace originals | 🔲     |       |
| Update infra-contracts references        | 🔲     |       |

---

## Issues & Blockers

| Issue    | Status | Resolution |
| -------- | ------ | ---------- |
| None yet | -      | -          |

---

## Decisions Made

| Date       | Decision                                                   | Rationale                                                                        |
| ---------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 2025-11-30 | Use `@thiru-ai-labs` package scope                         | Business name for Thiru AI Labs                                                  |
| 2025-11-30 | Single monorepo package                                    | Simpler for solo development                                                     |
| 2025-11-30 | Local file imports for dev                                 | Avoid publishing during development                                              |
| 2025-11-30 | Create template copies for testing                         | Don't break existing templates                                                   |
| 2025-11-30 | Config interfaces in respective modules                    | `IServiceConfig` in core, `IAuthServiceConfig` in auth, `IAiServiceConfig` in ai |
| 2025-11-30 | Both L2 and L3 constructs                                  | Flexibility (L2) + convenience (L3)                                              |
| 2025-11-30 | Feature flags for optional capabilities                    | Sensible defaults, opt-out when needed                                           |
| 2025-11-30 | Props-based config, not env detection                      | Explicit > implicit, easier testing                                              |
| 2025-11-30 | Fail fast with actionable errors                           | Synth-time validation, clear messages                                            |
| 2025-11-30 | Naming: `XxxConstruct`, `IXxxConstructProps`, `IXxxConfig` | Matches existing codebase conventions                                            |

---

## Notes & Observations

### November 30, 2025

- Identified drift between templates (bindings endpoint location differs)
- `svc-ai-template` uses `lib/api/endpoints/bindings` (older approach)
- `svc-users-template` uses `lib/api/endpoints/users/well-known` (newer approach)
- Both templates missing `@super-deals/infra-contracts` dependency (was removed during refactoring)
- Need to restore infra-contracts dependency and update to new package name

### Template Differences Identified

| Aspect              | svc-users-template  | svc-ai-template         |
| ------------------- | ------------------- | ----------------------- |
| Bindings location   | `users/well-known/` | `bindings/`             |
| Config location     | `src/config/`       | `config/`               |
| Has auth constructs | Yes                 | No (imports from users) |
| Has AI constructs   | No                  | Yes                     |
| infra-contracts     | Missing             | Missing                 |

---

## Change Log

| Date       | Change                                           | Files Affected                                                                                                    |
| ---------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| 2025-11-30 | Created planning documents                       | `01-shared-constructs-plan.md`, `02-implementation-tracker.md`                                                    |
| 2025-11-30 | Updated config interface locations               | `01-shared-constructs-plan.md`                                                                                    |
| 2025-11-30 | Added design decisions (replaced open questions) | `01-shared-constructs-plan.md`                                                                                    |
| 2025-11-30 | Updated decisions log                            | `02-implementation-tracker.md`                                                                                    |
| 2025-11-30 | Created cdk-constructs package                   | `cdk-constructs/` (package.json, tsconfig.json, README, CHANGELOG)                                                |
| 2025-11-30 | Created core module with IServiceConfig          | `cdk-constructs/src/core/`                                                                                        |
| 2025-11-30 | Created auth module with IAuthServiceConfig      | `cdk-constructs/src/auth/`                                                                                        |
| 2025-11-30 | Created ai module with IAiServiceConfig          | `cdk-constructs/src/ai/`                                                                                          |
| 2025-11-30 | Created svc-users-template-v2                    | Copy of users template with shared deps                                                                           |
| 2025-11-30 | Created svc-ai-template-v2                       | Copy of AI template with shared deps                                                                              |
| 2025-11-30 | Extracted SingleTableConstruct                   | `cdk-constructs/src/core/dynamodb/single-table/`                                                                  |
| 2025-11-30 | Extracted FauxSqlConstruct                       | `cdk-constructs/src/core/dynamodb/faux-sql/`                                                                      |
| 2025-11-30 | Extracted SSM helpers                            | `cdk-constructs/src/core/ssm/`                                                                                    |
| 2025-11-30 | Extracted LambdaErrorAlarmsConstruct             | `cdk-constructs/src/core/monitoring/alarms/`                                                                      |
| 2025-11-30 | Extracted DynamoDbThrottleAlarmsConstruct        | `cdk-constructs/src/core/monitoring/alarms/`                                                                      |
| 2025-12-01 | Extracted StorageConstruct                       | `cdk-constructs/src/core/storage/`                                                                                |
| 2025-12-01 | Extracted SecretsConstruct                       | `cdk-constructs/src/core/secrets/`                                                                                |
| 2025-12-01 | Extracted QueueConstruct                         | `cdk-constructs/src/core/queues/`                                                                                 |
| 2025-12-01 | Extracted SchedulerConstruct                     | `cdk-constructs/src/core/scheduler/`                                                                              |
| 2025-12-01 | Extracted DynamoDbStreamConsumerConstruct        | `cdk-constructs/src/core/streams/`                                                                                |
| 2025-12-01 | Extracted ApiStageConstruct                      | `cdk-constructs/src/core/api/`                                                                                    |
| 2025-12-01 | Extracted ApiAlarmsConstruct                     | `cdk-constructs/src/core/api/`                                                                                    |
| 2025-12-01 | Extracted TopicConstruct                         | `cdk-constructs/src/core/events/`                                                                                 |
| 2025-12-01 | Extracted UserPoolConstruct                      | `cdk-constructs/src/auth/cognito/`                                                                                |
| 2025-12-01 | Extracted IdentityPoolConstruct                  | `cdk-constructs/src/auth/cognito/`                                                                                |
| 2025-12-01 | Extracted UserGroupsConstruct                    | `cdk-constructs/src/auth/cognito/`                                                                                |
| 2025-12-01 | Extracted CognitoAuthorizerConstruct             | `cdk-constructs/src/auth/cognito/`                                                                                |
| 2025-12-01 | Extracted IdentityPoolRolesConstruct             | `cdk-constructs/src/auth/iam/`                                                                                    |
| 2025-12-01 | Extracted AgentsConstruct                        | `cdk-constructs/src/ai/agents/`                                                                                   |
| 2025-12-01 | Extracted PineconeConstruct                      | `cdk-constructs/src/ai/vector-db/`                                                                                |
| 2025-12-01 | Extracted OpenSearchConstruct                    | `cdk-constructs/src/ai/vector-db/`                                                                                |
| 2025-12-01 | Created documentation README                     | `cdk-constructs/docs/README.md`                                                                                   |
| 2025-12-01 | Created Getting Started guide                    | `cdk-constructs/docs/getting-started.md`                                                                          |
| 2025-12-01 | Created Core Constructs guide                    | `cdk-constructs/docs/guides/core-constructs.md`                                                                   |
| 2025-12-01 | Created Auth Constructs guide                    | `cdk-constructs/docs/guides/auth-constructs.md`                                                                   |
| 2025-12-01 | Created AI Constructs guide                      | `cdk-constructs/docs/guides/ai-constructs.md`                                                                     |
| 2025-12-01 | Created Migration guide                          | `cdk-constructs/docs/guides/migration-guide.md`                                                                   |
| 2025-12-01 | **Refactored to L3 pattern**                     | Created proper L3 facade constructs                                                                               |
| 2025-12-01 | Created L3 DatabaseConstruct                     | `cdk-constructs/src/core/database/construct.ts`                                                                   |
| 2025-12-01 | Created L3 MonitorConstruct                      | `cdk-constructs/src/core/monitor/construct.ts`                                                                    |
| 2025-12-01 | Created L3 ApiConstruct                          | `cdk-constructs/src/core/api/construct.ts`                                                                        |
| 2025-12-01 | Created L3 AuthConstruct                         | `cdk-constructs/src/auth/construct.ts`                                                                            |
| 2025-12-01 | Created L3 IamConstruct                          | `cdk-constructs/src/auth/iam-construct.ts`                                                                        |
| 2025-12-01 | Created L3 SsmBindingsConstruct                  | `cdk-constructs/src/core/ssm-bindings/construct.ts`                                                               |
| 2025-12-01 | Created L3 SsmPublicationsConstruct              | `cdk-constructs/src/core/ssm-publications/construct.ts`                                                           |
| 2025-12-01 | Updated exports to L3 constructs only            | Internal constructs no longer exported                                                                            |
| 2025-12-01 | Moved old constructs to \_internal/              | `core/_internal/`, `auth/_internal/`                                                                              |
| 2025-12-01 | Deleted \_internal/ folders                      | Old constructs no longer needed                                                                                   |
| 2025-12-01 | Updated all documentation for L3 pattern         | README, getting-started, core/auth/migration guides                                                               |
| 2025-12-03 | **Corrected extraction approach**                | Direct copy from svc-merchants, not rewritten constructs                                                          |
| 2025-12-03 | Removed incorrectly created L3 constructs        | Deleted files that didn't match svc-merchants patterns                                                            |
| 2025-12-03 | Copied core constructs from svc-merchants/lib    | `api/`, `db/`, `monitor/`, `ssm-bindings/`, `ssm-publications/`, `permissions/`, `events/`, `pipeline/`, `utils/` |
| 2025-12-03 | Copied auth constructs from svc-merchants/lib    | `auth/` → `cognito/`, `iam/` → `iam/`                                                                             |
| 2025-12-03 | Refactored ai module structure                   | `agents/`, `vector-db/`                                                                                           |
| 2025-12-03 | Updated plan Section 3 (What Goes Where)         | Added extraction approach, API endpoints pattern                                                                  |
| 2025-12-03 | Updated plan Section 4 (Config Management)       | Construct owns config interface, service satisfies it                                                             |
| 2025-12-03 | Updated tracker with corrected approach          | Phase status, architecture diagram, changelog                                                                     |
| 2025-12-05 | Updated plan Section 4 with config categories    | Base config, construct-specific, pipeline/deferred categories                                                     |
| 2025-12-05 | Created `cdk-constructs/src/types.ts`            | `IBaseConfig`, `IServiceConfig`, `IResourcesConfig`                                                               |
| 2025-12-05 | Created `cdk-constructs/src/core/db/types.ts`    | `IDatabaseConfig`, `TDatabaseConstructConfig`, related types                                                      |
| 2025-12-05 | Updated DatabaseConstruct to use typed config    | `db/construct.ts`, `db/faux-sql/construct.ts`, `db/single-table/construct.ts`                                     |
| 2025-12-05 | Rewrote svc-merchants-v2/lib/service-stack.ts    | Now matches svc-merchants pattern exactly, uses local constructs                                                  |
| 2025-12-07 | Extracted StorageConstruct                       | `cdk-constructs/src/core/storage/` from microservice-template                                                     |
| 2025-12-07 | Extracted QueueConstruct                         | `cdk-constructs/src/core/queues/` from microservice-template                                                      |
| 2025-12-07 | Extracted SchedulerConstruct                     | `cdk-constructs/src/core/scheduler/` from microservice-template                                                   |
| 2025-12-07 | Extracted SecretsConstruct                       | `cdk-constructs/src/core/secrets/` from microservice-template                                                     |
| 2025-12-07 | Extracted DynamoDbStreamConsumerConstruct        | `cdk-constructs/src/core/streams/` from microservice-template                                                     |

---

## References

- [Shared Constructs Plan](./01-shared-constructs-plan.md)
- [Template Conversion Progress](../ai/03-template-conversion-progress.md)
- [Template Comparison](../ai/04-template-comparison.md)
- [infra-contracts Guide](/home/nickt/projects/smw/infra-contracts/docs/infra-contracts-guide.md)
