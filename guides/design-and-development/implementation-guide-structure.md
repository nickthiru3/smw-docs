# Implementation Guide Structure

**Overview**: Documentation structure for project-specific implementation guides

---

## Purpose

This document describes the structure and organization of implementation guides within each project (frontend and backend).

**Key Principles**:

- **Project-specific** - Guides live within each project directory
- **Practical** - Step-by-step instructions with code examples
- **Story-driven** - Document as we implement real stories
- **Template-ready** - Backend guides can be copied to microservice template

---

## Backend Implementation Guides

**Location**: `svc-{service-name}/docs/implementation/`

### Structure

```
svc-merchants/docs/
├── implementation/
│   ├── README.md                           # Overview & quick start
│   ├── data-access.md                      # DynamoDB data access layer
│   ├── adding-endpoints.md                 # Lambda handlers & API Gateway
│   ├── using-constructs.md                 # CDK constructs usage
│   ├── authentication.md                   # Cognito auth patterns
│   ├── monitoring.md                       # CloudWatch & SNS
│   ├── testing.md                          # Testing strategies
│   ├── deployment.md                       # CDK deployment
│   └── story-001-implementation-log.md     # Story implementation log
└── architecture/
    └── overview.md                         # Stack architecture
```

### Guide Contents

#### README.md

- Overview of the microservice
- Quick start guide
- Links to all implementation guides
- Project structure explanation
- Development workflow
- Best practices summary

#### data-access.md

- TypeScript interfaces (domain & DynamoDB)
- Transform functions (toItem/fromItem)
- CRUD operations
- Query operations (access patterns)
- Unit testing examples
- Best practices

#### adding-endpoints.md

- Lambda handler structure
- Input validation
- Response utilities
- API Gateway wiring
- Handler testing
- Integration testing

#### using-constructs.md

- Available CDK constructs
- Common tasks (adding endpoints, GSIs, alarms)
- Configuration management
- SSM parameter bindings

#### authentication.md

- Cognito User Pool configuration
- JWT token validation
- API Gateway authorizer
- User groups and permissions

#### monitoring.md

- CloudWatch Logs configuration
- Structured logging patterns
- CloudWatch Metrics
- CloudWatch Alarms
- SNS notifications

#### testing.md

- Test pyramid
- Unit testing patterns
- Integration testing with DynamoDB Local
- CDK testing
- Running tests

#### deployment.md

- CDK deployment commands
- Environment configuration
- Stack outputs
- Post-deployment verification
- Rollback procedures

#### story-001-implementation-log.md

- Implementation timeline
- Tasks completed per phase
- Decisions made
- Challenges encountered
- Patterns established
- Code examples
- Testing notes
- Performance metrics

---

## Frontend Implementation Guides

**Location**: `web/docs/implementation/`

### Structure

```
web/docs/
├── implementation/
│   ├── README.md                           # Overview & quick start
│   ├── adding-routes.md                    # SvelteKit routing
│   ├── bff-api-routes.md                   # BFF layer implementation
│   ├── state-management.md                 # Stores & runes
│   ├── authentication.md                   # Amplify integration
│   ├── styling.md                          # TailwindCSS patterns
│   ├── testing.md                          # Vitest & Playwright
│   ├── deployment.md                       # Amplify hosting
│   └── story-001-implementation-log.md     # Story implementation log
└── architecture/
    └── overview.md                         # Project architecture
```

### Guide Contents

#### README.md

- Overview of the frontend project
- Quick start guide
- Links to all implementation guides
- Project structure explanation
- Development workflow
- Best practices summary

#### adding-routes.md

- SvelteKit routing conventions
- Page components
- Page load functions
- Layouts
- Dynamic routes
- Error pages

#### bff-api-routes.md

- BFF layer responsibilities
- Creating API routes (+server.ts)
- Request handlers
- Backend orchestration
- Response transformation
- Error handling
- Testing API routes

#### state-management.md

