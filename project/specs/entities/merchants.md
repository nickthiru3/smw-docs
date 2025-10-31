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

### CategoryIndex

**Purpose**: Enable searching merchants by category (primary use case for Search by Category job)

| Entity   | GSI PK   | GSI SK | Notes                       |
| -------- | -------- | ------ | --------------------------- |
| Merchant | Category | -      | Query merchants by category |

**Key Design Decisions**:

- **Partition Key: Category** (String)
  - Values: "Repair", "Refill", "Recycling", "Donate"
  - Groups all merchants by primary category
  - Low cardinality (4 categories in Phase 1a)

- **No Sort Key**
  - Client-side sorting by distance after fetching results
  - Alternative considered: Location as SK, but geospatial queries need client-side calculation anyway
  - Keeps GSI simple and flexible

- **Projection: ALL**
  - Includes all merchant attributes
  - Simplest approach for MVP
  - Can optimize to KEYS_ONLY or INCLUDE later if costs become concern

**Performance Considerations**:

- **Hot partitions**: Possible if one category has significantly more merchants
  - Mitigation: Monitor CloudWatch metrics, consider sharding if needed
- **Result set size**: Potentially large (hundreds of merchants per category)
  - Mitigation: Implement pagination, limit results to reasonable radius

**Future Enhancements**:
- Phase 1b: May add sort key for advanced filtering
- Phase 2: Consider ElasticSearch/OpenSearch for complex geospatial queries

---

## Access Patterns

### Implemented Patterns (Current)

| Pattern                        | API Call | Index/Table   | Keys Used            | Notes                           |
| ------------------------------ | -------- | ------------- | -------------------- | ------------------------------- |
| Get merchant by ID             | GetItem  | Main Table    | MerchantId           | Direct lookup from search click |
| Search merchants by category   | Query    | CategoryIndex | Category = "Repair"  | Returns all in category         |
| Filter by category + location  | Query    | CategoryIndex | Category = "Repair"  | Client filters by distance      |
| Create merchant                | PutItem  | Main Table    | MerchantId (new UUID)| New merchant registration       |
| Update merchant                | UpdateItem | Main Table  | MerchantId           | Edit merchant details           |
| Delete merchant                | DeleteItem | Main Table  | MerchantId           | Soft delete preferred           |

### Query Examples

**Get merchant by ID**:
```typescript
const result = await docClient.send(
  new GetCommand({
    TableName: "Merchants",
    Key: { MerchantId: merchantId },
  })
);
```

**Search by category**:
```typescript
const result = await docClient.send(
  new QueryCommand({
    TableName: "Merchants",
    IndexName: "CategoryIndex",
    KeyConditionExpression: "Category = :category",
    ExpressionAttributeValues: {
      ":category": "Repair",
    },
  })
);

// Client-side: Filter by distance, sort by rating/distance
const nearbyMerchants = result.Items
  .map(merchant => ({
    ...merchant,
    distance: calculateDistance(userLocation, merchant.Location)
  }))
  .filter(m => m.distance <= radiusKm)
  .sort((a, b) => a.distance - b.distance);
```

### Planned Patterns (Future Jobs)

| Pattern                          | Index Needed   | Job/Phase              | Notes                           |
| -------------------------------- | -------------- | ---------------------- | ------------------------------- |
| Get merchant by email            | EmailIndex GSI | Merchant Portal (Phase 2) | Login, password reset        |
| Filter by verification status    | StatusIndex GSI| Admin Dashboard (Phase 3) | Admin filtering              |
| Search by product tags           | TagsIndex GSI  | Enhanced Search (Phase 1b)| Multi-tag filtering          |
| Get merchants by signup date     | DateIndex GSI  | Analytics (Phase 3)    | Growth metrics               |

---

## Attributes

**Note**: Attributes are defined in TypeScript entity schemas (not in ERD/entity-key tables). This section provides a reference.

### Core Identity
- `MerchantId` (string, UUID) - Primary key
- `LegalName` (string) - Official business name
- `TradingName` (string, optional) - Display name
- `ShortDescription` (string) - Brief description for search results
- `PrimaryCategory` (enum) - Repair | Refill | Recycling | Donate

### Location
- `PrimaryAddress` (string) - Street address
- `City` (string)
- `State` (string)
- `PostalCode` (string)
- `Latitude` (number) - GPS coordinate
- `Longitude` (number) - GPS coordinate

### Contact
- `PhoneNumber` (string)
- `Email` (string)
- `WebsiteUrl` (string, optional)

### Search Metadata
- `Categories` (string[]) - Array of categories (max 4, denormalized)
- `Services` (Service[]) - List of services offered (embedded, bounded)

### Metrics (Denormalized)
- `RatingAverage` (number) - Average rating (0-5)
- `RatingCount` (number) - Total number of reviews

### Operational
- `OperatingHours` (OperatingHours[]) - Weekly schedule (embedded, bounded)

### Timestamps
- `CreatedAt` (datetime, ISO 8601)
- `UpdatedAt` (datetime, ISO 8601)

**Type Definitions**: See `merchants-search-service/src/entities/merchant.entity.ts`

---

## Design Decisions Log

| Date       | Decision                                      | Rationale                                                                 |
| ---------- | --------------------------------------------- | ------------------------------------------------------------------------- |
| 2024-10-28 | Simple primary key (MerchantId only)         | No parent-child relationships, all queries via GSIs                       |
| 2024-10-28 | CategoryIndex with no sort key                | Client-side distance filtering, keeps GSI simple                          |
| 2024-10-28 | Denormalize rating average/count              | Avoid join with Reviews table for sorting                                 |
| 2024-10-28 | Embed operating hours (bounded list)          | Max 7 items, always displayed together                                    |
| 2024-10-28 | Embed services list (bounded)                 | Max 10 items for Basic tier, always displayed together                    |
| 2024-10-28 | Separate Reviews table                        | Unbounded relationship (hundreds of reviews possible)                     |

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
  await Promise.all(reviews.map(r => deleteReview(r.ReviewId)));
}
```

---

## Evolution History

| Date       | Job                | Change                                    |
| ---------- | ------------------ | ----------------------------------------- |
| 2024-10-28 | Search by Category | Created Merchants table with CategoryIndex|

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
