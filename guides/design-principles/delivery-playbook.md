# Delivery Playbook for Serverless Full-Stack Teams

## Purpose & Audience

- **Who** This plays to product, design, frontend, backend, platform, and QA contributors shipping features on our serverless stack.
- **Why** Provide a single reference that marries the delivery methodology with the AWS-based architecture patterns captured across existing guides.
- **How** Connect each step of the lifecycle to the specific artefacts, tools, and guardrails defined in:
  - `docs/guides/design-principles/serverless-microservices-architecture-v3.md`
  - `docs/guides/design-principles/design-and-development-methodology-v2.md`
  - `docs/project/` planning artefacts and `docs/specs/` delivery assets

## Core Principles

- **Actor & job-led discovery** Ground every story in the user jobs captured in `docs/project/actors.md` and backlog entries.
- **Architecture-aligned delivery** Follow the frontend-embedded BFF, single-table DynamoDB, and serverless microservice tenets in `serverless-microservices-architecture-v3.md`.
- **Contract-first collaboration** Lock API expectations early via OpenAPI specs to enable parallel work.
- **Observable & secure by default** Instrument logs/metrics, enforce IAM least privilege, and keep internal APIs private.
- **Documentation as code** Treat diagrams, specs, and runbooks as version-controlled assets that evolve with the system.

## Architecture Snapshot (See Section 2 of `serverless-microservices-architecture-v3.md`)

- **Frontend & BFF** SvelteKit routes (`+page.svelte`, `+page.server.ts`, `+server.ts`) deployed via SST on AWS Lambda + CloudFront + S3.
- **Microservices** Independent AWS Lambda functions with CDK-defined stacks, exposing internal-only APIs.
- **Data layer** Per-service DynamoDB single-table schemas, optional S3 buckets, strict ownership boundaries.
- **Async flows** SNS/SQS and EventBridge for decoupled events, Step Functions for orchestrated transactions.
- **Service discovery & config** AWS Systems Manager Parameter Store, Secrets Manager for sensitive data.

## Delivery Lifecycle

### Phase 0 – Product Discovery & Intake

- **Inputs** Vision in `docs/project/project-plan.md`, feature backlog in `docs/project/feature-backlog.md`.
- **Key steps**
  - Identify actors/jobs (`docs/project/actors.md`).
  - Draft job story cards in `docs/backlog/job-stories/` (When/I want/So that).
  - Prioritize and move into status board (`docs/project/status-board.md`).
- **Outputs** Ready-to-sequence job with acceptance criteria, success metrics, and dependencies.

### Phase 1 – Experience & Domain Design

- **Scenario articulation** Extend job cards with business rules and scenarios (`docs/specs/jobs/<job>/job-card.md`).
- **UI mockups** Produce flows in Figma/Excalidraw; store exports in `docs/specs/jobs/<job>/mockups/` with annotations.
- **ERD scaffolding** Draft conceptual relationships using `docs/guides/data-modeling/erd-creation-guide.md`; output to `docs/specs/jobs/<job>/erd.puml`
- **Access patterns** Follow `docs/guides/data-modeling/access-patterns-definition-guide.md` to complete `docs/specs/jobs/<job>/data-access-patterns.md`

### Phase 2 – Technical Design & Contracts

- **Sequence diagrams** Capture UI ⇄ BFF ⇄ microservice responsibilities in `docs/specs/jobs/<job>/sequence-diagram.puml` (see pattern guidance in Section 4 of the architecture guide).
- **Action vs query catalog** Document CQS split in `docs/specs/jobs/<job>/actions-queries.md`.
- **Data model specification** Translate ERD and data access patterns into:

  1. Primary key structure and secondary indexes following `docs/guides/data-modeling/primary-key-design-guide.md`
  2. Entity schemas (TypeScript types) capturing all non-key attributes in `docs/specs/jobs/<job>/entity-schemas.ts`

  Output: `docs/specs/jobs/<job>/data-model-spec.md` and `docs/specs/jobs/<job>/entity-chart.md`

- **API specification** Publish OpenAPI in `docs/specs/jobs/<job>/api.yml`; use shared components and reference auth schemes from the architecture guide.
- **Readiness check** Ensure Definition of Ready items (DoR) from `design-and-development-methodology-v2.md` are met before implementation.

### Phase 3 – Validation & Alignment

