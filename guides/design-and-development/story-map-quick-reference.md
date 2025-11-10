# Story Map Quick Reference

A cheat sheet for using the story map approach in product discovery.

---

## Story Map Structure

```markdown
- 👤 User: [Actor Name]
  - 🎯 Goal: [High-level user goal]
    - 📝 Epic: [Major capability area]
      - 📄 Story: [Implementable story] `[Story Number]`
      - 📄 Story: [Implementable story] `[Story Number]`
```

### Hierarchy Levels

| Level     | Icon | Purpose                    | Example                                      |
| --------- | ---- | -------------------------- | -------------------------------------------- |
| **User**  | 👤   | Who uses the system        | Consumer, Merchant, Admin                    |
| **Goal**  | 🎯   | What they want to achieve  | Find providers, Get listed, Maintain quality |
| **Epic**  | 📝   | Major capability area      | Browse by category, Create profile           |
| **Story** | 📄   | Implementable unit of work | Display search filters, Upload photos        |

---

## Story ID Assignment

### Format

- Simple sequential: `[001]`, `[002]`, `[003]`
- No prefixes needed (context is clear from location)

### Bundling Rules

**Bundle stories with same ID when they:**

- ✅ Work on same page/component
- ✅ Pull from same data model
- ✅ Have similar technical complexity
- ✅ Are naturally tested together

**Keep separate IDs when:**

- ❌ Different technical complexity
- ❌ Can be delivered incrementally
- ❌ Have complex acceptance criteria
- ❌ Are cross-cutting concerns

### Example: Bundled Stories

```markdown
- 📝 Epic: View detailed business information
  - 📄 Story: Display business name and description `[001]`
  - 📄 Story: Show operating hours `[001]`
  - 📄 Story: Display contact details `[001]`
  - 📄 Story: Show physical address `[001]`
  - 📄 Story: Generate SEO-friendly URL `[002]` ← Separate (technical complexity)
```

Story 001 bundles the first 4 stories.  
Story 002 handles SEO URLs separately.

---

## Workflow

### 1. Discovery (Continuous)

```
Identify Actors → Create Story Map → Assign Story IDs → Create Story Cards → Prioritize
```

**Story Map Benefits:**

- Quick idea capture mid-flow
- Big picture view of all user needs

### 2. From Story Map to Story Card

**When:** Ready to implement a story

**Steps:**

1. Find all stories with same Story ID in story map
2. Create story card using template: `story-card-template.md`
3. List bundled stories in "Bundled Stories" section
4. Write BDD scenarios for key behaviors
5. Fill in Extended Requirements and Technical Notes

**File naming:** `story-card-[ID].md` where ID matches story map

---

## Story Card Essentials

### Must-Have Sections

- **Story (Job Story Format)** - When/I want/So I can
- **Bundled Stories** - Which stories from map are included
- **Acceptance Criteria** - Testable outcomes
- **Scenarios (BDD)** - Given-When-Then examples
- **Extended Requirements** - Business rules, assumptions, phase scope
- **Related Artefacts** - Links to designs, APIs, data models

### Optional Sections

- Status/Assignee/Sprint (if using formal tracking)
- Technical Notes (helpful for implementation)
- Definition of Done (comprehensive checklist)
- Design References (if mockups exist)
- Notes & Questions (decision log)

---

## BDD Scenarios

### Format

```gherkin
Given [initial context/state]
And [additional context]
When [action/event occurs]
Then [expected outcome]
And [additional outcome]
```

### Purpose

- **Stories** define scope (what capability we need)
- **Scenarios** define tests (how we know it works)

### Example

**Story:** Display business contact details

**Scenario:**

```gherkin
Given I am viewing a merchant listing page
And the business has provided a phone number
When I tap the phone number on mobile
Then my phone's dialer should open
And the number should be pre-filled
```

### Tips

- Write 3-5 scenarios per story
- Cover happy path, edge cases, error states
- Make them specific and testable
- Each becomes an automated test

---

## Traceability

### Story IDs Connect Everything

```
Story Map `[001]`
    ↓
Story Card 001
    ↓
Git Commit: "feat: implement story 001 - display merchant details"
    ↓
PR #123: "Story 001: Display merchant business details"
    ↓
Automated Tests: "Scenario 1: View complete business information"
```

### In Practice

**Story Map:**

```markdown
- 📄 Story: Display business name `[001]`
```

