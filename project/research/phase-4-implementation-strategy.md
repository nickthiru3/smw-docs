# Phase 4 Implementation Strategy - Story 001

**Document Type**: Research & Planning  
**Created**: 2025-11-10  
**Story**: Browse Providers by Waste Category  
**Phase**: Phase 4 - Implementing & Testing

---

## Purpose

This document captures the comprehensive strategy for implementing Story 001 through Phase 4, including the resource-by-resource approach, documentation strategy, and workflow.

---

## Core Principles

### 1. Treat All Guides as Placeholders

**Rationale**: Even detailed guides should be validated against actual implementation.

**Approach**:

- Implement following the opinionated microservice template patterns
- Let implementation inform documentation, not the other way around
- Validate guide content against working code
- Update guides to reflect actual patterns used

### 2. Generic vs App-Specific Documentation

**Generic Documentation** (Implementation Guides):

- ✅ CDK construct usage patterns
- ✅ Resource provisioning approaches
- ✅ Testing strategies and best practices
- ✅ Common patterns (e.g., "how to add a GSI")
- ✅ Reusable code patterns
- ✅ Architecture patterns

**Location**: `{project}/docs/implementation/*.md`  
**Purpose**: Reusable for microservice/frontend templates

**App-Specific Documentation** (Code Comments):

- ✅ Business logic details
- ✅ Specific entity schemas for SMW
- ✅ Actual access patterns for SMW use cases
- ✅ SMW-specific configuration
- ✅ Story-specific decisions and trade-offs

**Location**: Inline code comments, JSDoc  
**Purpose**: Preserve context without cluttering guides

**Example**:

```typescript
// Generic guide: "Transform functions convert between domain and DynamoDB formats"

// App-specific code comment:
/**
 * SMW: Merchant entity uses composite PK for multi-tenancy support.
 * Pattern: TENANT#<tenantId>#MERCHANT#<merchantId>
 * This allows efficient querying of merchants within a tenant boundary.
 */
```

### 3. Two-Log Tracking System

**Log 1: Story Implementation Log**  
**Purpose**: Document the story implementation journey  
**Location**: `docs/implementation/story-001-implementation-log.md`

**Content**:

- Implementation timeline and phases
- Tasks completed
- Decisions made and rationale
- Challenges encountered and solutions
- Code examples that worked well
- Testing notes and coverage
- Performance metrics
- Key learnings

**Log 2: Guide Updates Tracker**  
**Purpose**: Track guide updates needed after each resource  
**Location**: `docs/implementation/guide-updates-tracker.md`

**Content**:

- Resource implementation status
- Analysis of implementation vs guides
- Deviations and new patterns discovered
- Guide updates needed (checklist)
- Feedback and improvement suggestions
- Update completion status

---

## Implementation Strategy

### Bottom-Up Approach (Recommended)

**Order**: Data Layer → Business Logic → API → Monitoring → Testing → Deployment

```
┌─────────────────────────────────────┐
│ 1. DynamoDB Table & Access Layer    │
│    - Entity structure               │
│    - Access patterns                │
│    - Transform functions            │
│    - CRUD operations                │
│    - Unit tests                     │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 2. Lambda Handler                   │
│    - Business logic                 │
│    - Input validation               │
│    - Error handling                 │
│    - Integration tests              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 3. API Gateway                      │
│    - REST API resource              │
│    - Method integration             │
│    - Request/response mapping       │
│    - CORS configuration             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 4. Monitoring & Alarms              │
│    - CloudWatch Logs                │
│    - Metrics                        │
│    - Alarms                         │
│    - SNS notifications              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 5. Testing                          │
│    - Unit tests (alongside code)    │
│    - Integration tests              │
│    - E2E tests                      │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 6. Deployment & Verification        │
│    - Deploy to dev                  │
│    - Verify end-to-end              │
│    - Monitor metrics                │
└─────────────────────────────────────┘
```

