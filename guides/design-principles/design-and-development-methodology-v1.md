# Design and Development Methodology

Version: Draft 1

Purpose
- Capture the story-driven methodology we are using to design and implement Super-Deals features.
- Serve as a living reference for cross-functional teams (PO, FE, BFF, BE, QA) as we iterate.

Scope
- Focuses on practical, repeatable steps from story conception to working software.
- Optimized for our v3 architecture: frontend-embedded BFF (SvelteKit `+server.ts`) orchestrating private microservices.

## Core Principles
- Story-first, scenario-driven: Requirements are expressed through user stories, business rules, and concrete scenarios.
- Contract-first integration: APIs are specified early to enable parallel work.
- Visual clarity: Sequence diagrams to reveal responsibilities and edge cases.
- Data consciousness: Distinguish payloads “over the wire” from persisted entities.
- Incremental rigor: Start simple; add automation and gates as the system matures.

## Planning Artefacts & Flow
- **Project plan** (`docs/project/project-plan.md`): Captures vision, roadmap phases, risks, and open questions.
- **Feature backlog** (`docs/project/feature-backlog.md`): User-type oriented job list; select the next candidate from here.
- **Status board** (`docs/project/status-board.md`): Lightweight Kanban (To Do/In Progress/Blocked/Done). Move the chosen job into **To Do**, then advance it as work progresses.

Flow summary:
1. Prioritise within the feature backlog.
2. Move the selected job into the status board **To Do** column.
3. Produce job artefacts (card, diagrams, contracts, etc.).
4. Implement, test, and deploy; transition the job across status columns until **Done**.

## Artifacts and Their Intent
0) Job card (Markdown)
   - Location: `docs/specs/jobs/.../job.md` (legacy references in `docs/specs-OLD/jobs/` remain for guidance).
   - Contents: Job statement (who/wants/so that), business rules, early assumptions, links to supporting artefacts.

1) Story file (Markdown) *(optional when the job card already captures story detail)*
   - Location: `specs/stories/.../story.md`
   - Contents: Title, story description, business rules (acceptance criteria), scenarios (success + failure variants).

2) Scenario sequence diagram (PlantUML)
   - Location: `specs/stories/.../scenarios/*.puml`
   - Purpose: Clarify interactions among UI, BFF (`+server.ts`), and microservices; note validations and enrichments.

3) API specification (OpenAPI)
   - Location: `specs/stories/.../api.yml`
   - Purpose: Formal contract for request/response, errors, and security for the scenario’s endpoints.

4) Data model (ER diagram)
   - Location: `specs/er-diagram.drawio`
   - Purpose: Persisted entities, constraints, and generated fields (ids, timestamps). May evolve independently from API shapes.

5) Implementation stubs
   - BFF route: SvelteKit `+server.ts` implementing the endpoint(s) per spec.
   - Microservice endpoint(s): Handler(s) and CDK wiring for private access.

6) Tests (high level for now)
   - Unit: input validation, mapping, basic domain rules.
   - Integration (LocalStack): End-to-end path for happy-flow scenario.

End-to-End Workflow (per Story)
1) Author/Refine Story
   - Write `story.md`: Title, description, business rules, scenarios.
   - Ensure scenarios cover at least: primary success, key validation failures, auth/authorization failures.

2) Visualize Scenario(s)
   - Create/update `*.puml` sequence diagram for each scenario.
   - Model BFF responsibilities vs microservice responsibilities; mark validation/enrichment points.

3) Specify API Contract
   - Draft/extend `api.yml` with the endpoint(s) for the scenario.
   - Define request body, required fields, and initial response shapes; list security requirements.

4) Align Data Model
   - Update ER diagram for new/modified entities.
   - Note generated fields and constraints relevant to the scenario.

5) Implement Iteratively
   - BFF: Add `+server.ts` route(s) matching `api.yml`.
   - Microservice: Implement handler(s) and internal routing; keep it private and callable only by the BFF.
   - Wire configuration for LocalStack vs AWS staging.

6) Test and Demonstrate
   - Unit tests for validators/mappers.
   - LocalStack-backed integration test for the success scenario.
   - Demo against BFF endpoint; confirm acceptance criteria.

Definitions
- Definition of Ready (DoR)
  - `story.md` with business rules and at least one success + key failure scenarios.
  - Scenario diagram for the first scenario.
  - Initial `api.yml` with request schema and provisional responses.
  - ER note of any new/changed entities and generated fields.

- Definition of Done (DoD)
  - BFF and microservice changes implemented for the first scenario.
  - Tests pass locally (unit + LocalStack integration for success scenario).
  - `api.yml`, `*.puml`, and `story.md` updated to reflect the final behavior.
  - Basic logs added for traceability (request id, principal, outcome).

Parallelization Guidance
- Once `api.yml` is drafted, FE/BFF and BE can proceed in parallel.
- FE can integrate against BFF stub/mocks while BE finalizes handlers.
- QA uses story + `api.yml` to draft acceptance tests early.

