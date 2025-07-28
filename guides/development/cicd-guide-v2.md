# CI/CD Guide v2: Simplified Trunk-Based Strategy for Microservices

## Table of Contents

- [Introduction](#introduction)
- [Strategy Overview](#strategy-overview)
- [Core Concepts](#core-concepts)
- [LocalStack Integration](#localstack-integration-for-local-development)
- [GitHub Actions Integration](#github-actions-integration-for-tag-based-deployments)
- [Simplified Trunk-Based Workflow](#simplified-trunk-based-release-workflow)
- [Teams Coordination](#teams-coordination--workflow)
- [Reference Implementation](#reference-implementation)
- [Migration Guide](#migration-from-v1-implementation)

## Introduction

This guide outlines a **simplified trunk-based CI/CD strategy** specifically designed for **small team microservices development**. After extensive research into various branching strategies (Three-Flow, GitLab Flow, Release Flow, Vercel's Three-Branch Strategy), we've adopted a pragmatic approach that balances simplicity with enterprise-grade practices.

**Key Design Principles:**

- **Simplicity over complexity**: Minimize branches and merge conflicts
- **Speed over ceremony**: Fast path from development to production
- **Quality gates**: Code review and testing at every step
- **Cost optimization**: LocalStack for development, AWS for staging/production
- **Team size optimization**: Perfect for small, focused microservice teams

## Strategy Overview

### Why Simplified Trunk-Based?

**Traditional Complex Strategies Issues:**

- ❌ Too many branches (master, candidate, release, feature)
- ❌ Complex merge strategies and coordination overhead
- ❌ Slow feedback loops and documentation inconsistencies

**Our Simplified Approach Benefits:**

- ✅ Two environments: LocalStack (local) → Staging → Production
- ✅ One main branch: All integration happens on `main`
- ✅ Simple versioning: Semantic versioning with `v*` tags
- ✅ Fast delivery: Direct path from merge to production
- ✅ Small team optimized: Perfect for microservice development

### Architecture Comparison

| Aspect            | Complex Strategies           | Our Simplified Approach               |
| ----------------- | ---------------------------- | ------------------------------------- |
| **Branches**      | 3-4 branches                 | 1 main + feature branches             |
| **Environments**  | 4 environments               | 3 environments                        |
| **Tagging**       | Multiple patterns            | Single pattern (v\*)                  |
| **Team Roles**    | Multiple specialized roles   | Developers + optional release manager |
| **Feedback Loop** | Long path                    | Short path                            |
| **Cost**          | High (multiple AWS accounts) | Low (LocalStack + 2 accounts)         |

## Core Concepts

### Branch Strategy

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Feature Branch │ →  │   Main Branch   │ →  │   Production    │
│   (LocalStack)  │    │   (Staging)     │    │   (Tagged)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Environment Strategy

| Environment    | Source           | Trigger      | Purpose                | AWS Account |
| -------------- | ---------------- | ------------ | ---------------------- | ----------- |
| **Local**      | Feature branches | Manual       | Individual development | LocalStack  |
| **Staging**    | `main` branch    | Auto (push)  | Integration testing    | Current AWS |
| **Production** | `main` branch    | Manual (tag) | Live deployment        | Future AWS  |

### Versioning Strategy

**Semantic Versioning (SemVer):**

- **Format**: `v1.2.3` (major.minor.patch)
- **Major**: Breaking changes (v1.0.0 → v2.0.0)
- **Minor**: New features (v1.0.0 → v1.1.0)
- **Patch**: Bug fixes (v1.0.0 → v1.0.1)

### Team Workflow

**Daily Development:**

1. Create feature branch from `main`
2. Develop locally with LocalStack
3. Create merge request when ready
4. Code review by team member
5. Merge to main → automatic staging deployment
6. QA testing in staging environment
7. Create release tag → production deployment

## LocalStack Integration for Local Development

### Overview

LocalStack provides a fully functional local AWS cloud stack, enabling developers to develop and test their cloud applications offline.

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

## GitHub Actions Integration for Tag-Based Deployments

### Overview

We use **GitHub Actions** to orchestrate our CI/CD pipeline, with **tag-based deployments** for production releases.

### Workflow Configuration

```yaml
name: Simplified Microservice CI/CD

on:
  push:
    branches: [main]
    tags: ["v*"]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"
          cache: "npm"
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: Deploy to Staging
        run: |
          aws codepipeline start-pipeline-execution \
            --name "super-deals-deals-ms-staging-pipeline"

  deploy-production:
    needs: test
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: Extract version from tag
        id: version
        run: |
          VERSION=${GITHUB_REF#refs/tags/v}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
      - name: Deploy to Production
        run: |
          aws codepipeline start-pipeline-execution \
            --name "super-deals-deals-ms-production-pipeline"
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