**Rationale**:

- ✅ **Data layer first** - Defines entity structure, establishes access patterns, can be unit tested independently
- ✅ **Lambda depends on data layer** - Business logic uses data access layer
- ✅ **API Gateway is thin** - Just proxies to Lambda, minimal logic
- ✅ **Monitoring added after core** - Observability for deployed system
- ✅ **Testing throughout** - Unit tests alongside code, integration after data+lambda, E2E after full stack

### Alternative: Outside-In (API-First)

**Order**: API → Lambda (mocked) → Data Layer → Integration

**Trade-offs**:

- ✅ Validates API contract early
- ✅ Frontend can start with stub responses
- ❌ More mocking/stubbing needed
- ❌ Integration comes later
- ❌ May need rework if data model changes

**Recommendation**: Use bottom-up for Story 001 to establish solid foundation.

---

## Workflow Per Resource

### Step-by-Step Process

```
┌─────────────────────────────────────┐
│ 1. IMPLEMENT RESOURCE               │
│    - Follow template patterns       │
│    - Document decisions in code     │
│    - Write tests alongside          │
│    - Commit working code            │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 2. DOCUMENT IN STORY LOG            │
│    - What was implemented           │
│    - Decisions made and why         │
│    - Challenges faced               │
│    - Solutions implemented          │
│    - Code examples                  │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 3. ANALYZE VS GUIDES                │
│    - Compare with existing guide    │
│    - Note deviations                │
│    - Identify new patterns          │
│    - Separate generic vs specific   │
│    - Document in tracker            │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 4. DISCUSS & PROVIDE FEEDBACK       │
│    - What worked well               │
│    - Improvement suggestions        │
│    - Alternative approaches         │
│    - Guide update proposals         │
│    - Questions/concerns             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 5. UPDATE GUIDES                    │
│    - Make generic updates           │
│    - Add code examples              │
│    - Update best practices          │
│    - Keep SMW-specific in code      │
│    - Review and commit              │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│ 6. MOVE TO NEXT RESOURCE            │
│    - Mark current complete          │
│    - Update tracker                 │
│    - Begin next resource            │
└─────────────────────────────────────┘
```

### Time Estimates (Story 001 Backend)

| Resource                | Estimated Sessions | Notes                                            |
| ----------------------- | ------------------ | ------------------------------------------------ |
| DynamoDB + Access Layer | 2-3 sessions       | Entity, access patterns, transforms, CRUD, tests |
| Lambda Handler          | 1-2 sessions       | Handler, validation, business logic, tests       |
| API Gateway             | 1 session          | REST API, method integration, CORS               |
| Monitoring              | 1 session          | Logs, metrics, alarms, SNS                       |
| E2E Testing             | 1 session          | Deploy to dev, test full flow                    |
| Guide Updates           | 1 session          | Review and finalize all updates                  |
| **Total**               | **7-10 sessions**  | Includes implementation + documentation          |

---

## Artifacts to Reference

### Phase 1-3 Artifacts

All artifacts inform implementation decisions:

| Artifact              | Purpose                                   | Location                                                                                    |
| --------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------- |
| **Story Card**        | Acceptance criteria, scope, constraints   | `docs/project/specs/stories/consumers/browse-providers-by-waste-category/story-card-001.md` |
| **Sequence Diagram**  | Flow, components, interactions            | `docs/project/specs/stories/.../sequence-diagram.puml`                                      |
| **Actions & Queries** | CQS separation, operations                | `docs/project/specs/stories/.../actions-and-queries.md`                                     |
| **Data Model**        | Entities, relationships, access patterns  | `docs/project/specs/stories/.../data-model.md`                                              |
| **OpenAPI Spec**      | API contract, request/response schemas    | `docs/project/specs/api/openapi.yaml`                                                       |
| **UI Mockups**        | Frontend screens (for web implementation) | `docs/project/specs/stories/.../mockups/`                                                   |