**Story Card:**

```markdown
# Story Card 001: Display Merchant Business Details

## Bundled Stories

- [x] Story: Display business name `[001]`
```

**Git Commit:**

```bash
git commit -m "feat(story-001): add merchant detail view component"
```

**PR Description:**

```markdown
## Story 001: Display Merchant Business Details

Implements stories from story map with Story ID [001]

- Display business name and description
- Show operating hours
- Display contact details
```

---

## Quick Decision Tree

### Should I bundle these stories?

```
Are they on the same page? ──No──→ Separate IDs
    │
   Yes
    │
    ↓
Same data source? ──No──→ Separate IDs
    │
   Yes
    │
    ↓
Similar complexity? ──No──→ Separate IDs
    │
   Yes
    │
    ↓
Bundle with same ID ✓
```

### Should I create a BDD scenario?

```
Is behavior obvious? ──Yes──→ Maybe skip (but scenarios help!)
    │
    No
    │
    ↓
Are there edge cases? ──Yes──→ Definitely write scenarios
    │
    No
    │
    ↓
Will this be tested? ──Yes──→ Write scenarios (they become tests)
    │
    No
    │
    ↓
Write at least 1 happy path scenario ✓
```

---

## Common Patterns

### Pattern 1: CRUD Operations

**Bundle:** All basic CRUD for same entity

```markdown
- 📄 Story: Create merchant profile `[010]`
- 📄 Story: Edit merchant profile `[010]`
- 📄 Story: Delete merchant profile `[010]`
```

### Pattern 2: Display Multiple Fields

**Bundle:** All fields on same view

```markdown
- 📄 Story: Display name `[015]`
- 📄 Story: Display description `[015]`
- 📄 Story: Display contact info `[015]`
```

### Pattern 3: Complex Feature

**Separate:** Different technical complexity

```markdown
- 📄 Story: Display static map `[020]`
- 📄 Story: Add interactive map controls `[021]` ← More complex
- 📄 Story: Enable route planning `[022]` ← Even more complex
```

### Pattern 4: Progressive Enhancement

**Separate:** Can be delivered incrementally

```markdown
- 📄 Story: Show basic search results `[025]`
- 📄 Story: Add sorting options `[026]` ← Enhancement
- 📄 Story: Add advanced filters `[027]` ← Further enhancement
```

---

## Tips for Solo Developers

### Use Story Map As...

- **Idea capture tool** - Jot down thoughts without breaking flow
- **Planning tool** - See what needs to be built
- **Prioritization tool** - Order by value/dependencies

### Keep It Simple

- Don't overthink bundling - adjust as you learn
- Start with fewer scenarios, add more as needed
- Use template sections that add value, skip others

### Maintain Traceability

- Always use Story IDs in commits: `feat(story-001): ...`
- Reference Story IDs in PRs and discussions
- Update story map as you discover new stories

---

## Files and Locations

| Artifact            | Location                                                                 | Purpose                    |
| ------------------- | ------------------------------------------------------------------------ | -------------------------- | --- |
| Story Map           | `docs/project/specs/story-map.md`                                        | Master list of all stories |
| Story Card Template | `docs/project/specs/stories/story-card-template.md`                      | Blank template             |     |
| Story Cards         | `docs/project/specs/stories/[user]/[epic]/story-card-[ID].md`            | Actual story cards         |
| Methodology         | `docs/guides/design-principles/design-and-development-methodology-v2.md` | Full process               |

---

## Quick Start

### For a New Feature

1. **Add to story map** with Story ID

   ```markdown
   - 📄 Story: [Your story] `[099]`
   ```

2. **Create story card** from template

   ```bash
   cp docs/project/specs/stories/story-card-template.md \
      docs/project/specs/stories/[user]/[epic]/story-card-099.md
   ```

3. **Fill in sections:**

   - Story (Job Story Format)
   - Bundled Stories (list stories with `[099]`)
   - Acceptance Criteria
   - 3-5 BDD Scenarios
   - Extended Requirements

4. **Implement** with Story ID in commits

   ```bash
   git commit -m "feat(story-099): implement [feature]"
   ```

5. **Test** using BDD scenarios as test specs

6. **Check off** Definition of Done items

---

## Resources

- **Full Methodology:** `design-and-development-methodology-v2.md`

---

## Questions?

Add them to the "Notes & Questions" section of your story card or create a discussion in the methodology guide.
