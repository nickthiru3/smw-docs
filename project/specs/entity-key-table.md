# Entity-Key Table (Master Reference)

**Last Updated**: 2024-10-28  
**Approach**: Faux-SQL (multiple tables, descriptive key names)  
**Purpose**: Quick reference for all entities, tables, and GSIs across the application

---

## Main Tables

### Merchants Table

| Entity   | Partition Key | Sort Key | Notes                     |
| -------- | ------------- | -------- | ------------------------- |
| Merchant | MerchantId    | -        | Simple key, direct lookup |

**Details**: See `entities/merchants.md`

---

### Reviews Table

| Entity | Partition Key | Sort Key | Notes                     |
| ------ | ------------- | -------- | ------------------------- |
| Review | ReviewId      | -        | Simple key, direct lookup |

**Details**: See `entities/reviews.md`

---

## Global Secondary Indexes

### CategoryIndex (on Merchants Table)

| Entity   | GSI PK   | GSI SK | Notes                       |
| -------- | -------- | ------ | --------------------------- |
| Merchant | Category | -      | Query merchants by category |

**Details**: See `entities/merchants.md#categoryindex`

---

### MerchantIndex (on Reviews Table)

| Entity | GSI PK     | GSI SK     | Notes                                  |
| ------ | ---------- | ---------- | -------------------------------------- |
| Review | MerchantId | ReviewDate | Get reviews for merchant, newest first |

**Details**: See `entities/reviews.md#merchantindex`

---

## Evolution Log

| Date       | Job                | Change                                          |
| ---------- | ------------------ | ----------------------------------------------- |
| 2024-10-28 | Search by Category | Created Merchants table, Reviews table          |
| 2024-10-28 | Search by Category | Added CategoryIndex GSI, MerchantIndex GSI      |

---

## Quick Navigation

- **Merchants**: [entities/merchants.md](entities/merchants.md)
- **Reviews**: [entities/reviews.md](entities/reviews.md)
- **ERD**: [erd.puml](erd.puml)

---

## Notes

- Each entity file (`entities/*.md`) contains:
  - Main table entity-key structure
  - GSI definitions
  - Access patterns
  - Design decisions
  - Query examples
  - Evolution history

- This master table provides a quick reference only
- For detailed information, rationale, and examples, see individual entity files
