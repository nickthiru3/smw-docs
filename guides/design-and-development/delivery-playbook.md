# Quick Start: Delivery Playbook

**A concise guide to shipping features on our serverless full-stack**

## Who This Is For

Product owners, designers, frontend developers, backend engineers, platform teams, and QA contributors working on our SvelteKit + AWS serverless stack.

## Core Principles

1. **Customer-centric design** - Start with actors and their stories to be done
2. **Contract-first collaboration** - Define API contracts early to enable parallel work
3. **Architecture-aligned delivery** - Follow established patterns for frontend BFF, microservices, and data layer
4. **Observable & secure by default** - Instrument logs/metrics, enforce IAM least privilege
5. **Documentation as code** - Version-control all design artifacts alongside implementation

## The 4-Phase Delivery Lifecycle

### Phase 1: Requirements (Product Discovery)

**Goal:** Identify users and their needs, create prioritized backlog

**Key Activities:**

- Identify actors and their stories to be done
- Create story cards (When/I want/So that - Job Story Format)
- Prioritize backlog

**Outputs:** Story cards ready for design

### Phase 2: Conceptual Design

**Goal:** Understand what to build from user and data perspectives

**Key Activities:**

- Create UI mockups for all user flows
- Draft ERD showing entities and relationships
- Document access patterns from UI requirements
- Define data model (entities, keys, relationships)

**Outputs:** Mockups, ERD, entity files, access patterns

### Phase 3: API Design & Contracts

**Goal:** Define component interactions and API contracts

**Key Activities:**

- Create sequence diagrams (UI ↔ BFF ↔ Backend)
- Separate actions from queries (CQS principle)
- Publish OpenAPI specification

**Outputs:** Sequence diagrams, actions/queries doc, OpenAPI spec

### Phase 4: Implementation & Testing

**Goal:** Build, test, and deploy the feature

**Key Activities:**

- Frontend: Build UI components and BFF API routes
- Backend: Implement microservice logic and data access
- QA: Execute test plans (unit, integration, E2E)

**Outputs:** Working feature deployed to production

## Comprehensive Guides

For detailed step-by-step instructions, refer to:

### 📘 [Design & Development Methodology](./design-and-development-methodology-v2.md)

**Complete workflow guide covering:**

- Detailed steps for each phase
- Team roles and responsibilities
- Artifact templates and examples
- Definition of Ready and Definition of Done
- Best practices and conventions

### 🏗️ [Serverless Microservices Architecture](./serverless-microservices-architecture-v3.md)

**Technical architecture patterns:**

- Frontend & BFF architecture (SvelteKit)
- Microservices design patterns
- Data layer strategies (DynamoDB)
- Async communication (SNS/SQS, EventBridge)
- Security, observability, and deployment

## Quick Reference: Key Artifacts

| Phase       | Primary Artifacts           | Location                                                                                            |
| ----------- | --------------------------- | --------------------------------------------------------------------------------------------------- |
| **Phase 1** | actors, story map           | `docs/project/specs/actors.md`<br>`docs/project/specs/story-map.md`                                 |
| **Phase 2** | Mockups, ERD, entity files  | `docs/specs/jobs/<job>/mockups/`<br>`docs/project/specs/erd.puml`<br>`docs/project/specs/entities/` |
| **Phase 3** | Sequence diagrams, API spec | `docs/specs/jobs/<job>/sequence-diagrams/`<br>`docs/specs/jobs/<job>/api.yml`                       |
| **Phase 4** | Code, tests, runbooks       | `src/routes/`<br>`services/`<br>`docs/runbooks/`                                                    |

## Getting Started

1. **New to the team?** Read the [Design & Development Methodology](./design-and-development-methodology-v2.md) for complete onboarding
2. **Starting a new feature?** Follow Phase 1 → Phase 2 → Phase 3 → Phase 4
3. **Need architecture guidance?** Consult the [Serverless Microservices Architecture](./serverless-microservices-architecture-v3.md)
4. **Questions?** Reach out to your team lead or check existing job specs in `docs/specs/jobs/` for examples

**Happy shipping!** 🚀
