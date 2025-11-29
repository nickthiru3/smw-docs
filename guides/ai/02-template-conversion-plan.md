# Template Conversion Plan

**Purpose:** Plan the conversion of svc-merchants into a reusable GitHub template for AI agentic backend services.

**Status:** Planning  
**Created:** November 2025  
**Last Updated:** November 2025

---

## Table of Contents

1. [Overview](#overview)
2. [Conversion Goals](#conversion-goals)
3. [Phase 1: Create Template Copy](#phase-1-create-template-copy)
4. [Phase 2: Identify Template Handles](#phase-2-identify-template-handles)
5. [Phase 3: Remove Merchant-Specific Code](#phase-3-remove-merchant-specific-code)
6. [Phase 4: Add AI-Specific Constructs](#phase-4-add-ai-specific-constructs)
7. [Phase 5: Update Documentation](#phase-5-update-documentation)
8. [Phase 6: Create Configuration Guide](#phase-6-create-configuration-guide)
9. [Template Handle Reference](#template-handle-reference)
10. [Progress Tracking](#progress-tracking)

---

## Overview

### Current State

The `svc-merchants` directory is a working microservice for the SMW (Sustainable Marketplace for Waste) application. It contains:

- Production-grade AWS CDK infrastructure
- Merchant-specific business logic
- SMW-specific configuration
- Documentation with merchant references
- Cognito authentication (User Pool, Identity Pool, Groups)

### Two-Template Strategy

We are creating **two separate templates**:

#### 1. Users Service Template (`svc-users-template`)

**Source:** `svc-merchants` (this conversion)

**Purpose:** User management and authentication service

**Includes:**

- Cognito User Pool, Identity Pool, User Groups
- User sign-up/sign-in flows
- IAM roles for authenticated users
- Lambda triggers (welcome email, verification)

**Usage:** One instance per application (shared auth across all microservices)

#### 2. Generic Microservice Template (to be reviewed)

**Source:** User's existing generic template project (to be added to workspace)

**Purpose:** Domain-specific microservices without auth

**Includes:**

- API Gateway + Lambda endpoints
- DynamoDB database
- Monitoring and alarms
- CI/CD pipeline

**Usage:** Multiple instances per application (one per domain/service)

### Target State for Users Template

A **GitHub template repository** (`svc-users-template`) that can be used to bootstrap user management services for AI agentic apps with:

- Generic, configurable infrastructure
- Placeholder handles for service-specific values
- Template-ready documentation
- AI-specific constructs (agents, orchestration, payments)
- Clear configuration guide for developers

### Approach

1. **Copy, don't modify** - Created `svc-users-template/` from `svc-merchants/`
2. **Identify handles** - Find all merchant/SMW-specific references
3. **Replace with placeholders** - Use consistent naming convention
4. **Add AI constructs** - Implement new capabilities
5. **Update docs** - Make all documentation generic
6. **Review generic template** - Merge improvements from svc-merchants

---

## Conversion Goals

### Must Have

- [ ] Generic service name configuration
- [ ] No merchant-specific business logic
- [ ] No SMW-specific references
- [ ] Template-ready documentation
- [ ] Clear configuration handles
- [ ] Working deployment to AWS

### Should Have

- [ ] AI agent execution construct
- [ ] Step Functions orchestration construct
- [ ] Stripe payment integration construct
- [ ] Vector database integration (optional)

### Nice to Have

- [ ] GitHub Actions workflow for template testing
- [ ] Cookiecutter or similar templating integration
- [ ] Interactive setup script

---

## Phase 1: Create Template Copy

### Task 1.1: Copy Directory ✅

**Status:** Completed

Created template copy at `/home/nickt/projects/smw/svc-users-template/`

```bash
cp -r svc-merchants svc-users-template
```

### Task 1.2: Update Git Configuration

**Status:** Pending user action

After user creates GitHub repository, run:

```bash
cd /home/nickt/projects/smw/svc-users-template
rm -rf .git
git init
git add .
git commit -m "Initial commit: Users service template from svc-merchants"
git remote add origin git@github.com:nickthiru/svc-users-template.git
git branch -M main
git push -u origin main
```

### Task 1.3: Enable GitHub Template

**Status:** Pending (after Task 1.2)

In GitHub repository settings:

1. Go to Settings → General
2. Check "Template repository"
3. This enables "Use this template" button

---

## Phase 2: Identify Template Handles

### Files to Audit

The following files contain service-specific values that need template handles:

#### Configuration Files

| File                  | Handle Type               | Priority |
| --------------------- | ------------------------- | -------- |
| `.env.example`        | Environment variables     | High     |
| `config/default.ts`   | Service configuration     | High     |
| `config/service.ts`   | Service name, description | High     |
| `config/database.ts`  | Table names               | High     |
| `config/api.ts`       | API name, description     | High     |
| `config/resources.ts` | Resource naming patterns  | High     |
| `config/github.ts`    | Repository name           | Medium   |
| `config/aws.ts`       | AWS settings              | Medium   |
| `package.json`        | Package name, description | High     |
| `cdk.json`            | App name                  | Medium   |

#### Infrastructure Files

| File                   | Handle Type       | Priority |
| ---------------------- | ----------------- | -------- |
| `bin/app.ts`           | Stack naming      | High     |
| `lib/service-stack.ts` | Stack description | Medium   |
| `lib/api/endpoints/*`  | Endpoint paths    | High     |
| `lib/db/faux-sql/*`    | Table definitions | High     |
| `lib/auth/*`           | User pool naming  | Medium   |

#### Source Files

| File                           | Handle Type      | Priority |
| ------------------------------ | ---------------- | -------- |
| `src/data-access/merchants.ts` | Business logic   | High     |
| `src/types/merchant.ts`        | Type definitions | High     |
| `src/helpers/*`                | Helper functions | Low      |

#### Documentation Files

| File                        | Handle Type               | Priority |
| --------------------------- | ------------------------- | -------- |
| `README.md`                 | Project description       | High     |
| `docs/README.md`            | Documentation overview    | High     |
| `docs/implementation/*.md`  | All implementation guides | Medium   |
| `docs/testing/*.md`         | All testing guides        | Medium   |
| `docs/DOCUMENTATION-MAP.md` | Documentation map         | Low      |

#### Test Files

| File                         | Handle Type       | Priority |
| ---------------------------- | ----------------- | -------- |
| `test/lib/**/*.test.ts`      | CDK tests         | Medium   |
| `test/data-access/*.test.ts` | Data access tests | High     |
| `test/e2e/**/*.test.ts`      | E2E tests         | High     |

---

## Phase 3: Remove Merchant-Specific Code

### Task 3.1: Replace Service Name

**Current:** `svc-merchants`, `merchants`

**Template handle:** `{{SERVICE_NAME}}`, `{{SERVICE_NAME_SINGULAR}}`

**Files to update:**

```
config/service.ts
├── name: "svc-merchants" → "svc-{{SERVICE_NAME}}"
├── description: "Merchants microservice" → "{{SERVICE_DESCRIPTION}}"

package.json
├── name: "@smw/svc-merchants" → "@{{ORG_NAME}}/svc-{{SERVICE_NAME}}"
├── description: "..." → "{{SERVICE_DESCRIPTION}}"

bin/app.ts
├── Stack names: "dev-svc-merchants-ServiceStack" → "dev-svc-{{SERVICE_NAME}}-ServiceStack"
```

### Task 3.2: Replace Database Tables

**Current:** `Merchants` table with merchant-specific attributes

**Template handle:** `{{ENTITY_NAME}}` table with generic attributes

**Files to update:**

```
config/database.ts
├── tableName: "Merchants" → "{{ENTITY_NAME}}"
├── attributes: merchant-specific → generic example

lib/db/faux-sql/construct.ts
├── Table definitions → generic example

src/data-access/merchants.ts
├── Rename to: src/data-access/{{entity}}.ts
├── Functions: getMerchant → get{{Entity}}
```

### Task 3.3: Replace API Endpoints

**Current:** `/users`, `/merchants/*`

**Template handle:** Generic CRUD endpoints

**Files to update:**

```
lib/api/endpoints/
├── merchants/ → {{entity}}/
├── users/ → users/ (keep as common pattern)

config/api.ts
├── Endpoint configurations
```

### Task 3.4: Replace Business Logic

**Current:** Merchant sign-up, merchant data access

**Template handle:** Generic entity CRUD

**Files to update:**

```
src/data-access/merchants.ts → src/data-access/example-entity.ts
src/types/merchant.ts → src/types/example-entity.ts
lib/api/endpoints/merchants/ → lib/api/endpoints/example/
```

### Task 3.5: Replace Test Files

**Current:** Merchant-specific tests

**Template handle:** Generic entity tests

**Files to update:**

```
test/data-access/merchants.test.ts → test/data-access/example-entity.test.ts
test/e2e/users/*.test.ts → Keep as example
```

---

## Phase 4: Add AI-Specific Constructs

### Task 4.1: Agent Execution Construct

Create `lib/agents/construct.ts`:

```typescript
/**
 * Agent Execution Infrastructure
 *
 * Provides Lambda functions for AI agent execution with:
 * - 15-minute timeout for long-running agents
 * - SQS queue for async invocation
 * - DLQ for failed executions
 * - CloudWatch metrics for agent performance
 */
class AgentsConstruct extends Construct {
  // Implementation
}
```

**Sub-tasks:**

- [ ] Create construct file
- [ ] Add to ServiceStack
- [ ] Create configuration in config/agents.ts
- [ ] Add feature flag
- [ ] Write tests
- [ ] Write documentation

### Task 4.2: Orchestration Construct (Step Functions)

Create `lib/orchestration/construct.ts`:

```typescript
/**
 * Agent Orchestration Infrastructure
 *
 * Provides Step Functions state machine for:
 * - Multi-agent workflows
 * - Parallel execution
 * - Error handling and retries
 * - Human-in-the-loop approval
 */
class OrchestrationConstruct extends Construct {
  // Implementation
}
```

**Sub-tasks:**

- [ ] Create construct file
- [ ] Add to ServiceStack
- [ ] Create configuration in config/orchestration.ts
- [ ] Add feature flag
- [ ] Write tests
- [ ] Write documentation

### Task 4.3: Payments Construct (Stripe)

Create `lib/payments/construct.ts`:

```typescript
/**
 * Payment Processing Infrastructure
 *
 * Provides Stripe integration for:
 * - Webhook endpoint
 * - Subscription management
 * - Usage-based billing
 * - Customer portal
 */
class PaymentsConstruct extends Construct {
  // Implementation
}
```

**Sub-tasks:**

- [ ] Create construct file
- [ ] Add to ServiceStack
- [ ] Create configuration in config/payments.ts
- [ ] Add feature flag
- [ ] Store Stripe secret in Secrets Manager
- [ ] Write tests
- [ ] Write documentation

### Task 4.4: Vector Database Construct (Optional)

Create `lib/vector-db/construct.ts`:

```typescript
/**
 * Vector Database Infrastructure
 *
 * Provides semantic search capabilities via:
 * - Option A: Pinecone integration
 * - Option B: OpenSearch Serverless
 */
class VectorDbConstruct extends Construct {
  // Implementation
}
```

**Sub-tasks:**

- [ ] Create construct file
- [ ] Add to ServiceStack
- [ ] Create configuration in config/vector-db.ts
- [ ] Add feature flag
- [ ] Write tests
- [ ] Write documentation

---

## Phase 5: Update Documentation

### Task 5.1: Update README.md

**Current:** SMW/merchants-specific

**Target:** Generic template README with:

- Template description
- Quick start guide
- Configuration instructions
- Link to detailed guides

### Task 5.2: Update Implementation Guides

**Files to update:**

| Guide                                        | Changes Needed                         |
| -------------------------------------------- | -------------------------------------- |
| `adding-endpoints-part-1-lambda-handlers.md` | Replace merchant examples with generic |
| `adding-endpoints-part-2-api-gateway.md`     | Replace merchant examples with generic |
| `authentication.md`                          | Keep mostly as-is                      |
| `data-access.md`                             | Replace merchant examples with generic |
| `database-setup.md`                          | Replace merchant examples with generic |
| `deployment.md`                              | Update stack names                     |
| `environment-variables.md`                   | Update variable names                  |
| `configuration-management/*.md`              | Update examples                        |

### Task 5.3: Update Testing Guides

**Files to update:**

| Guide                           | Changes Needed             |
| ------------------------------- | -------------------------- |
| `handler-testing-guide.md`      | Replace merchant examples  |
| `cdk-template-testing-guide.md` | Update stack names         |
| `e2e-testing-guide.md`          | Replace merchant endpoints |

### Task 5.4: Add AI-Specific Guides

**New guides to create:**

| Guide                | Purpose                    |
| -------------------- | -------------------------- |
| `agent-execution.md` | How to implement AI agents |
| `orchestration.md`   | How to use Step Functions  |
| `payments.md`        | How to integrate Stripe    |
| `vector-search.md`   | How to add semantic search |

### Task 5.5: Update DOCUMENTATION-MAP.md

Add new guides to the documentation map and update all references.

---

## Phase 6: Create Configuration Guide

### Task 6.1: Create Template Configuration Guide

Create `docs/TEMPLATE-CONFIGURATION.md`:

```markdown
# Template Configuration Guide

## Quick Start

1. Clone this template
2. Run setup script: `./scripts/setup.sh`
3. Configure your service in `config/`
4. Deploy: `npm run deploy`

## Configuration Handles

### Required Configuration

| Handle       | Location          | Description       |
| ------------ | ----------------- | ----------------- |
| SERVICE_NAME | config/service.ts | Your service name |
| ...          | ...               | ...               |

### Optional Configuration

| Handle | Location | Description |
| ------ | -------- | ----------- |
| ...    | ...      | ...         |
```

### Task 6.2: Create Setup Script

Create `scripts/setup.sh`:

```bash
#!/bin/bash
# Interactive setup script that prompts for configuration values
# and updates all template handles
```

---

## Template Handle Reference

### Naming Convention

All template handles use the format: `{{HANDLE_NAME}}`

### Complete Handle List

| Handle                    | Description               | Example Value                       | Files                                       |
| ------------------------- | ------------------------- | ----------------------------------- | ------------------------------------------- |
| `{{SERVICE_NAME}}`        | Service name (kebab-case) | `linkedin-agent`                    | config/service.ts, package.json, bin/app.ts |
| `{{SERVICE_NAME_PASCAL}}` | Service name (PascalCase) | `LinkedInAgent`                     | lib/\*.ts                                   |
| `{{SERVICE_DESCRIPTION}}` | Service description       | `LinkedIn content generation agent` | config/service.ts, package.json             |
| `{{ORG_NAME}}`            | Organization/scope name   | `nickthiru`                         | package.json                                |
| `{{ENTITY_NAME}}`         | Primary entity name       | `Post`                              | config/database.ts, src/data-access/\*.ts   |
| `{{ENTITY_NAME_PLURAL}}`  | Primary entity plural     | `Posts`                             | lib/api/endpoints/\*.ts                     |
| `{{AWS_ACCOUNT_ID}}`      | AWS account ID            | `123456789012`                      | .env                                        |
| `{{AWS_REGION}}`          | AWS region                | `us-east-1`                         | .env                                        |
| `{{GITHUB_REPO}}`         | GitHub repository         | `nickthiru/linkedin-agent`          | config/github.ts                            |

---

## Progress Tracking

### Phase 1: Create Template Copy

| Task                  | Status         | Notes |
| --------------------- | -------------- | ----- |
| 1.1 Copy directory    | ⬜ Not started |       |
| 1.2 Update git config | ⬜ Not started |       |
| 1.3 Initial commit    | ⬜ Not started |       |

### Phase 2: Identify Template Handles

| Task                       | Status         | Notes |
| -------------------------- | -------------- | ----- |
| Audit config files         | ⬜ Not started |       |
| Audit infrastructure files | ⬜ Not started |       |
| Audit source files         | ⬜ Not started |       |
| Audit documentation        | ⬜ Not started |       |
| Audit test files           | ⬜ Not started |       |

### Phase 3: Remove Merchant-Specific Code

| Task                        | Status         | Notes |
| --------------------------- | -------------- | ----- |
| 3.1 Replace service name    | ⬜ Not started |       |
| 3.2 Replace database tables | ⬜ Not started |       |
| 3.3 Replace API endpoints   | ⬜ Not started |       |
| 3.4 Replace business logic  | ⬜ Not started |       |
| 3.5 Replace test files      | ⬜ Not started |       |

### Phase 4: Add AI-Specific Constructs

| Task                          | Status         | Notes |
| ----------------------------- | -------------- | ----- |
| 4.1 Agent execution construct | ⬜ Not started |       |
| 4.2 Orchestration construct   | ⬜ Not started |       |
| 4.3 Payments construct        | ⬜ Not started |       |
| 4.4 Vector database construct | ⬜ Not started |       |

### Phase 5: Update Documentation

| Task                             | Status         | Notes |
| -------------------------------- | -------------- | ----- |
| 5.1 Update README.md             | ⬜ Not started |       |
| 5.2 Update implementation guides | ⬜ Not started |       |
| 5.3 Update testing guides        | ⬜ Not started |       |
| 5.4 Add AI-specific guides       | ⬜ Not started |       |
| 5.5 Update DOCUMENTATION-MAP.md  | ⬜ Not started |       |

### Phase 6: Create Configuration Guide

| Task                           | Status         | Notes |
| ------------------------------ | -------------- | ----- |
| 6.1 Create configuration guide | ⬜ Not started |       |
| 6.2 Create setup script        | ⬜ Not started |       |

---

## Related Documents

- [Backend Infrastructure Adoption](./01-backend-infrastructure-adoption.md)
- [svc-merchants Documentation](../../svc-merchants/docs/)
- [AI Agentic Products Strategy](../../../business/docs/ai-agentic-products-strategy/)

---

## Notes

### Open Questions

1. **Template location:** Should the template be a separate repository or within the smw workspace?
2. **Templating tool:** Should we use a tool like Cookiecutter for variable substitution?
3. **Setup automation:** How much should be automated vs. manual configuration?

### Decisions Made

| Decision                  | Date     | Rationale                                     |
| ------------------------- | -------- | --------------------------------------------- |
| Use svc-merchants as base | Nov 2025 | Production-grade patterns, comprehensive docs |
| Copy instead of modify    | Nov 2025 | Preserve working svc-merchants                |

### Lessons Learned

(To be updated as we progress)
