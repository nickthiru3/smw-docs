# Story [ID]: [Story Title]

**Status:** [Backlog / Ready for Dev / In Progress / In Review / Done]  
**Assignee:** [Name or TBD]  
**Sprint:** [Sprint number or TBD]  
**Epic:** [Epic name from story map]

---

## Story

**Actor:** [Actor name]

**When** [circumstance],  
**I want to** [motivation],  
**So I can** [goal].

**Priority:** [High / Medium / Low]

**Estimated Complexity:** [Small / Medium / Large]

---

## Bundled Stories

> This story bundles the following stories from the story map:

- [ ] Story: [Story description from story map] `[Story Number]`
- [ ] Story: [Story description from story map] `[Story Number]`
- [ ] Story: [Story description from story map] `[Story Number]`

> **Note:** If this story represents a single story, remove this section.

---

## Acceptance Criteria

- [ ] [Criterion 1 - testable, specific outcome]
- [ ] [Criterion 2 - testable, specific outcome]
- [ ] [Criterion 3 - testable, specific outcome]
- [ ] [Criterion 4 - handles edge cases gracefully]
- [ ] [Criterion 5 - responsive/accessible/performant]

---

## Scenarios

### Scenario 1: [Happy path scenario name]

```gherkin
Given [initial context/state]
When [action/event occurs]
Then [expected outcome]
And [additional expected outcome]
And [additional expected outcome]
```

### Scenario 2: [Edge case or error scenario name]

```gherkin
Given [initial context/state]
When [action/event occurs]
Then [expected outcome]
And [system behavior]
```

### Scenario 3: [Alternative flow scenario name]

```gherkin
Given [initial context/state]
When [action/event occurs]
Then [expected outcome]
```

> **Note:** Each scenario should be specific and testable. These will guide both implementation and automated testing.

---

## Extended Requirements

### Business Rules

[Detailed business logic, constraints, validation rules, and domain-specific requirements]

### Implementation Notes & Assumptions

- [Technical assumptions for this story]
- [MVP scope decisions]
- [Dependencies on other services/stories]
- [Backend/Frontend ownership notes]
- [Data source and integration details]

### Phase Scope

- **Phase 1a (MVP):** [What gets delivered in initial release]
- **Phase 1b:** [Enhancements building on MVP]
- **Phase 2:** [Future extensions and advanced features]

### Data Requirements Snapshot

[Key data fields, entities, and structures needed for this story. Reference full data model docs for details.]

**Core fields needed:**

- [Entity 1]: field1, field2, field3
- [Entity 2]: field1, field2

**Deferred to later stories:**

- [Fields/features not needed for this story]

See `docs/project/research/[relevant-research-doc].md` for full attribute catalogue.

---

## Technical Notes

**Data Sources:**

- [Table/Entity name and access pattern]
- [API endpoints involved]

**Components:**

- Frontend: `[ComponentName.svelte]`
- BFF Route: `[+server.ts route path]`
- Backend Service: `[service-name]`

**API Endpoints:**

- `GET /api/[endpoint]` - [purpose]
- `POST /api/[endpoint]` - [purpose]

**Dependencies:**

- Depends on: [story number or service]
- Blocks: [story number that needs this completed]

**Performance Targets:**

- [Specific performance requirements, e.g., <200ms response time]

---

## Definition of Done

- [ ] Code written and unit tested
- [ ] BDD scenarios implemented as automated tests
- [ ] Responsive design verified (mobile/tablet/desktop)
- [ ] Accessibility checked (screen reader, keyboard navigation, ARIA labels)
- [ ] Code reviewed and approved
- [ ] Integration tests passing
- [ ] Documentation updated
- [ ] Deployed to staging environment
- [ ] Demo to Product Owner completed
- [ ] Acceptance criteria validated

---

## Design References

[Link to Figma mockups, wireframes, or design assets]

**Mockups:**

- [mockups/01-screen-name.png](./mockups/01-screen-name.png)
- [mockups/02-screen-name.png](./mockups/02-screen-name.png)

---

## Related Artefacts

**Story Map:** `docs/project/specs/story-map.md` (Stories bundled in this story: [list story numbers])

**Data Model:**

- ERD: `docs/project/specs/erd.puml`
- Entity files: `docs/project/specs/entities/[entity-name].md`

**Sequence Diagrams:**

- [sequence-diagrams/01-flow-name.puml](./sequence-diagrams/01-flow-name.puml)

**Actions & Queries:**

- [actions-queries.md](./actions-queries.md)

**API Specifications:**

- [api.yml](./api.yml) or reference to service OpenAPI spec

**Status Tracking:**

- `docs/project/status-board.md`

---

## Notes & Questions

[Space for ongoing notes, decisions, blockers, and questions that arise during implementation]

**Open Questions:**

- [Question 1]
- [Question 2]

**Decisions Log:**

- [Date]: [Decision made and rationale]
