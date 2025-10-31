# Entities Directory

This directory contains detailed documentation for each entity in the SMW application.

## Purpose

Each entity file is a **comprehensive reference** that includes:

1. **Entity-Key Tables** - Primary key and GSI structures
2. **Access Patterns** - All queries that use this entity
3. **Design Decisions** - Rationale for key choices, trade-offs
4. **Query Examples** - Code snippets for common operations
5. **Evolution History** - Which jobs created/modified this entity
6. **Relationships** - Links to related entities

## File Structure

```
entities/
├── README.md           # This file
├── merchants.md        # Merchant entity (Merchants table + CategoryIndex GSI)
└── reviews.md          # Review entity (Reviews table + MerchantIndex GSI)
```

## When to Create/Update Entity Files

### Creating New Entity Files

**Trigger**: A job introduces a new entity type

**Steps**:
1. Create `entities/<entity-name>.md` with all sections
2. Update `docs/project/specs/entity-key-table.md` (master table)
3. Update `docs/project/specs/erd.puml` (add entity and relationships)

**Example**: When "User Authentication" job is implemented, create `entities/users.md`

### Updating Existing Entity Files

**Trigger**: A job adds new access patterns or GSIs to an existing entity

**Steps**:
1. Update relevant entity file (e.g., `entities/merchants.md`)
2. Add new GSI section if applicable
3. Add new access patterns to the table
4. Document design decisions in the log
5. Update master `entity-key-table.md` if new GSI added

**Example**: When "Merchant Portal" job adds EmailIndex GSI to Merchants table, update `entities/merchants.md`

## Entity File Template

Each entity file follows this structure:

```markdown
# [Entity Name] Entity

**Table Name**: [DynamoDB table name]
**Approach**: Faux-SQL / Single-Table
**Created By**: [Job name]
**Last Updated**: [Date]

---

## Overview
[Brief description of entity purpose]

---

## Main Table Structure
[Primary key design with table]

---

## Global Secondary Indexes
[One section per GSI with design decisions]

---

## Access Patterns
[Table of all queries, with examples]

---

## Attributes
[Reference to TypeScript schemas]

---

## Design Decisions Log
[Chronological log of key decisions]

---

## Referential Integrity
[Foreign key relationships and checks]

---

## Evolution History
[Which jobs created/modified this entity]

---

## Related Entities
[Links to other entity files]

---

## Implementation References
[Links to code: schemas, repositories, services]
```

## Navigation

### Quick Reference
- **Master Entity-Key Table**: `../entity-key-table.md` - All entities at a glance
- **ERD**: `../erd.puml` - Visual entity relationships

### Detailed Entity Files
- **Merchants**: `merchants.md` - Service provider listings
- **Reviews**: `reviews.md` - Customer feedback and ratings

### Future Entities
- **Users** (Phase 2) - User accounts and authentication
- **Favorites** (Phase 2) - Saved merchants per user
- **CheckIns** (Phase 2) - User visit tracking

## Best Practices

### 1. Entity-Centric Organization
- All information about an entity lives in ONE file
- Don't split entity-key tables, GSIs, and access patterns across multiple files
- Access patterns are entity-specific, not job-specific

### 2. Keep Master Table in Sync
- Update `entity-key-table.md` whenever you add/modify an entity
- Master table is for quick reference only
- Detailed info lives in entity files

### 3. Document Rationale
- Explain WHY, not just WHAT
- Include trade-offs considered
- Reference job that drove the decision

### 4. Link Related Entities
- Cross-reference related entities
- Explain relationships (one-to-many, many-to-many)
- Document referential integrity checks

### 5. Evolution Over Perfection
- Start simple, evolve as jobs require
- Document planned enhancements in "Future" sections
- Track changes in Evolution History

## Example Workflow

### Scenario: Implementing "Add to Favorites" Job

**Step 1**: Review existing entities
- Check `entity-key-table.md` to see Merchant entity exists
- Read `entities/merchants.md` to understand MerchantId key

**Step 2**: Create new entity
- Create `entities/favorites.md` for Favorite entity
- Define Favorites table with composite key (UserId + MerchantId)
- Add UserIndex GSI for "get all favorites for user"

**Step 3**: Update master references
- Add Favorites table to `entity-key-table.md`
- Add Favorite entity to `erd.puml`
- Link Merchant ← Favorite relationship

**Step 4**: Document relationships
- In `entities/favorites.md`: Link to Merchant and User entities
- In `entities/merchants.md`: Add note about Favorite relationship
- Document referential integrity checks

## Questions?

See the data modeling guides:
- `docs/guides/data-modeling/guides/3-primary-key-design.md` - Entity-key table guidance
- `docs/guides/data-modeling/guides/faux-sql-dynamodb-modeling.md` - Faux-SQL approach
- `docs/guides/data-modeling/phases/1-experience-and-domain-design.md` - ERD creation
