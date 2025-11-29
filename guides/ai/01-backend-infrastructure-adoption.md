# Backend Infrastructure Adoption Guide

**Purpose:** Document the decision to adopt the svc-merchants microservice template as the backend infrastructure for AI agentic products.

**Status:** Work in Progress  
**Created:** November 2025  
**Last Updated:** November 2025

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Background](#background)
3. [Comparison Analysis](#comparison-analysis)
4. [Template Advantages](#template-advantages)
5. [Required Additions for AI Agents](#required-additions-for-ai-agents)
6. [Recommendations](#recommendations)
7. [Template Evolution Path](#template-evolution-path)
8. [Decision Record](#decision-record)

---

## Executive Summary

The `svc-merchants` microservice template has been evaluated and **approved as the backend infrastructure foundation** for all AI agentic products. This template provides a production-grade AWS CDK architecture that is superior to generic AI-suggested implementations.

### Key Decision

**Use the svc-merchants template** (converted to a generic template) instead of:

- Claude's recommendations in the AI strategy guide
- AI-generated boilerplate
- Third-party services like Clerk for auth

### Rationale

1. **Battle-tested patterns** - Months of refinement
2. **Unified AWS ecosystem** - No vendor fragmentation
3. **Comprehensive documentation** - Template-ready guides
4. **Extensible architecture** - Constructs, feature flags, typed config

---

## Background

### The AI Agentic Products Strategy

A comprehensive strategy was developed (see `ai-agentic-products-strategy/`) for building a portfolio of AI agent products:

- **Product #1:** LinkedIn Ghostwriter Agent (Months 1-4)
- **Product #2:** Integration-Heavy Agent (Months 5-8)
- **Product #3:** Complex Multi-Agent System (Months 9-12)

### The Technology Decisions Guide

The strategy included a technology decisions guide (`03-technical-stack/3.1-technology-decisions.md`) recommending:

| Component       | Recommendation                           |
| --------------- | ---------------------------------------- |
| Frontend        | SvelteKit                                |
| Backend Hosting | Vercel (Frontend) + AWS Lambda (Backend) |
| Auth            | Clerk                                    |
| IaC             | AWS CDK (TypeScript)                     |
| Database        | DynamoDB                                 |
| API             | SvelteKit API Routes + AWS Lambda        |
| Orchestration   | AWS Step Functions                       |

### The svc-merchants Template

The `svc-merchants` directory is a microservice implementation that was designed to be extracted as a GitHub template. It contains:

- AWS CDK infrastructure (TypeScript)
- Cognito authentication (User Pool, Identity Pool, Groups)
- DynamoDB database (Faux-SQL and Single-Table patterns)
- API Gateway with Lambda endpoints
- Comprehensive documentation

---

## Comparison Analysis

### Claude's Recommendations vs. Template Implementation

| Component      | Claude's Suggestion           | svc-merchants Template                                            | Verdict                |
| -------------- | ----------------------------- | ----------------------------------------------------------------- | ---------------------- |
| **IaC**        | AWS CDK (TypeScript)          | AWS CDK with L3 constructs, Zod validation, typed config          | **Template is better** |
| **Auth**       | Clerk                         | Cognito User Pool + Identity Pool + User Groups + Lambda triggers | **Template is better** |
| **Database**   | DynamoDB (single-table only)  | DynamoDB with Faux-SQL OR Single-Table (configurable)             | **Template is better** |
| **API**        | SvelteKit API Routes + Lambda | API Gateway + Lambda with authorization patterns                  | **Template is better** |
| **Hosting**    | Vercel + AWS split            | Unified AWS ecosystem                                             | **Template is better** |
| **Monitoring** | CloudWatch + Sentry           | CloudWatch + SNS alarms                                           | **Comparable**         |
| **CI/CD**      | Not specified                 | AWS CodePipeline (ready)                                          | **Template has more**  |

### Why Clerk is Not Recommended

Claude recommended Clerk for simplicity, but Cognito offers:

| Feature                         | Clerk    | Cognito (Template)      |
| ------------------------------- | -------- | ----------------------- |
| Vendor lock-in                  | Yes      | No (AWS-native)         |
| Identity Pool (AWS credentials) | No       | Yes                     |
| User Groups with IAM roles      | No       | Yes                     |
| Lambda triggers                 | Limited  | Full support            |
| Free tier at scale              | 10K MAUs | 50K MAUs                |
| Direct AWS resource access      | No       | Yes (via Identity Pool) |

**For AI agents:** Identity Pool is crucial—agents can assume roles to access AWS services directly without routing through your API.

### Why Unified AWS is Better Than Vercel + AWS Split

| Aspect                 | Vercel + AWS Split     | Unified AWS (Template) |
| ---------------------- | ---------------------- | ---------------------- |
| Deployment complexity  | Two platforms          | One platform           |
| Function timeout       | 10 seconds (Vercel)    | 15 minutes (Lambda)    |
| Cold starts            | Varies                 | Consistent             |
| Cost at scale          | Higher                 | Lower                  |
| Agent execution        | Requires Lambda anyway | Native                 |
| Infrastructure as Code | Partial                | Complete               |

---

## Template Advantages

### 1. Production-Grade Architecture

The template has battle-tested patterns:

```
ServiceStack
├── SsmBindingsConstruct (reads external service configs)
├── MonitorConstruct (CloudWatch alarms, SNS topics)
├── DatabaseConstruct (DynamoDB table)
├── AuthConstruct (Cognito User Pool, Identity Pool, Groups)
├── IamConstruct (IAM roles for authenticated users)
├── PermissionsConstruct (OAuth scopes, resource server) [optional]
├── ApiConstruct (API Gateway, Lambda endpoints)
└── SsmPublicationsConstruct (publishes service configs to SSM)
```

### 2. Typed Configuration with Validation

```typescript
// config/default.ts - Zod-validated configuration
const result = ConfigSchema.safeParse(config);
if (!result.success) {
  const formatted = result.error.issues
    .map((i) => `${i.path.join(".")}: ${i.message}`)
    .join("\n");
  throw new Error(`Invalid configuration for env '${envName}':\n${formatted}`);
}
```

### 3. Feature Flags

```typescript
// Conditional construct deployment
const permissions: IPermissionsProvider = config.features?.permissionsEnabled
  ? new PermissionsConstruct(this, "PermissionsConstruct", { ... })
  : new NoopPermissionsConstruct();
```

### 4. Environment-Aware Configuration

```typescript
// Supports: local, localstack, staging, production
switch (envName) {
  case "localstack":
    config = { ...defaultConfig, ...localstackConfig };
    break;
  case "staging":
    config = { ...defaultConfig, ...stagingConfig };
    break;
  case "production":
    config = { ...defaultConfig, ...productionConfig };
    break;
}
```

### 5. Flexible Database Patterns

```typescript
// Configurable approach
if (approach === "faux-sql") {
  // Multiple tables with descriptive keys
  const db = new DdbFauxSqlConstruct(this, "DdbFauxSql", { config });
} else {
  // Single table with generic PK/SK
  const db = new DdbSingleTableConstruct(this, "DdbSingleTable", { config });
}
```

### 6. Comprehensive Documentation

```
docs/
├── implementation/
│   ├── adding-endpoints-part-1-lambda-handlers.md
│   ├── adding-endpoints-part-2-api-gateway.md
│   ├── authentication.md
│   ├── configuration-management/
│   ├── data-access.md
│   ├── database-setup.md
│   └── ...
├── testing/
│   ├── handler-testing-guide.md
│   ├── cdk-template-testing-guide.md
│   ├── e2e-testing-guide.md
│   └── ...
└── DOCUMENTATION-MAP.md
```

---

## Required Additions for AI Agents

### 1. LangGraph Integration Point

Add a construct for long-running agent execution:

```typescript
// lib/agents/construct.ts (new)
interface IAgentsConstructProps {
  readonly config: IConfig;
  readonly db: DatabaseConstruct;
}

class AgentsConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IAgentsConstructProps) {
    super(scope, id);

    // Lambda with 15-min timeout for agent execution
    // SQS queue for async agent invocation
    // DLQ for failed agent runs
  }
}
```

### 2. Step Functions for Multi-Agent Orchestration

For Product #3 (Complex Multi-Agent System):

```typescript
// lib/orchestration/construct.ts (new)
class OrchestrationConstruct extends Construct {
  constructor(
    scope: Construct,
    id: string,
    props: IOrchestrationConstructProps
  ) {
    super(scope, id);

    // State machine for agent orchestration
    // Parallel execution patterns
    // Error handling and retries
    // Human-in-the-loop approval states
  }
}
```

### 3. Vector Database Integration

For semantic search in Products #2 and #3. **Both options should be available as constructs** to allow selection based on project needs.

#### Option A: Pinecone Construct

**Best for:** Development, low-cost prototyping, free tier usage

```typescript
// lib/vector-db/pinecone/construct.ts (new)
class PineconeConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IPineconeConstructProps) {
    super(scope, id);

    // Pinecone API key in Secrets Manager
    // Lambda layer with Pinecone SDK
    // Helper functions for upsert/query
  }
}
```

**Pricing:**

- Free tier: 1 index, 100K vectors, unlimited queries
- Starter: $70/month for 1M vectors

#### Option B: OpenSearch Serverless Construct

**Best for:** Production, AWS-native unified approach, enterprise scale

```typescript
// lib/vector-db/opensearch/construct.ts (new)
class OpenSearchConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IOpenSearchConstructProps) {
    super(scope, id);

    // OpenSearch Serverless collection
    // Vector search index configuration
    // IAM policies for Lambda access
  }
}
```

**Pricing:**

- No free tier (minimum ~$700/month for always-on)
- Pay-per-use with auto-scaling

#### Selection Criteria

| Criteria            | Pinecone              | OpenSearch Serverless  |
| ------------------- | --------------------- | ---------------------- |
| **Free tier**       | ✅ Yes (100K vectors) | ❌ No                  |
| **Dev cost**        | ✅ Free               | ❌ ~$700/month minimum |
| **AWS-native**      | ❌ External           | ✅ Yes                 |
| **Unified billing** | ❌ Separate           | ✅ AWS consolidated    |
| **IAM integration** | ❌ API key            | ✅ Native IAM          |
| **Latency**         | Good                  | Better (same VPC)      |
| **Scale**           | Good                  | Enterprise-grade       |

#### Recommendation

- **Development/MVP:** Use Pinecone (free tier)
- **Production/Enterprise:** Use OpenSearch Serverless (unified AWS)
- **Template:** Include both constructs with feature flag selection

```typescript
// config/features.ts
vectorDb: {
  enabled: true,
  provider: "pinecone" | "opensearch"
}
```

### 4. Stripe Payment Integration

```typescript
// lib/payments/construct.ts (new)
class PaymentsConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IPaymentsConstructProps) {
    super(scope, id);

    // Webhook endpoint Lambda
    // Stripe secret in Secrets Manager
    // Subscription management
    // Usage-based billing tracking
  }
}
```

### 5. External API Integration Patterns

For Product #2 (Integration-Heavy Agent):

```typescript
// lib/integrations/construct.ts (new)
class IntegrationsConstruct extends Construct {
  // API key storage (Secrets Manager)
  // Rate limiting (API Gateway usage plans)
  // Retry policies
  // Circuit breaker patterns
}
```

---

## Recommendations

### Use Your Template, Not AI-Generated Code

| Aspect            | AI-Generated  | Your Template          |
| ----------------- | ------------- | ---------------------- |
| Context awareness | Generic       | Specific to your needs |
| Integration       | Often poor    | Already integrated     |
| Documentation     | Minimal       | Comprehensive          |
| Testing           | Often missing | Full test suite        |
| Patterns          | Inconsistent  | Consistent style       |

### When to Use AI Assistance

✅ **Do use AI for:**

- Adding new constructs following your patterns
- Generating boilerplate that matches your style
- Reviewing and improving existing code
- Suggesting optimizations
- Writing tests

❌ **Don't use AI for:**

- Replacing your architecture
- Generating entire services from scratch
- Making architectural decisions without review

---

## Template Evolution Path

### Phase 1: Product #1 (LinkedIn Ghostwriter)

**Timeline:** Months 1-4

**Template additions:**

- Single agent Lambda with LangGraph
- Stripe payment construct
- Usage tracking (credits system)

**Architecture:**

```
ServiceStack
├── [existing constructs]
├── AgentConstruct (single agent execution)
└── PaymentsConstruct (Stripe integration)
```

### Phase 2: Product #2 (Integration-Heavy Agent)

**Timeline:** Months 5-8

**Template additions:**

- Step Functions construct (basic)
- External API integration patterns
- S3 for scraped data storage
- Rate limiting patterns

**Architecture:**

```
ServiceStack
├── [existing constructs]
├── AgentConstruct
├── PaymentsConstruct
├── OrchestrationConstruct (basic Step Functions)
└── IntegrationsConstruct (external APIs)
```

### Phase 3: Product #3 (Multi-Agent System)

**Timeline:** Months 9-12

**Template additions:**

- Full Step Functions orchestration
- Parallel agent execution
- Vector database integration
- Human-in-the-loop patterns

**Architecture:**

```
ServiceStack
├── [existing constructs]
├── AgentConstruct (multiple agents)
├── PaymentsConstruct
├── OrchestrationConstruct (full orchestration)
├── IntegrationsConstruct
└── VectorDbConstruct (semantic search)
```

---

## Decision Record

### ADR-001: Adopt svc-merchants as Backend Template

**Status:** Accepted

**Context:**

- Need backend infrastructure for AI agentic products
- AI strategy guide recommended Vercel + Clerk + AWS Lambda
- Existing svc-merchants template available with production patterns

**Decision:**
Adopt svc-merchants as the backend template for all AI agentic products.

**Consequences:**

**Positive:**

- Unified AWS ecosystem (no vendor fragmentation)
- Production-grade patterns already implemented
- Comprehensive documentation
- Extensible architecture
- No additional learning curve

**Negative:**

- Need to convert to generic template
- Need to add AI-specific constructs
- Cognito more complex than Clerk initially

**Alternatives Considered:**

1. **Follow AI strategy recommendations (Vercel + Clerk)**

   - Rejected: Fragmented ecosystem, vendor lock-in, shorter timeouts

2. **Generate new template with AI**

   - Rejected: Loses context, inconsistent patterns, poor integration

3. **Use serverless framework**
   - Rejected: Less flexible than CDK, different language

---

## Next Steps

1. **Create template copy** - Copy svc-merchants to new location for template conversion
2. **Remove merchant-specific code** - Replace with template handles
3. **Update documentation** - Make all docs template-ready
4. **Add AI constructs** - Implement agent, orchestration, payments constructs
5. **Create configuration guide** - Document all template handles

See: [Template Conversion Plan](./02-template-conversion-plan.md)

---

## Related Documents

- [AI Agentic Products Strategy](../../../business/docs/ai-agentic-products-strategy/)
- [Technology Decisions Guide](../../../business/docs/ai-agentic-products-strategy/03-technical-stack/3.1-technology-decisions.md)
- [Template Conversion Plan](./02-template-conversion-plan.md) (WIP)
- [svc-merchants Documentation](../../svc-merchants/docs/)
