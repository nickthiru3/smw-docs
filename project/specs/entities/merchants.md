# Merchant Entity

**Table Name**: Merchants  
**Approach**: Faux-SQL (descriptive key names, separate table)  
**Created By**: Search by Category job  
**Last Updated**: 2024-10-28

---

## Overview

The Merchant entity represents circular-economy service providers (repair shops, refill stations, recycling centers, donation centers) that consumers can search for and view details about.

**Core Purpose**: Enable consumers to discover and learn about sustainable service providers in their area.

---

## Access Patterns

| Access Pattern               | Table/Index | Parameters | Notes                                                       |
| ---------------------------- | ----------- | ---------- | ----------------------------------------------------------- |
| Get merchant by ID           | Main table  | merchantId | GetItem - Direct lookup from search click                   |
| Search merchants by category | GSI1        | category   | Query - Returns all in category; client filters by distance |

### Planned Patterns (Future Jobs)

| Access Pattern                | API Call   | Index Needed | Job/Phase                  | Notes                                              |
| ----------------------------- | ---------- | ------------ | -------------------------- | -------------------------------------------------- |
| Create merchant               | PutItem    | Main Table   | Merchant Portal (Phase 2)  | New merchant registration                          |
| Update merchant               | UpdateItem | Main Table   | Merchant Portal (Phase 2)  | Edit merchant details                              |
| Delete merchant               | DeleteItem | Main Table   | Merchant Portal (Phase 2)  | Soft delete preferred                              |
| Get merchant by email         | Query      | GSI2         | Merchant Portal (Phase 2)  | Login, password reset (GSI2PK: Email)              |
| Filter by verification status | Query      | GSI3         | Admin Dashboard (Phase 3)  | Admin filtering (GSI3PK: VerificationStatus)       |
| Search by product tags        | Query      | GSI4         | Enhanced Search (Phase 1b) | Multi-tag filtering (GSI4PK: Tag)                  |
| Get merchants by signup date  | Query      | GSI5         | Analytics (Phase 3)        | Growth metrics (GSI5PK: SignupDate or date bucket) |

---

## Main Table Structure

### Primary Key Design

| Entity   | Partition Key | Sort Key | Notes                                 |
| -------- | ------------- | -------- | ------------------------------------- |
| Merchant | MerchantId    | -        | Simple key, UUID, direct lookup by ID |

**Key Design Decisions**:

- **Simple key (partition key only)**: No parent-child relationships in Merchants table
- **UUID for MerchantId**: Globally unique, no collisions, generated at creation
- **No composite key needed**: All "get many" patterns handled via GSIs
- **Merchants are independent**: No hierarchical relationships

**Why Simple Key?**

- Merchants don't have child items in the same table
- All queries beyond "get by ID" use GSIs
- Simpler application code (no sort key management)

---

## Global Secondary Indexes

### GSI1: Category Queries

**Purpose**: Enable searching merchants by category (primary use case for Search by Category job)

**Index Name**: GSI1

| Entity   | GSI1PK       | GSI1SK | Notes                                                  |
| -------- | ------------ | ------ | ------------------------------------------------------ |
| Merchant | PrimaryCategory | -      | Generic attribute name for flexibility; stores category value |

**Key Design Decisions**:

- **Partition Key: GSI1PK** (String) - Generic attribute name
  - **Value stored**: PrimaryCategory (e.g., "Repair", "Refill", "Recycling", "Donate")
  - Groups all merchants by primary category
  - Low cardinality (4 categories in Phase 1a)
  - Generic name allows future reuse for related patterns

- **No Sort Key (GSI1SK)**
  - Client-side sorting by distance after fetching results
  - Alternative considered: Location as SK, but geospatial queries need client-side calculation anyway
  - Keeps GSI simple and flexible
  - Can add GSI1SK later if needed without schema migration

- **Projection: ALL**
  - Includes all merchant attributes
  - Simplest approach for MVP
  - Can optimize to KEYS_ONLY or INCLUDE later if costs become concern

**Item Structure Example**:

```typescript
{
  MerchantId: "merchant-123",
  LegalName: "Green Repair Shop",
  PrimaryCategory: "Repair",
  GSI1PK: "Repair",        // Generic GSI attribute
  // ... other attributes
}
```

**Query Example**:

```typescript
QueryCommand({
  TableName: "Merchants",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :category",
  ExpressionAttributeValues: {
    ":category": "Repair"
  }
})
```

**Performance Considerations**:

- **Hot partitions**: Possible if one category has significantly more merchants
  - Mitigation: Monitor CloudWatch metrics, consider sharding if needed
- **Result set size**: Potentially large (hundreds of merchants per category)
  - Mitigation: Implement pagination, limit results to reasonable radius

**Future Enhancements**:

- Phase 1b: May add GSI1SK for advanced filtering (e.g., verification status, rating)
- Phase 1b: Could reuse GSI1 for subcategory queries by changing GSI1PK value
- Phase 2: Consider ElasticSearch/OpenSearch for complex geospatial queries

---

## Entity Attributes

**Note**: This section documents the attributes for design purposes. TypeScript interfaces will be created during implementation by the backend service team.

### Primary Keys & GSI Attributes

- `MerchantId` (string, UUID, required) - Primary partition key
- `GSI1PK` (string, required) - Generic GSI attribute; stores PrimaryCategory value for category queries

### Core Identity

- `LegalName` (string, required) - Official business name
- `TradingName` (string, optional) - Display name (defaults to LegalName if not provided)
- `ShortDescription` (string, required, max 200 chars) - Brief description for search results
- `PrimaryCategory` (enum, required) - Repair | Refill | Recycling | Donate
- `VerificationStatus` (enum, required) - Pending | Verified | Rejected (for verification badge)