Backlog of Enhancements (to be adopted incrementally)
- Contract execution and validation
  - Add response schemas, examples, and reusable components in OpenAPI.
  - Introduce contract tests (provider verification) and lightweight consumer checks in BFF.
  - Optional mock server (e.g., Prism) for FE and smoke tests.

- Type and validator generation
  - Generate TS types from OpenAPI/JSON Schema for BFF and microservice.
  - Generate request validators in BFF to standardize error handling.

- CI/CD quality gates
  - Block breaking OpenAPI changes without versioning/approval.
  - Run LocalStack integration tests per feature tag/branch.

- Observability and security
  - Standardized error model and correlation IDs.
  - Explicit authz matrix per endpoint (who can/can’t) and claim expectations.
  - Idempotency strategy for mutating endpoints (headers/keys, 409 semantics).

- Documentation hygiene
  - Keep sequence diagrams in sync with BFF-in-frontend pattern.
  - Note non-functional expectations (latency, rate limits) where relevant.

Detailed Enhancements Backlog (verbatim from initial review)
1) Make contracts executable and enforced
   - Adopt contract tests:
     - Provider-side (microservice) tests to verify it satisfies OpenAPI.
     - Consumer-side (BFF) pact/contract tests or at least schema validation on responses.
   - Add JSON Schema examples and strict schemas in `api.yml`:
     - request/response examples
     - reusable components/schemas (`#/components/schemas/Deal`, `Error`)
     - error model with code, message, details
   - Add a mock server step:
     - Use Prism or WireMock to spin up a mock from `api.yml` for FE development and E2E smoke tests.

2) Tighten story/scenario quality
   - Use INVEST for stories; ensure acceptance criteria are SMART.
   - For each scenario, add:
     - Pre-conditions (auth context, merchant status)
     - Post-conditions (persistence, side effects, emitted events)
     - Non-functional expectations (latency, rate limits)
   - Explicitly tag security rules: authz matrix (who can/can’t create deals), JWT claims expectations, idempotency strategy.

3) Ensure diagrams reflect the new BFF-in-frontend architecture
   - Update PlantUML to:
     - Show SvelteKit `+server.ts` as the orchestrator.
     - Microservices not publicly exposed.
     - Where validation happens (BFF vs BE), and where enrichment occurs.
   - If events exist later, add AsyncAPI for event contracts (optional now).

4) Data modeling cohesion
   - Align OpenAPI payload schemas with DB schema via shared JSON Schema definitions (source of truth):
     - Generate TS types for BFF and BE from the same schemas.
     - Keep ERD focused on persisted entities; annotate fields that are generated server-side (ids, timestamps, status).
   - Define idempotency keys and uniqueness constraints upfront for create flows.

5) Traceability and “Definition of Ready/Done”
   - Definition of Ready for a story includes: story.md + scenario PlantUML + OpenAPI with examples + acceptance tests plan + auth rules + persistence fields (min).
   - Definition of Done includes: passing contract tests, FE integration against mock/live, observability (logs/metrics), and docs updated.
   - Add a PR checklist referencing these artefacts.

6) Test strategy wired to artefacts
   - Unit: validators and mappers driven by JSON Schema.
   - Contract: OpenAPI verification (microservice), BFF consumer tests.
   - Integration: LocalStack runs with seeded data; run scenario E2E.
   - E2E: Playwright against BFF mock first, then LocalStack.
   - CI gates: fail if OpenAPI changes without version bump and approval.

7) Automation opportunities
   - Generate TS types from OpenAPI/JSON Schema for both BFF and microservice.
   - Generate request/response validators in BFF from schema.
   - Use OpenAPI “Try It” or mock endpoints in dev docs to support PO/QA.

8) Non-functional and ops baked in
   - Add observability acceptance criteria per scenario: include structured logs, metrics, correlation IDs.
   - Security: declare scopes/claims needed for the endpoint in OpenAPI `securitySchemes` and per-operation `security`.
   - Rate limits and quotas (even if placeholder) documented in the spec.

Practical Application: Create Deal (current focus)
- Story: `specs/stories/merchant/manage-deals/create-deal/story.md`
- Scenario (success): `specs/stories/merchant/manage-deals/create-deal/scenarios/success.puml`
- API: `specs/stories/merchant/manage-deals/create-deal/api.yml`
- Next steps:
  1) Finalize request fields and requireds in `api.yml`; sketch responses (201 + error cases).
  2) Ensure sequence diagram shows SvelteKit `+server.ts` as BFF and private microservice invocation.
  3) Implement BFF `POST /api/deals` stub and wire LocalStack config.
  4) Implement microservice handler for Create Deal (internal), returning provisional response.
  5) Add a LocalStack-backed integration test for the success path.

Change Management
- Treat this document as living; update alongside story artefacts.
- When adopting items from the enhancements backlog, promote them into Core steps as they become standard practice.
