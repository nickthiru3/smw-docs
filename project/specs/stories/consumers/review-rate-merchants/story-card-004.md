# Story 004: Review and Rate Merchants

**Status:** Backlog  
**Assignee:** TBD  
**Sprint:** TBD  
**Epic:** Review and rate merchants

---

## Story

**Actor:** Consumer / End User

**When** a consumer has used a merchant's services and wants to share their experience,  
**I want to** write a review and provide a star rating,  
**So I can** help other consumers make informed decisions and provide feedback to merchants.

**Priority:** Medium

**Estimated Complexity:** Medium

---

## Bundled Stories

> This story bundles the following stories from the story map:

- [ ] Story: Write a review for a merchant `[004]`
- [ ] Story: Rate merchant with star rating `[004]`
- [ ] Story: Upload photos with review `[004]`
- [ ] Story: Edit my own review `[004]`
- [ ] Story: Delete my own review `[004]`

---

## Acceptance Criteria

- [ ] TBD - To be expanded when ready to implement

---

## Scenarios (BDD Style)

### Scenario 1: TBD

```gherkin
Given TBD
When TBD
Then TBD
```

---

## Extended Requirements

### Business Rules

TBD

### Implementation Notes & Assumptions

- When a review and rating is provided for a merchant, use DynamoDB streams to update the merchant's average rating and review count
- TBD - Additional notes to be added during implementation planning

### Phase Scope

TBD

### Data Requirements Snapshot

TBD

---

## Technical Notes

**Data Sources:**

TBD

**Components:**

TBD

**API Endpoints:**

TBD

**Dependencies:**

- Depends on: 002 (Reviews displayed on detail view)
- Blocks: TBD

**Performance Targets:**

TBD

---

## Definition of Done

- [ ] TBD - To be expanded when ready to implement

---

## Design References

**Mockups:**

TBD

**Referenced from Other Stories:**

TBD

---

## Related Artefacts

**Story Map:** `docs/project/specs/story-map.md` (Stories bundled in this story: All stories under "Review and rate merchants" epic with ID `[004]`)

**Sequence Diagrams:**

TBD

**API Specifications:**

TBD

**Data Model:**

TBD

**Actions & Queries:**

TBD

**Status Tracking:**

- `docs/project/status-board.md`

---

## Notes & Questions

**Open Questions:**

- Should reviews require moderation before being published?
- What's the minimum/maximum length for review text?
- Should we allow anonymous reviews or require authentication?
- How do we handle review spam and abuse?

**Decisions Log:**

- **2024-11-03:** Created placeholder story card to capture initial thoughts about DynamoDB streams for rating updates

**Implementation Notes:**

- DynamoDB streams will trigger Lambda to update merchant aggregate ratings
- Consider using Step Functions for review moderation workflow