- **Design reviews** Validate mockups and flows with stakeholders; log decisions in `docs/specs/jobs/<job>/feedback-log.md`.
- **Technical walkthrough** Review contracts with affected microservice teams; reconcile implementation plan and capacity.
- **Compliance** Confirm non-functional requirements (latency, observability, security) are captured.

### Phase 4 – Implementation & Testing

- **Frontend & BFF**
  - Build UI components in SvelteKit routes under `src/routes/`.
  - Implement BFF APIs in `src/routes/api/**/+server.ts`, orchestrating microservice calls using AWS SDK credentials (see Section 4 of the architecture guide).
- **Microservices**
  - Extend CDK stacks (`lib/service-stack.ts`) with required Lambdas, DynamoDB tables, queues, etc.
  - Implement business logic handlers, transactions (e.g., DynamoDB `TransactWriteItems`), and publish domain events.
- **Testing strategy**
  - Unit: Vitest/Jest for SvelteKit and Lambda handlers.
  - Integration: LocalStack for AWS services, contract tests for OpenAPI compliance.
  - E2E: Playwright/Cypress hitting deployed BFF endpoints.
- **Definition of Done** Validate against DoD checklist in `design-and-development-methodology-v2.md` (tests, docs, monitoring, deployment).

### Phase 5 – Deployment, Operations & Continuous Improvement

- **CI/CD**
  - Leverage SST/CDK pipelines for infrastructure updates.
  - Enforce lint/type checks, unit tests, and contract tests on PRs.
- **Observability**
  - Collect structured logs, metrics, and traces per architecture guidance.
  - Configure CloudWatch alarms, dashboards, and DLQs for async workloads.
- **Runbooks & retros**
  - Maintain operational guides in `docs/runbooks/`.
  - Capture learnings back into this playbook and linked artefacts.

## Artefact Mapping

| Lifecycle Phase   | Primary Artefacts                                                                                                                                                                                                                                          | Supporting References                                                           |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Discovery         | `docs/backlog/job-stories/`, `docs/project/status-board.md`                                                                                                                                                                                                | `docs/project/actors.md`                                                        |
| Experience Design | `docs/specs/jobs/<job>/mockups/`, `docs/specs/jobs/<job>/job-card.md`                                                                                                                                                                                      | `design-and-development-methodology-v2.md`                                      |
| Technical Design  | `docs/specs/jobs/<job>/sequence-diagram.puml`, `docs/specs/jobs/<job>/actions-queries.md`, `docs/specs/jobs/<job>/api.yml`, `docs/specs/jobs/<job>/data-model-spec.md`, `docs/specs/jobs/<job>/entity-chart.md`, `docs/specs/jobs/<job>/entity-schemas.ts` | `serverless-microservices-architecture-v3.md`, `docs/guides/data-modeling/*.md` |
| Implementation    | `src/routes/**`, `src/routes/api/**/+server.ts`, `services/*/lib/**`, `infrastructure/*`                                                                                                                                                                   | `serverless-microservices-architecture-v3.md`, CDK constructs                   |
| Testing & Ops     | Test suites, `docs/runbooks/`, monitoring dashboards                                                                                                                                                                                                       | DoD checklist                                                                   |

## Tooling & Environment Expectations

- **SST & AWS CDK** for IaC, deployed via automated pipelines.
- **LocalStack** for local AWS service simulation; align test data with `docs/specs/jobs/<job>/data-access-patterns.md`.
- **Prism/MSW** optional for mocking OpenAPI contracts during frontend development.
- **GitHub Actions (or equivalent)** to run linting, tests, CDK diff/synth.

## Governance & Quality Gates

- **Definition of Ready** must be satisfied before coding (story completeness, diagrams, API draft, data mapping).
- **Definition of Done** includes merged code, passing automated checks, updated documentation, and observability hooks.
- **Change management** Significant architectural changes require ADRs in `docs/adr/` and updates to both this playbook and the architecture guide.

## How to Use This Playbook

1. **Start here** when planning a new job/story to ensure outputs from each phase are captured.
2. **Follow cross-links** to deeper guidance (architecture, data modeling, API standards).
3. **Update alongside work** Treat this document as living—submit PRs when teams adopt new patterns or tooling.

---

_Last updated: {{DATE}} (replace with actual edit date when updating)._
