# AI Agentic Products Guides

**Purpose:** Documentation for building AI agentic products using the AWS CDK microservice template.

**Status:** Work in Progress  
**Created:** November 2025

---

## Overview

This directory contains guides for:

1. **Adopting the backend infrastructure** - Using svc-merchants as the template base
2. **Converting to a reusable template** - Removing service-specific code
3. **Adding AI capabilities** - Agent execution, orchestration, payments
4. **Building AI products** - Product-specific implementation guides

---

## Guide Index

### Foundation

| Guide                                                                            | Status    | Description                                          |
| -------------------------------------------------------------------------------- | --------- | ---------------------------------------------------- |
| [01-backend-infrastructure-adoption.md](./01-backend-infrastructure-adoption.md) | ✅ Draft  | Decision to adopt svc-merchants as backend template  |
| [02-template-conversion-plan.md](./02-template-conversion-plan.md)               | ✅ Draft  | Plan for converting to reusable GitHub template      |
| [03-template-conversion-progress.md](./03-template-conversion-progress.md)       | 🔄 Active | Progress log for template conversion work            |
| [04-template-comparison.md](./04-template-comparison.md)                         | ✅ Draft  | Comparison of svc-merchants vs microservice-template |

### AI Capabilities (To Be Created)

| Guide                           | Status     | Description                              |
| ------------------------------- | ---------- | ---------------------------------------- |
| 05-agent-execution-patterns.md  | ⬜ Planned | LangGraph integration, Lambda patterns   |
| 06-orchestration-patterns.md    | ⬜ Planned | Step Functions for multi-agent systems   |
| 07-payment-integration.md       | ⬜ Planned | Stripe subscription and usage billing    |
| 08-vector-search-integration.md | ⬜ Planned | Semantic search with Pinecone/OpenSearch |

### Product Guides (To Be Created)

| Guide                            | Status     | Description               |
| -------------------------------- | ---------- | ------------------------- |
| products/linkedin-ghostwriter.md | ⬜ Planned | Product #1 implementation |
| products/integration-agent.md    | ⬜ Planned | Product #2 implementation |
| products/multi-agent-system.md   | ⬜ Planned | Product #3 implementation |

---

## Related Documentation

- **AI Strategy:** `business/docs/ai-agentic-products-strategy/`
- **Original Sources:**
  - `svc-merchants/` (do not modify)
  - `microservice-template/` (external, do not modify)
- **Working Copies:**
  - `svc-users-template/` - Users/auth service template (from svc-merchants)
  - `svc-ai-template/` - Generic AI microservice template (from microservice-template)

---

## How to Use These Guides

### If you're starting fresh:

1. Read [01-backend-infrastructure-adoption.md](./01-backend-infrastructure-adoption.md) to understand the architecture decisions
2. Follow [02-template-conversion-plan.md](./02-template-conversion-plan.md) to create your template
3. Use the AI capability guides to add features

### If you're building a specific product:

1. Use the template (once created)
2. Follow the relevant product guide
3. Reference capability guides as needed

---

## Contributing

These guides are living documents. Update them as:

- Decisions are made
- Implementation progresses
- Lessons are learned

Use the progress tracking sections to mark completed tasks.
