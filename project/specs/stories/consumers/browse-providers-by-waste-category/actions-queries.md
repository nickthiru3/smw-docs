# Actions and Queries

**Story**: 001 - Browse Providers by Waste Category  
**Last Updated**: 2024-11-06

---

## Overview

This story involves **read-only operations** (queries only). No state modifications occur in the backend. All filtering, sorting, and view manipulations happen client-side after the initial data fetch.

---

## Queries

### Query 1: Get Merchants by Category

**Description**: Retrieves all merchants that offer services in a specified category (Repair, Refill, Recycling, Donate). Returns complete merchant data including location coordinates for client-side distance calculations.

**Type**: Query (Read-only, no side effects)

**API Endpoint**:

- **BFF**: `GET /api/merchants?category={category}`
- **Backend Service**: `GET /merchants?category={category}`

**Inputs**:

- `category` (string, required) - The primary service category
  - Valid values: `"Repair"`, `"Refill"`, `"Recycling"`, `"Donate"`
  - Validation: Must be one of the allowed enum values
  - Case-sensitive

**Expected Output**:

```json
{
  "merchants": [
    {
      "merchantId": "uuid-string",
      "legalName": "Green Repair Shop",
      "tradingName": "Green Repair",
      "shortDescription": "Expert electronics and appliance repair",
      "primaryCategory": "Repair",
      "categories": ["Repair", "Recycling"],
      "verificationStatus": "Verified",
      "location": {
        "address": "123 Main St",
        "city": "Toronto",
        "state": "ON",
        "postalCode": "M5V 1A1",
        "latitude": 43.6532,
        "longitude": -79.3832
      },
      "contact": {
        "phoneNumber": "+1-416-555-0123",
        "email": "contact@greenrepair.com",
        "websiteUrl": "https://greenrepair.com"
      },
      "rating": {
        "average": 4.5,
        "count": 127
      },
      "services": [
        {
          "name": "Electronics Repair",
          "description": "Smartphones, laptops, tablets"
        }
      ],
      "operatingHours": [
        {
          "dayOfWeek": "Monday",
          "openTime": "09:00",
          "closeTime": "17:00"
        }
      ],
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-10-28T14:22:00Z"
    }
  ],
  "count": 42,
  "category": "Repair"
}
```

**Error Cases**:

- `400 Bad Request`: Invalid category value
  ```json
  {
    "error": "Invalid category",
    "message": "Category must be one of: Repair, Refill, Recycling, Donate",
    "code": "INVALID_CATEGORY"
  }
  ```
- `500 Internal Server Error`: Database query failure
  ```json
  {
    "error": "Internal server error",
    "message": "Failed to retrieve merchants",
    "code": "QUERY_FAILED"
  }
  ```

**Caching**:

- **BFF Layer**: Can be cached for 5 minutes (merchants data changes infrequently)
- **CDN**: Can be cached with category-based cache keys
- **Cache Invalidation**: On merchant create/update/delete operations

**Performance Target**:

- **Backend Query**: <100ms (DynamoDB GSI1 query)
- **BFF Response**: <150ms (including transformation)
- **Total (UI → BFF → Backend → UI)**: <200ms

**DynamoDB Query Details**:

```typescript
{
  TableName: "Merchants",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :category",
  ExpressionAttributeValues: {
    ":category": "Repair"
  }
}
```

**Client-Side Processing** (not part of backend API):

After receiving the response, the frontend performs:

1. **Distance Calculation**: Haversine formula using user location and merchant lat/lng
2. **Radius Filtering**: Filter merchants within selected radius (5km, 10km, 20km)
3. **Additional Filtering**: Category tags, rating thresholds, verification status
4. **Sorting**: By distance (ascending)
5. **Rendering**: List view and map view with markers

---

## Actions

**None** - This story contains no state-modifying operations. All interactions are read-only queries.

---

## Summary

| Operation                    | Type  | Modifies State | Idempotent | Cacheable | Priority |
| ---------------------------- | ----- | -------------- | ---------- | --------- | -------- |
| Search Merchants by Category | Query | No             | Yes        | Yes (5m)  | High     |

---

## Notes

### Architecture Decisions

1. **No Location Parameters in API**: User location (lat/lng) and radius are NOT passed to the backend. This allows:

   - Response caching (same for all users)
   - Instant client-side radius adjustments
   - Flexible distance calculation methods

2. **Return All Merchants in Category**: Backend returns complete dataset for the category. Client filters by distance. Benefits:

   - Simpler backend logic
   - Better caching
   - Instant filter adjustments
   - No pagination complexity for MVP

3. **Complete Merchant Objects**: Return full merchant data (not just IDs) to avoid N+1 query problems and enable offline-capable UI.

### Future Considerations

- **Phase 1b**: May add pagination if category result sets become very large (>500 merchants)
- **Phase 2**: Consider ElasticSearch/OpenSearch for complex geospatial queries and full-text search
- **Phase 2**: Add filtering by subcategories or tags (e.g., "Electronics Repair" within "Repair")