### How to Use Artifacts

**Story Card**:

- ✅ Defines what to build
- ✅ Acceptance criteria = test cases
- ✅ Constraints guide implementation

**Sequence Diagram**:

- ✅ Shows component interactions
- ✅ Identifies integration points
- ✅ Reveals error handling needs

**Actions & Queries**:

- ✅ Separates commands from queries
- ✅ Defines operation signatures
- ✅ Guides Lambda handler structure

**Data Model**:

- ✅ Defines entity structure
- ✅ Specifies access patterns
- ✅ Informs DynamoDB schema

**OpenAPI Spec**:

- ✅ API contract (single source of truth)
- ✅ Request/response validation
- ✅ Guides API Gateway configuration

---

## Backend Implementation Sequence

### Story 001: Browse Providers by Waste Category

#### 1. DynamoDB Table & Access Layer

**Tasks**:

- [ ] Review data model artifact
- [ ] Define Merchant entity TypeScript interface
- [ ] Create DynamoDB item interface
- [ ] Implement transform functions (toItem/fromItem)
- [ ] Implement CRUD operations
- [ ] Implement query by category (access pattern)
- [ ] Write unit tests for transforms and operations
- [ ] Document in story log
- [ ] Analyze vs data-access.md guide
- [ ] Update guide

**Guides to Update**:

- `data-access.md`
- `using-constructs.md` (DynamoDB construct)
- `testing.md` (unit test patterns)

**Key Decisions**:

- Entity schema design
- Access pattern implementation
- Transform function patterns
- Error handling approach

#### 2. Lambda Handler

**Tasks**:

- [ ] Review actions & queries artifact
- [ ] Create Lambda handler structure
- [ ] Implement input validation
- [ ] Implement business logic (query merchants)
- [ ] Implement response formatting
- [ ] Add error handling
- [ ] Write handler unit tests
- [ ] Write integration tests with DynamoDB Local
- [ ] Document in story log
- [ ] Analyze vs adding-endpoints.md guide
- [ ] Update guide

**Guides to Update**:

- `adding-endpoints.md`
- `using-constructs.md` (Lambda construct)
- `testing.md` (integration test patterns)

**Key Decisions**:

- Validation strategy
- Error response format
- Logging approach

#### 3. API Gateway

**Tasks**:

- [ ] Review OpenAPI spec
- [ ] Create REST API resource
- [ ] Configure method integration
- [ ] Set up request/response mapping
- [ ] Configure CORS
- [ ] Add request validation
- [ ] Test API endpoint
- [ ] Document in story log
- [ ] Analyze vs adding-endpoints.md guide
- [ ] Update guide

**Guides to Update**:

- `adding-endpoints.md` (API Gateway section)
- `using-constructs.md` (API Gateway construct)

**Key Decisions**:

- API Gateway configuration
- CORS policy
- Request validation rules

#### 4. Monitoring & Alarms

**Tasks**:

- [ ] Configure CloudWatch Logs
- [ ] Add structured logging
- [ ] Define CloudWatch Metrics
- [ ] Create CloudWatch Alarms
- [ ] Set up SNS notifications
- [ ] Test alarm triggering
- [ ] Document in story log
- [ ] Analyze vs monitoring.md guide
- [ ] Update guide

**Guides to Update**:

- `monitoring.md`
- `using-constructs.md` (monitoring constructs)

**Key Decisions**:

- Logging format
- Metrics to track
- Alarm thresholds

#### 5. E2E Testing

**Tasks**:

- [ ] Deploy to dev environment
- [ ] Test full flow end-to-end
- [ ] Verify monitoring and alarms
- [ ] Load test (if applicable)
- [ ] Document in story log
- [ ] Analyze vs testing.md guide
- [ ] Update guide

**Guides to Update**:

- `testing.md` (E2E patterns)
- `deployment.md`

#### 6. Guide Finalization

**Tasks**:

