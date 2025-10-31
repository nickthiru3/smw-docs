# Review Entity

**Table Name**: Reviews  
**Approach**: Faux-SQL (separate table for unbounded relationship)  
**Created By**: Search by Category job  
**Last Updated**: 2024-10-28

---

## Overview

The Review entity represents customer feedback and ratings for merchants. Reviews are stored in a separate table due to the unbounded nature of the relationship (merchants can have hundreds of reviews).

**Core Purpose**: Provide trust signals and detailed feedback to help consumers make informed decisions.

---

## Main Table Structure

### Primary Key Design

| Entity | Partition Key | Sort Key | Notes                                 |
| ------ | ------------- | -------- | ------------------------------------- |
| Review | ReviewId      | -        | Simple key, UUID, direct lookup by ID |

**Key Design Decisions**:

- **Simple key (partition key only)**: Each review is independent
- **UUID for ReviewId**: Globally unique identifier
- **Separate table**: Unbounded relationship with Merchant (can't denormalize)

**Why Separate Table?**
- **Unbounded relationship**: Merchants can have hundreds of reviews
- **Independent access**: Reviews can be queried, updated, deleted independently
- **Faux-SQL principle**: One table per entity type
- **Scalability**: Doesn't bloat Merchant items

---

## Global Secondary Indexes

### MerchantIndex

**Purpose**: Query all reviews for a specific merchant, sorted by date (newest first)

| Entity | GSI PK     | GSI SK     | Notes                                      |
| ------ | ---------- | ---------- | ------------------------------------------ |
| Review | MerchantId | ReviewDate | Get reviews for merchant, newest first     |

**Key Design Decisions**:

- **Partition Key: MerchantId** (UUID)
  - Groups all reviews for a merchant
  - Foreign key reference to Merchants table
  - Enables "get all reviews for merchant" query

- **Sort Key: ReviewDate** (ISO 8601 datetime string)
  - Sorts reviews chronologically
  - Use ScanIndexForward: false for descending order (newest first)
  - Enables pagination through reviews

- **Projection: ALL**
  - Need all review fields for display
  - Includes: Rating, Comment, UserName, ReviewDate

---

## Access Patterns

### Implemented Patterns (Current)

| Pattern                          | API Call   | Index/Table   | Keys Used                          | Notes                        |
| -------------------------------- | ---------- | ------------- | ---------------------------------- | ---------------------------- |
| Get review by ID                 | GetItem    | Main Table    | ReviewId                           | Rare, mostly via merchant    |
| Get reviews for merchant         | Query      | MerchantIndex | MerchantId                         | All reviews                  |
| Get recent reviews for merchant  | Query      | MerchantIndex | MerchantId, Limit = 10             | Latest 10 reviews            |
| Paginate through reviews         | Query      | MerchantIndex | MerchantId + ExclusiveStartKey     | "Load More" button           |
| Create review                    | PutItem    | Main Table    | ReviewId (new UUID)                | User submits review          |
| Update review                    | UpdateItem | Main Table    | ReviewId                           | Edit comment/rating          |
| Delete review                    | DeleteItem | Main Table    | ReviewId                           | Remove review                |

---

## Referential Integrity

**Challenge**: DynamoDB doesn't enforce foreign keys

**Relationships**:
- Review → Merchant (many-to-one)

**Application-Layer Checks**:

Before creating review:
- Verify merchant exists
- Verify user hasn't already reviewed this merchant (if enforcing one review per user)

Before deleting merchant:
- Check for existing reviews
- Either prevent deletion or cascade delete reviews

---

## Evolution History

| Date       | Job                | Change                                    |
| ---------- | ------------------ | ----------------------------------------- |
| 2024-10-28 | Search by Category | Created Reviews table with MerchantIndex  |

---

## Related Entities

- **Merchant** - See entities/merchants.md
- **User** (Future) - For review authorship

---

## Implementation References

- **Entity Schema**: merchants-search-service/src/entities/review.entity.ts
- **Repository**: merchants-search-service/src/repositories/review.repository.ts
- **Service**: merchants-search-service/src/services/review.service.ts