- Svelte stores (writable, readable, derived)
- Svelte 5 runes ($state, $derived, $effect)
- When to use stores vs runes
- Global vs component state
- Form state management

#### authentication.md

- AWS Amplify integration
- Cognito authentication flow
- Session management
- Protected routes
- Login/logout flows
- Token refresh

#### styling.md

- TailwindCSS configuration
- Utility-first CSS
- Component-scoped styles
- Responsive design
- Dark mode
- Accessibility

#### testing.md

- Component testing (Vitest)
- API route testing
- E2E testing (Playwright)
- Mocking strategies
- Testing authentication

#### deployment.md

- AWS Amplify Hosting setup
- Build configuration
- Environment variables
- Custom domains
- CI/CD integration

#### story-001-implementation-log.md

- Implementation timeline
- UI components created
- BFF API routes implemented
- State management patterns
- User interactions
- Testing notes
- Performance metrics
- Accessibility compliance

---

## Central Guides vs Project Guides

### Central Guides (`docs/guides/`)

**Purpose**: Conceptual, cross-cutting concerns

**Topics**:

- Design methodology
- Architecture patterns
- Data modeling approaches
- Testing strategy (high-level)
- Authentication patterns (conceptual)
- Deployment strategy

**Audience**: All team members, cross-functional

### Project Guides (`{project}/docs/implementation/`)

**Purpose**: Practical, how-to, project-specific

**Topics**:

- Step-by-step implementation
- Code examples from the actual project
- Testing examples with actual test files
- Project-specific patterns
- Story implementation logs

**Audience**: Developers working on that specific project

### Cross-Reference Strategy

- **Central guides** provide theory and patterns
- **Project guides** provide practical implementation
- **Project guides link to central guides** for conceptual background
- **Central guides reference project guides** for examples

---

## Story Implementation Logs

### Purpose

Document the implementation of each story as we build it:

- **Capture decisions** - Why we chose certain approaches
- **Record challenges** - Problems encountered and solutions
- **Establish patterns** - Reusable patterns for future stories
- **Track metrics** - Performance, testing, accessibility
- **Share learnings** - What worked well, what to improve

### Structure

Each story log includes:

1. **Implementation Timeline** - Phases and tasks completed
2. **Decisions Made** - Technical decisions and rationale
3. **Challenges** - Problems and solutions
4. **Patterns Established** - Reusable patterns
5. **Code Examples** - Working code snippets
6. **Testing Notes** - Coverage, test cases, scenarios
7. **Performance Metrics** - Latency, memory, load times
8. **Key Learnings** - What worked, what to improve
9. **Next Steps** - Follow-up actions

### Benefits

- **Documentation as we go** - No need to remember later
- **Knowledge sharing** - Team learns from each story
- **Template improvement** - Patterns copied to template
- **Onboarding** - New developers see real examples
- **Continuous improvement** - Learn and adapt

---

## Maintenance

### Updating Guides

- **During implementation** - Update guides with real examples
- **After story completion** - Refine based on learnings
- **Quarterly review** - Ensure guides stay current
- **Template sync** - Copy backend patterns to template

### Version Control

- All guides are version-controlled with code
- Changes reviewed in pull requests
- Guides evolve with the codebase

---

## Next Steps

1. ✅ **Backend guides created** - Ready for Story 001 implementation
2. ✅ **Frontend guides created** - Ready for Story 001 implementation
3. 🔄 **Implement Story 001** - Document as we go
4. 📝 **Update guides** - Add real examples from Story 001
5. 📋 **Copy to template** - Update microservice template with learnings
6. 🔁 **Repeat** - Continue for Story 002, 003, etc.

---

## References

- **Backend Implementation**: `svc-merchants/docs/implementation/README.md`
- **Frontend Implementation**: `web/docs/implementation/README.md`
- **Central Guides**: `docs/guides/README.md`
- **Design Methodology**: `docs/guides/design-and-development/design-and-development-methodology-v3.md`