- [ ] Review all guide updates
- [ ] Ensure generic patterns only
- [ ] Add code examples
- [ ] Update best practices
- [ ] Commit all changes
- [ ] Mark Story 001 backend complete

---

## Frontend Implementation Sequence

### Story 001: Browse Providers by Waste Category

**Note**: Start after backend is deployed to dev.

#### 1. BFF API Routes

**Tasks**:

- [ ] Create `/api/merchants` route
- [ ] Implement request validation
- [ ] Call backend microservice
- [ ] Transform response for frontend
- [ ] Add error handling
- [ ] Write API route tests
- [ ] Document in story log
- [ ] Analyze vs bff-api-routes.md guide
- [ ] Update guide

#### 2. UI Pages & Components

**Tasks**:

- [ ] Create merchant listing page
- [ ] Implement category filter component
- [ ] Create merchant card component
- [ ] Add loading and error states
- [ ] Implement responsive layout
- [ ] Add accessibility features
- [ ] Document in story log
- [ ] Analyze vs adding-routes.md guide
- [ ] Update guide

#### 3. State Management

**Tasks**:

- [ ] Create merchant store
- [ ] Implement category filter state
- [ ] Add loading state management
- [ ] Implement error state handling
- [ ] Document in story log
- [ ] Analyze vs state-management.md guide
- [ ] Update guide

#### 4. Testing & Deployment

**Tasks**:

- [ ] Write component tests
- [ ] Write API route tests
- [ ] Write E2E tests
- [ ] Deploy to dev
- [ ] Verify functionality
- [ ] Document in story log
- [ ] Update testing.md and deployment.md guides

---

## Success Criteria

### Implementation Complete When:

- ✅ All resources implemented and tested
- ✅ Story acceptance criteria met
- ✅ E2E tests passing
- ✅ Deployed to dev environment
- ✅ Monitoring and alarms working

### Documentation Complete When:

- ✅ Story implementation log filled out
- ✅ All guide updates analyzed
- ✅ Generic patterns extracted
- ✅ Guides updated with real examples
- ✅ Code comments added for SMW-specific details
- ✅ Patterns ready for template

---

## Key Questions to Answer

### During Implementation:

1. Does the template pattern work for this use case?
2. Are there better approaches?
3. What challenges did we encounter?
4. What patterns emerged?
5. What should be documented?

### During Analysis:

1. Does the guide match implementation?
2. What's missing from the guide?
3. What's generic vs app-specific?
4. What examples should be added?
5. What best practices emerged?

### During Updates:

1. Is this pattern reusable?
2. Should this go in the guide or code?
3. Is the example clear and complete?
4. Does this apply to the template?
5. What else needs updating?

---

## Starting Point

### Begin with: `svc-merchants/bin/app.ts`

**Why**:

- ✅ Entry point for CDK application
- ✅ Shows stack orchestration
- ✅ Reveals configuration management
- ✅ Provides context for entire project
- ✅ Helps understand flow from top to bottom

**What to Understand**:

1. How CDK app is initialized
2. How configuration is loaded
3. How stacks are instantiated
4. How environment is determined
5. How resources flow through layers

**Next Steps After app.ts**:

1. Review configuration (`config/default.ts`)
2. Examine ServiceStack (`lib/service-stack.ts`)
3. Study constructs used
4. Understand resource dependencies
5. Identify where to add Merchant resources

---

## References

- **Design Methodology**: `docs/guides/design-and-development/design-and-development-methodology-v3.md`
- **Story Card**: `docs/project/specs/stories/consumers/browse-providers-by-waste-category/story-card-001.md`
- **Backend Implementation Guides**: `svc-merchants/docs/implementation/`
- **Frontend Implementation Guides**: `web/docs/implementation/`
- **Story Implementation Logs**: `{project}/docs/implementation/story-001-implementation-log.md`
- **Guide Update Trackers**: `{project}/docs/implementation/guide-updates-tracker.md`
