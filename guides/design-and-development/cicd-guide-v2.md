# CI/CD Guide v2: Simplified Trunk-Based Strategy for Microservices

## Table of Contents

- [CI/CD Guide v2: Simplified Trunk-Based Strategy for Microservices](#cicd-guide-v2-simplified-trunk-based-strategy-for-microservices)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Strategy Overview](#strategy-overview)
    - [Why Refined Trunk-Based with Native AWS Integration?](#why-refined-trunk-based-with-native-aws-integration)
    - [Architecture Comparison](#architecture-comparison)
  - [Core Concepts](#core-concepts)
    - [Branch Strategy](#branch-strategy)
    - [Environment Strategy](#environment-strategy)
    - [Versioning Strategy](#versioning-strategy)
    - [Team Workflow](#team-workflow)
  - [LocalStack Integration for Local Development](#localstack-integration-for-local-development)
    - [Overview](#overview)
    - [Quick Setup](#quick-setup)
    - [Configuration Management](#configuration-management)
      - [SSM Parameter Path Standardization](#ssm-parameter-path-standardization)
  - [Simplified Trunk-Based Release Workflow](#simplified-trunk-based-release-workflow)
    - [Complete Enterprise Architecture](#complete-enterprise-architecture)
      - [Environment Strategy](#environment-strategy-1)
      - [Complete Workflow Summary](#complete-workflow-summary)
    - [Team Roles and Responsibilities](#team-roles-and-responsibilities)
      - [Development Team](#development-team)
      - [Release Manager (Optional)](#release-manager-optional)
    - [Automation Scripts](#automation-scripts)
      - [Package.json Scripts](#packagejson-scripts)
      - [Release Script](#release-script)
    - [Manual Approval Integration](#manual-approval-integration)
      - [AWS CodePipeline Manual Approval (Recommended)](#aws-codepipeline-manual-approval-recommended)
    - [Rollback Strategies](#rollback-strategies)
      - [Production Rollback Options](#production-rollback-options)
    - [Merge Request Strategy](#merge-request-strategy)
      - [Code Review Process](#code-review-process)
    - [Hotfix Process](#hotfix-process)
      - [Emergency Hotfix Workflow](#emergency-hotfix-workflow)
  - [Teams Coordination \& Workflow](#teams-coordination--workflow)
    - [Platform Team Responsibilities](#platform-team-responsibilities)
    - [Development Team Integration](#development-team-integration)
  - [Multi-Repo CI/CD Architecture](#multi-repo-cicd-architecture)
    - [Overview](#overview-1)
    - [Repository Structure](#repository-structure)
    - [Pipeline Per Service Strategy](#pipeline-per-service-strategy)
    - [Resource Naming \& Isolation](#resource-naming--isolation)
    - [Rollback Strategies](#rollback-strategies-1)
      - [Service-Level Rollbacks](#service-level-rollbacks)
      - [App-Level Rollbacks](#app-level-rollbacks)
      - [Emergency Rollback Scenarios](#emergency-rollback-scenarios)
    - [Pipeline Bootstrap \& LocalStack Separation](#pipeline-bootstrap--localstack-separation)
    - [Team Responsibilities](#team-responsibilities)
    - [CodePipeline Rollback Mechanics](#codepipeline-rollback-mechanics)
    - [Coordination Workflows](#coordination-workflows)
  - [Reference Implementation](#reference-implementation)
    - [Pipeline Configuration](#pipeline-configuration)
    - [Stage Configuration](#stage-configuration)
  - [Migration from v1 Implementation](#migration-from-v1-implementation)

## Introduction

This guide outlines a **refined trunk-based CI/CD strategy** specifically designed for **enterprise microservices development**. After extensive research and practical experience with various branching strategies (Three-Flow, GitLab Flow, Release Flow), we've developed a pragmatic approach that leverages **native AWS CodePipeline triggers** and eliminates unnecessary complexity.

**Key Design Principles:**

- **Native AWS integration**: CodePipeline triggers directly on branch changes
- **Simplified branching**: Master branch + release branch for staging/production
- **Cost optimization**: LocalStack for development, minimal AWS accounts
- **Quality gates**: Manual approval for production deployments
- **Enterprise-ready**: Scalable for teams while maintaining simplicity
- **Fast feedback**: Direct path from development to production

## Strategy Overview

### Why Refined Trunk-Based with Native AWS Integration?

**Traditional Complex Strategies Issues:**

- ❌ Too many branches (master, candidate, release, feature)
- ❌ GitHub Actions orchestration complexity
- ❌ Manual pipeline triggering and coordination overhead
- ❌ Slow feedback loops and merge conflicts

**Our Refined Approach Benefits:**

- ✅ **Native CodePipeline triggers**: Pipelines trigger automatically on branch changes
- ✅ **Two-branch strategy**: Master (development) + Release (staging/production)
- ✅ **LocalStack development**: $0 cost local testing with full AWS emulation
- ✅ **Single pipeline per environment**: Staging pipeline with production stage
- ✅ **Manual production gates**: Controlled production deployments
- ✅ **Simplified hotfix flow**: Master → LocalStack → Version Cut → Staging → Production

### Architecture Comparison

| Aspect                | Complex Strategies (Three-Flow)         | Our Refined Approach                       |
| --------------------- | --------------------------------------- | ------------------------------------------ |
| **Branches**          | 3-4 branches (master/candidate/release) | 2 branches (master + release)              |
| **Pipeline Triggers** | GitHub Actions orchestration            | Native CodePipeline branch triggers        |
| **Environments**      | 4 environments (local/dev/staging/prod) | 3 environments (LocalStack/staging/prod)   |
| **AWS Accounts**      | 3-4 accounts                            | 2 accounts (staging + production)          |
| **Deployment Flow**   | Complex merge-back automation           | Simple version cut-and-deploy workflow     |
| **Manual Approvals**  | Multiple approval points                | Single production approval gate            |
| **Hotfix Process**    | Separate release branch workflow        | Master → LocalStack → Version Cut → Deploy |
| **Cost**              | High (multiple AWS accounts)            | Low (LocalStack + 2 accounts)              |
| **Complexity**        | High coordination overhead              | Low, enterprise-ready simplicity           |

## Core Concepts

### Branch Strategy

```
┌─────────────────┐    ┌──────────────────┐     ┌─────────────────┐    ┌─────────────────┐
│  Master Branch  │ →  │ "Version Cut" to │  →  │  Release Branch │ →  │   Production    │
│   (LocalStack)  │    │  Release Branch  │     │   (Staging)     │    │ (Manual Gate)   │
└─────────────────┘    └──────────────────┘     └─────────────────┘    └─────────────────┘
                                                        │
                                                        ▼
                                                ┌─────────────────┐
                                                │ CodePipeline    │
                                                │ Auto-Triggered  │
                                                └─────────────────┘
```

### Environment Strategy

| Environment    | Source         | Trigger                                                         | Purpose               | Account          |
| -------------- | -------------- | --------------------------------------------------------------- | --------------------- | ---------------- |
| **Local**      | Master branch  | Manual (Git "version cut" from master branch to release branch) | Development & testing | LocalStack       |
| **Staging**    | Release branch | Auto (CodePipeline)                                             | QA & integration test | AWS - Staging    |
| **Production** | Release branch | Manual approval gate (CodePipeline)                             | Live deployment       | AWS - Production |

### Versioning Strategy

**Semantic Versioning (SemVer):**

- **Format**: `v1.2.3` (major.minor.patch)
- **Major**: Breaking changes (v1.0.0 → v2.0.0)
- **Minor**: New features (v1.0.0 → v1.1.0)
- **Patch**: Bug fixes (v1.0.0 → v1.0.1)

### Team Workflow

**Daily Development:**

1. Work directly on `master` branch or create (local only) feature branches from `master`
2. Develop and test locally with LocalStack (`npm run dev:localstack`)
3. Test infrastructure changes with LocalStack deployment
4. Create merge request when ready (if using feature branches, merge them to local master)

**Release Process (Version Cut Workflow):**

1. **Prepare Release**: Ensure master branch is stable and tested in LocalStack
2. **Version Cut to Release Branch**:
   ```bash
   git checkout -b release  # or switch to existing release branch
   git merge master --no-ff
   git tag -a v1.2.3 -m "Release v1.2.3"  # increment version appropriately
   git push origin release --tags
   ```
3. **Automatic Staging Deployment**: CodePipeline automatically triggers on release branch push
4. **QA Testing**: Team tests the staging deployment
5. **Production Deployment**: Manual approval in CodePipeline console to deploy to production

**Hotfix Process:**

1. **Fix on Master**: Make hotfix directly on master branch
2. **Test Locally**: Verify fix using LocalStack
3. **Version Cut to Release**: Follow same "version cut" process with patch version increment
4. **Deploy**: Staging → Manual approval → Production

## LocalStack Integration for Local Development

### Overview

LocalStack provides a fully functional local AWS cloud stack, enabling developers to develop and test their cloud applications offline.

**Installation Methods:**

Developers can choose from multiple LocalStack installation approaches:

- **Docker Desktop Extension** (Recommended for GUI users)
- **CLI Installation** (Traditional command-line approach)
- **Docker Compose** (Team standardization)
- **Manual Docker Run** (Advanced users)

**Note:** The setup steps below focus on CLI/Docker Compose approach. Developers using other installation methods (like Docker Desktop extension) should adapt the configuration and workflow guidance to their chosen setup. The core concepts, environment variables, and CDK integration patterns remain the same regardless of installation method.

**Key Benefits:**

- **Cost Optimization**: $0 development costs
- **Speed**: No network latency, instant deployments
- **Isolation**: Each developer has complete AWS environment
- **Offline Development**: Work without internet connectivity

### Quick Setup

```bash
# One-time setup
npm run localstack:setup

# Daily workflow
npm run localstack:start
npm run deploy:localstack
npm run dev:localstack
npm run localstack:stop
```

### Configuration Management

#### SSM Parameter Path Standardization

To avoid hardcoding SSM paths, all services resolve their public/private binding base path using this precedence:

1. `APP_BASE_PATH` (env) — preferred
2. `parameterStorePrefix` (service config; legacy/back-compat)
3. Fallback: `/super-deals`

Construct paths as:

```
{APP_BASE_PATH}/{ENV_NAME}/{SERVICE_NAME}/public
{APP_BASE_PATH}/{ENV_NAME}/{SERVICE_NAME}/private
```

Required variables:

- `ENV_NAME` (e.g., `local`, `staging`, `prod`)
- `SERVICE_NAME` (e.g., `deals-ms`, `users-ms`)

Optional variables:

- `APP_BASE_PATH` (e.g., `/super-deals`) — overrides config when set
- `PARAMETER_STORE_PREFIX` — maps to `parameterStorePrefix` when `APP_BASE_PATH` is not set

CDK injects `SSM_PUBLIC_PATH` into discovery Lambdas and scopes IAM to `arn:aws:ssm:*:*:parameter{SSM_PUBLIC_PATH}*`.

#### Secure SSM Dynamic References for Secrets

For secrets like Slack webhooks, we use CloudFormation dynamic references to SSM SecureString parameters to avoid plaintext values in templates.

- Standardized binding name: `slackWebhookUrl`
- Standardized SSM path: `/super-deals/{ENV_NAME}/platform/private/monitor/slack/webhookUrl`
- CDK bindings opt-in to secure reads by passing `secure: true` to the shared bindings utility. When set, the helper resolves the latest SecureString version using `SecretValue.ssmSecure()` and emits a dynamic reference such as `{{resolve:ssm-secure:/super-deals/dev/platform/private/monitor/slack/webhookUrl}}`.

```ts
// lib/bindings/monitor/construct.ts
const bindings = new BindingsUtilConstruct<IMonitorBindings>(this, "MonitorBindings", {
  envName,
  producerServiceName: "platform",
  visibility: "private",
  secure: true,
  spec: { slackWebhookUrl: "monitor/slack/webhookUrl" },
});

// In monitor Lambda construct: sets env
SLACK_WEBHOOK_URL: bindings.values.slackWebhookUrl;
```

If you ever need to pin to a specific SecureString version, call `SecretValue.ssmSecure(parameterName, version)` directly in a bespoke construct.

**LocalStack Configuration:**

```typescript
export const config: Config = {
  environment: "localstack",
  aws: {
    region: "us-east-1",
    endpoints: {
      apiGateway: "http://localhost:4566",
      dynamodb: "http://localhost:4566",
      lambda: "http://localhost:4566",
      s3: "http://localhost:4566",
    },
  },
  api: {
    stageName: "local",
  },
  database: {
    tablePrefix: "local-",
  },
};
```

## Simplified Trunk-Based Release Workflow

### Complete Enterprise Architecture

#### Environment Strategy

| Environment    | Branch           | AWS Account | Purpose                | Trigger      |
| -------------- | ---------------- | ----------- | ---------------------- | ------------ |
| **Local**      | Feature branches | LocalStack  | Individual development | Manual       |
| **Staging**    | `main`           | Current AWS | Integration testing    | Push to main |
| **Production** | `main`           | Future AWS  | Live deployment        | Version tag  |

#### Complete Workflow Summary

```
1. Local Development: Feature branch + LocalStack testing
2. Integration: Merge request → main branch
3. Staging: Auto-deploy main → AWS Staging
4. QA Testing: Manual testing in staging environment
5. Production: Create version tag → AWS Production deployment
6. Hotfixes: Direct commit to main → staging → tag → production
```

### Team Roles and Responsibilities

#### Development Team

- **Daily Development**: Work on feature branches with LocalStack
- **Code Review**: Review merge requests from team members
- **Quality Assurance**: Test features in staging environment
- **Bug Fixes**: Address issues found during testing

#### Release Manager (Optional)

- **Release Planning**: Coordinate release schedules
- **Quality Gates**: Final approval for production deployments
- **Tag Creation**: Create version tags for production releases
- **Communication**: Update stakeholders on release status

### Automation Scripts

#### Package.json Scripts

```json
{
  "scripts": {
    "release": "./scripts/create-release.sh",
    "hotfix": "./scripts/create-hotfix.sh",
    "localstack:setup": "./scripts/localstack-setup.sh",
    "localstack:start": "docker-compose -f docker-compose.localstack.yml up -d",
    "localstack:stop": "docker-compose -f docker-compose.localstack.yml down",
    "deploy:localstack": "NODE_ENV=localstack npm run deploy",
    "deploy:staging": "NODE_ENV=staging npm run deploy",
    "deploy:production": "NODE_ENV=production npm run deploy"
  }
}
```

#### Release Script

```bash
#!/bin/bash
# scripts/create-release.sh
set -e

read -p "Enter release version (e.g., 1.2.3): " VERSION

if [[ ! $VERSION =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "❌ Invalid version format. Use X.Y.Z"
  exit 1
fi

echo "🏷️ Creating release v$VERSION..."

git checkout main
git pull origin main
git tag v$VERSION
git push origin v$VERSION

echo "✅ Release v$VERSION created and pushed"
echo "🚀 Production deployment will begin automatically"
```

### Manual Approval Integration

#### AWS CodePipeline Manual Approval (Recommended)

**Setup in Pipeline Stack:**

```typescript
const stageOptions: pipelines.AddStageOpts = {
  pre:
    stageName === "production"
      ? [new pipelines.ManualApprovalStep("PromoteToProduction")]
      : undefined,
};
```

**Approval Process:**

1. AWS sends email/SNS notification to approvers
2. Release manager reviews deployment in AWS Console
3. Click "Approve" in CodePipeline console
4. Pipeline continues to production deployment

### Rollback Strategies

#### Production Rollback Options

**1. AWS CodePipeline Console (Recommended):**

- Navigate to pipeline execution history
- Select previous successful deployment
- Click "Retry" to redeploy previous version

**2. Git-Based Rollback:**

```bash
# Create rollback tag
git tag v1.2.4-rollback v1.2.2  # Rollback to v1.2.2
git push origin v1.2.4-rollback
```

### Merge Request Strategy

#### Code Review Process

**Branch Protection Rules:**

- Require pull request reviews before merging
- Require status checks to pass before merging
- Require branches to be up to date before merging
- Restrict pushes to main branch

**Review Checklist:**

- ✅ Code follows team standards
- ✅ Tests are included and passing
- ✅ Documentation is updated
- ✅ No security vulnerabilities introduced

### Hotfix Process

#### Emergency Hotfix Workflow

```bash
# 1. Create hotfix branch
git checkout main
git pull origin main
git checkout -b hotfix/critical-fix

# 2. Make fix and test locally
# ... make changes ...
npm run test
npm run deploy:localstack

# 3. Create merge request
# ... create PR for code review ...

# 4. After merge to main
git checkout main
git pull origin main

# 5. Create hotfix tag
git tag v1.2.4
git push origin v1.2.4
```

## Teams Coordination & Workflow

### Platform Team Responsibilities

**IAM Management:**

- Create shared IAM user for GitHub Actions
- Manage access key rotation (quarterly)
- Update permissions as services are added

**Infrastructure:**

- Maintain CodeStar connections
- Manage pipeline configurations
- Monitor deployment health

### Development Team Integration

**Repository Setup:**

- Add GitHub repository secrets
- Configure branch protection rules
- Set up merge request templates

**Daily Workflow:**

- Use LocalStack for local development
- Create merge requests for all changes
- Test in staging before production release

## Multi-Repo CI/CD Architecture

### Overview

For the Super Deals microservices architecture, each BFF API and microservice team maintains their own repository and CI/CD pipeline. This approach maximizes team autonomy while maintaining system coherence through shared patterns and coordination workflows.

### Repository Structure

```
super-deals-bff-api-web/     # BFF API team repo
├── .github/workflows/
├── lib/pipeline/
└── src/

super-deals-ms-deals/        # Deals microservice team repo
├── .github/workflows/
├── lib/pipeline/
└── src/

super-deals-ms-users/        # Users microservice team repo
├── .github/workflows/
├── lib/pipeline/
└── src/
```

### Pipeline Per Service Strategy

**Architecture Pattern:**

```
Repo A (BFF-API)   → Pipeline A → Staging Account (BFF resources)
Repo B (Deals-MS)  → Pipeline B → Staging Account (Deals resources)
Repo C (Users-MS)  → Pipeline C → Staging Account (Users resources)
```

**Benefits:**

- ✅ **Team Independence**: Each team controls their deployment lifecycle
- ✅ **Parallel Development**: Teams can deploy simultaneously without conflicts
- ✅ **Isolated Failures**: Service issues don't block other teams
- ✅ **Service-Level Versioning**: Independent semantic versioning per service

### Resource Naming & Isolation

Since multiple services deploy to the same staging AWS account, all CDK construct IDs and AWS resource names must be prefixed with the service identifier to prevent conflicts.

**Naming Convention:**

```typescript
// In super-deals-ms-deals CDK stack
const dealsTable = new dynamodb.Table(this, "DealsTable", {
  tableName: "deals-ms-deals-table", // Prefixed with service name
  // ...
});

const dealsApi = new apigateway.RestApi(this, "DealsApi", {
  restApiName: "deals-ms-api", // Prefixed with service name
  // ...
});
```

**Service Versioning Pattern:**

```
Service Versions: bff-api@v1.2.3, deals-ms@v2.1.0, users-ms@v1.5.2
App Version: super-deals@v3.1.0 (composed of above service versions)
```

### Rollback Strategies

#### Service-Level Rollbacks

**When Needed:**

- Individual service deployment introduces bugs
- Service performance degradation
- Service-specific security issues
- Database migration failures

**How to Execute:**

1. **Via CodePipeline Console:**

   - Navigate to the specific service's pipeline (e.g., `deals-ms-staging-pipeline`)
   - View execution history
   - Select previous successful deployment
   - Click "Retry" to redeploy previous version

2. **Via Git Tag Rollback:**
   ```bash
   # Example: Rolling back deals-ms from v2.1.0 to v2.0.5
   git checkout release
   git reset --hard v2.0.5
   git tag -a v2.1.1-rollback -m "Rollback to v2.0.5"
   git push origin release --force --tags
   ```

**Scenario Example:**

```
Problem: deals-ms@v2.1.0 has a bug causing 500 errors on deal creation
Impact: Only deal creation affected, other services unimpacted
Solution: Service-level rollback of deals-ms to v2.0.5
Result: Deal creation restored, other services continue normal operation
```

#### App-Level Rollbacks

**When Needed:**

- Breaking changes in service contracts/APIs
- Cross-service integration failures
- Coordinated feature rollbacks
- Security vulnerabilities affecting multiple services

**How to Execute:**

1. **Coordinated Service Rollbacks:**

   ```bash
   # Roll back to known-good service combination
   # BFF API team:
   git tag -a v1.2.4-rollback -m "App rollback to stable state"

   # Deals MS team:
   git tag -a v2.0.6-rollback -m "App rollback to stable state"

   # Users MS team:
   git tag -a v1.5.3-rollback -m "App rollback to stable state"
   ```

2. **Via Release Coordination:**
   - Platform team coordinates rollback across all affected services
   - Each team executes service-level rollback to specified versions
   - Integration testing validates the rolled-back app state

**Scenario Example:**

```
Problem: BFF API v1.2.3 introduces breaking change in user authentication flow
Impact: All services that depend on user context affected
Solution: App-level rollback
- BFF API: v1.2.3 → v1.2.2 (remove breaking auth change)
- Users MS: v1.5.2 → v1.5.1 (revert auth integration)
- Deals MS: No rollback needed (not affected)
Result: Authentication restored across entire application
```

#### Emergency Rollback Scenarios

**Scenario 1: Database Schema Change Gone Wrong**

```
Service: deals-ms@v2.1.0
Problem: DynamoDB table schema migration corrupts data
Impact: Deal creation/retrieval completely broken
Action: Service-level rollback + data restoration
Steps:
1. Rollback deals-ms to v2.0.5 (pre-migration)
2. Restore DynamoDB table from backup
3. Coordinate with QA team for validation
```

**Scenario 2: Cross-Service API Breaking Change**

```
Service: bff-api@v1.3.0 changes deal request format
Problem: deals-ms can't process new request format
Impact: Deal creation fails across entire app
Action: App-level coordinated rollback
Steps:
1. BFF API: v1.3.0 → v1.2.9 (revert API change)
2. Deals MS: May need rollback if it was updated for new format
3. Full integration testing before declaring rollback complete
```

**Scenario 3: Performance Degradation**

```
Service: users-ms@v1.6.0
Problem: New caching layer causes memory leaks
Impact: Gradual performance degradation, eventual service failure
Action: Service-level rollback
Steps:
1. Monitor CloudWatch metrics to confirm issue
2. Rollback users-ms to v1.5.8 via CodePipeline console
3. Investigate caching implementation offline
```

### Pipeline Bootstrap & LocalStack Separation

**Initial Team Setup:**

Each team must bootstrap their CodePipeline once at project start:

```bash
# One-time pipeline deployment (team lead)
ENV_NAME=staging npm run deploy  # Deploys pipeline to staging AWS account
```

**Daily Development Workflow:**

```bash
# LocalStack development (all team members)
ENV_NAME=local npm run deploy:localstack  # Service infra only, no pipeline
```

**CDK App Structure:**

```typescript
// bin/service-app.ts - Conditional stack deployment
const envName = process.env.ENV_NAME || "local";

if (envName === "local") {
  // LocalStack: Deploy only service infrastructure
  new ServiceInfraStack(app, "ServiceInfraStack", {
    envName,
    env: { account: "000000000000", region: "us-east-1" }, // LocalStack
  });
} else {
  // AWS: Deploy both pipeline AND service infrastructure
  new ServicePipelineStack(app, "ServicePipelineStack", {
    envName,
    env: { account: config.account, region: config.region },
  });
}
```

### Team Responsibilities

**Service-Level Responsibilities:**

| Action                       | Responsible Party | Authority Level   |
| ---------------------------- | ----------------- | ----------------- |
| Manual production approval   | Team lead         | Service-scoped    |
| Service rollback decision    | Team lead         | Service-scoped    |
| Service health monitoring    | Service team      | Service-scoped    |
| Breaking change coordination | Service team      | Cross-team impact |

**App-Level Responsibilities:**

| Action                       | Responsible Party | Authority Level    |
| ---------------------------- | ----------------- | ------------------ |
| App-level rollback           | Platform team     | Cross-service      |
| Emergency coordination       | On-call engineer  | Immediate response |
| Release planning             | Release manager   | Cross-team         |
| Integration issue resolution | Platform team     | Cross-service      |

### CodePipeline Rollback Mechanics

**How Rollbacks Actually Work:**

When you execute a rollback via the CodePipeline console:

1. **Artifact Reuse**: Pipeline uses stored artifacts from the selected previous execution
2. **No Rebuild**: Does NOT pull fresh code or rebuild from source
3. **Exact Restoration**: Deploys the identical CloudFormation template and Lambda code
4. **Fast Execution**: Skips source/build stages, only runs deployment

**Step-by-Step Rollback Process:**

```
1. AWS Console → CodePipeline → Select service pipeline
2. Click "Execution history" tab
3. Identify successful execution from desired time/version
4. Click "Retry" on that specific execution
5. Pipeline re-executes deployment stage only
6. Infrastructure restored to exact previous state
7. Verify rollback success via monitoring/testing
```

**Artifact Management:**

- **Storage**: S3 bucket managed by CodePipeline
- **Retention**: 30 days default (configurable)
- **Contents**: Compiled code, CloudFormation templates, config files
- **Cost**: Minimal S3 storage fees

### Coordination Workflows

**Release Coordination:**

- Teams announce major releases in shared Slack channel
- Breaking changes require coordination with dependent services
- Platform team maintains app-level version tracking

**Rollback Coordination:**

- Service-level rollbacks: Team lead decides independently
- App-level rollbacks: Platform team coordinates across teams
- Emergency rollbacks: On-call engineer can initiate, coordinate later

## Reference Implementation

### Pipeline Configuration

**AWS CDK Pipeline Stack:**

```typescript
const pipeline = new pipelines.CodePipeline(this, "Pipeline", {
  pipelineName: `super-deals-deals-ms-${envName}-pipeline`,
  synth: new pipelines.CodeBuildStep("Synth", {
    input: pipelines.CodePipelineSource.connection(gitHubRepo, gitHubBranch, {
      connectionArn: connectionArn,
      triggerOnPush: envName === "staging", // Only staging auto-triggers
    }),
    installCommands: ["npm ci"],
    commands: ["npm run build", "npx cdk synth"],
  }),
});
```

### Stage Configuration

**Environment-Specific Stages:**

```typescript
if (envName === "production") {
  pipeline.addStage(stage, {
    pre: [new pipelines.ManualApprovalStep("PromoteToProduction")],
  });
} else {
  pipeline.addStage(stage);
}
```

## Migration from v1 Implementation

**Migration Steps:**

1. **Update GitHub Actions**: Replace complex branching with simple main branch workflow
2. **Simplify Tagging**: Use only `v*` tags for production releases
3. **Remove Complex Branches**: Delete candidate and release branches
4. **Update Documentation**: Train team on new simplified workflow
5. **Test Workflow**: Validate with test releases before production use

**Benefits After Migration:**

- ✅ Faster development cycles
- ✅ Reduced merge conflicts
- ✅ Simplified team coordination
- ✅ Lower operational overhead
- ✅ Clearer deployment process

---

This simplified trunk-based strategy provides enterprise-grade CI/CD capabilities while maintaining the simplicity needed for small microservice teams. The approach balances automation with control, ensuring fast delivery without sacrificing quality or security.
