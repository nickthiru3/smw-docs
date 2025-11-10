# Entity-Key Table (Master Reference)

**Last Updated**: 2024-11-06  
**Approach**: Faux-SQL (hybrid naming: descriptive primary keys, generic GSI attributes)  
**Purpose**: Quick reference for all entities, tables, and GSIs across the application

---

Base entities directory: `docs/project/specs/entities/`

## Main Tables

### Merchants Table

| Entity   | Partition Key | Sort Key | Notes                     |
| -------- | ------------- | -------- | ------------------------- |
| Merchant | MerchantId    | -        | Simple key, direct lookup |

**Details**: See `merchants.md`

---

### Reviews Table

| Entity | PK       | SK  | Notes                     |
| ------ | -------- | --- | ------------------------- |
| Review | ReviewId | -   | Simple key, direct lookup |

**Details**: See `reviews.md`

---

## Global Secondary Indexes

### GSI1 (on Merchants Table)

| Entity   | GSI1PK          | GSI1SK | Notes                             |
| -------- | --------------- | ------ | --------------------------------- |
| Merchant | PrimaryCategory | -      | Query merchants by category value |

**Details**: See `merchants.md#gsi1-category-queries`

---

### GSI1 (on Reviews Table)

| Entity | GSI1PK     | GSI1SK     | Notes                     |
| ------ | ---------- | ---------- | ------------------------- |
| Review | MerchantId | ReviewDate | Query reviews by merchant |

**Details**: See `reviews.md#gsi1`

---

## Evolution Log

| Date       | Story/Job                               | Change                                                         |
| ---------- | --------------------------------------- | -------------------------------------------------------------- |
| 2024-10-28 | Search by Category                      | Created Merchants table, Reviews table                         |
| 2024-10-28 | Search by Category                      | Added GSI1 to both tables (descriptive naming initially)       |
| 2024-11-06 | Story 001: Browse Providers by Category | Updated to generic GSI naming (GSI1PK, GSI1SK) for flexibility |

---

## Quick Navigation

- **Merchants**: [merchants.md](merchants.md)
- **Reviews**: [reviews.md](reviews.md)
- **ERD**: [erd.puml](erd.puml)

---

## Notes

- Each entity file (`entities/*.md`) contains:

  - Access patterns
  - Main table entity-key structure
  - GSI definitions
  - Design decisions
  - Query examples
  - Evolution history

- This master table provides a quick reference only
- For detailed information, rationale, and examples, see individual entity files