### Location

- `PrimaryAddress` (string, required) - Street address
- `City` (string, required)
- `State` (string, required)
- `PostalCode` (string, required)
- `Latitude` (number, required) - GPS coordinate for distance calculations
- `Longitude` (number, required) - GPS coordinate for distance calculations

### Contact

- `PhoneNumber` (string, required)
- `Email` (string, required)
- `WebsiteUrl` (string, optional)

### Search Metadata

- `Categories` (string[], max 4, required) - Array of categories for multi-category merchants (denormalized)
- `Services` (Service[], max 10, optional) - List of services offered (embedded object, bounded)

### Metrics (Denormalized)

- `RatingAverage` (number, 0-5, default 0) - Average rating from reviews
- `RatingCount` (number, default 0) - Total number of reviews

### Operational

- `OperatingHours` (OperatingHours[], max 7, optional) - Weekly schedule (embedded object, bounded)

### Timestamps

- `CreatedAt` (datetime, ISO 8601, required) - Merchant registration date
- `UpdatedAt` (datetime, ISO 8601, required) - Last modification date

**Type Definitions**: See `merchants-search-service/src/entities/merchant.entity.ts`

---

## Design Decisions Log

| Date       | Decision                             | Rationale                                                                       |
| ---------- | ------------------------------------ | ------------------------------------------------------------------------------- |
| 2024-10-28 | Simple primary key (MerchantId only) | No parent-child relationships, all queries via GSIs                             |
| 2024-10-28 | GSI1 with no sort key                | Client-side distance filtering, keeps GSI simple                                |
| 2024-11-06 | Generic GSI attribute names          | Use GSI1PK instead of Category for flexibility; allows future pattern reuse     |
| 2024-10-28 | Denormalize rating average/count     | Avoid join with Reviews table for sorting                                       |
| 2024-10-28 | Embed operating hours (bounded list) | Max 7 items, always displayed together                                          |
| 2024-10-28 | Embed services list (bounded)        | Max 10 items for Basic tier, always displayed together                          |
| 2024-10-28 | Separate Reviews table               | Unbounded relationship (hundreds of reviews possible)                           |
| 2024-11-06 | Reserve GSI2-GSI5 for future         | Plan ahead for email lookup, status filtering, tags, and analytics (agile-ready) |

---

## Anti-Pattern Validation

**Validation Date**: 2024-11-06  
**Validated By**: Data Modeling Step 10

### ✅ Passed Checks

**No unbounded attributes in items:**
- ✅ Reviews stored in separate table (not embedded in Merchant)
- ✅ Services array bounded to max 10 items
- ✅ OperatingHours bounded to max 7 items (one per day)
- ✅ Categories array bounded to max 4 items

**No hot partitions:**
- ✅ Each merchant has unique MerchantId as partition key
- ✅ GSI1 partitions by PrimaryCategory (4 categories, reasonable distribution)

**No scan operations for common queries:**
- ✅ Get by ID uses primary key (GetItem)
- ✅ Search by category uses GSI1 (Query)
- ✅ Future email lookup planned with GSI2

**No large items (>400KB limit):**
- ✅ All attributes are small (strings, numbers, small arrays)
- ✅ Estimated item size: ~2-5KB per merchant
- ✅ Well below 400KB limit

**Good Faux-SQL patterns:**
- ✅ One table for Merchants entity
- ✅ Descriptive primary key (MerchantId)
- ✅ Generic GSI attributes (GSI1PK) for flexibility
- ✅ Foreign key to Reviews table (separate entity)
- ✅ Bounded denormalization (rating metrics, operating hours)

### ⚠️ Considerations

**GSI1 potential hot partition:**
- If one category becomes significantly more popular than others
- **Mitigation**: Monitor CloudWatch metrics; consider sharding if needed in Phase 2
- **Current risk**: Low (4 categories, expected even distribution)

**Client-side distance filtering:**
- All merchants in category returned; client filters by distance
- **Mitigation**: Implement pagination, reasonable result limits
- **Future**: Consider ElasticSearch/OpenSearch for geospatial queries (Phase 2)

### 📋 Validation Summary

**Status**: ✅ **PASSED** - Design is ready for implementation

**Notes**:
- No critical anti-patterns detected
- Minor considerations documented with mitigation strategies
- Design follows Faux-SQL best practices
- Generic GSI naming allows future flexibility

---

## Referential Integrity

**Challenge**: DynamoDB doesn't enforce foreign keys

**Relationships**:

- Merchant → Reviews (one-to-many, unbounded)

**Application-Layer Checks**:

```typescript
// Before deleting merchant
const reviews = await getReviewsForMerchant(merchantId);
if (reviews.length > 0) {
  // Option 1: Prevent deletion
  throw new Error("Cannot delete merchant with existing reviews");

  // Option 2: Cascade delete (use carefully)
  await Promise.all(reviews.map((r) => deleteReview(r.ReviewId)));
}
```

---

## Evolution History

| Date       | Job                | Change                                                              |
| ---------- | ------------------ | ------------------------------------------------------------------- |
| 2024-10-28 | Search by Category | Created Merchants table with GSI1 for category queries              |
| 2024-11-06 | Search by Category | Updated GSI naming to use generic attributes (GSI1PK) for flexibility |

---

## Related Entities

- **Review** - See `entities/reviews.md`
- **User** (Future) - For favorites, authentication
- **Favorite** (Future) - For saved merchants

---

## Implementation References

- **Entity Schema**: `merchants-search-service/src/entities/merchant.entity.ts`
- **Repository**: `merchants-search-service/src/repositories/merchant.repository.ts`
- **Service**: `merchants-search-service/src/services/merchant.service.ts`
- **API Contract**: `merchants-search-service/docs/api/openapi.yml`
