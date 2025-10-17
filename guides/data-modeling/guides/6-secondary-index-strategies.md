# Guide 6: Secondary Index Strategies

## When to Use This Guide

- [ ] Completed primary key design
- [ ] Have access patterns not satisfied by primary key
- [ ] Need to decide when and how to create GSIs

## Purpose

Understand when to use Global Secondary Indexes (GSIs), how to design them efficiently, and common patterns for leveraging them in your serverless microservices. GSIs are powerful tools for enabling additional access patterns beyond what your primary key can handle.

## Core Concepts

### What is a Global Secondary Index?

A GSI is an alternate key structure on your table that enables different query patterns. Think of it as a "view" of your data organized differently:

- **Different partition key**: Query items by attributes other than the main table's PK
- **Different sort key**: Enable different sorting or filtering requirements
- **Eventually consistent**: All GSI reads are eventually consistent
- **Separate throughput**: GSIs have their own read/write capacity (with on-demand billing)

### Key Decision: Do I Need a GSI?

**Start with these questions:**

1. **Can my primary key satisfy this pattern?**

   - If yes → Don't create a GSI
   - If no → Continue

2. **Is this a common access pattern or rare?**

   - Rare (analytics, admin tools) → Consider Scan or normalization
   - Common → Continue

3. **Is eventual consistency acceptable?**
   - No → Redesign primary key to handle it
   - Yes → GSI is appropriate

## Step-by-Step Process

### Step 1: Identify Unsatisfied Access Patterns

Review your access patterns table. Mark which patterns are NOT satisfied by your primary key design:

```markdown
| Entity | Access Pattern          | Satisfied by PK? | Notes                      |
| ------ | ----------------------- | ---------------- | -------------------------- |
| Order  | Get Orders for Customer | ✅ Yes           | PK = CUSTOMER#<username>   |
| Order  | Get Order by OrderId    | ❌ No            | OrderId not in primary key |
| Order  | Get Orders by Status    | ❌ No            | Status filtering needed    |
```

Patterns marked ❌ are candidates for GSIs.

### Step 2: Choose GSI Strategy

For each unsatisfied pattern, choose the appropriate strategy:

#### Strategy 1: Simple Lookup (Inverted Index)

**When:** Need to fetch an item by an alternate unique identifier.

**Example:** Get Order by OrderId when primary key is based on Customer

```typescript
// Main table
PK: CUSTOMER#<username>
SK: ORDER#<orderId>

// GSI1: Enable OrderId lookup
GSI1PK: ORDER#<orderId>
GSI1SK: ORDER#<orderId>
```

**Query code:**

```typescript
const result = await dynamodb.query({
  TableName: "AppTable",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :orderId",
  ExpressionAttributeValues: {
    ":orderId": "ORDER#550e8400",
  },
});
```

**Trade-offs:**

- ✅ Enables alternate entity lookup
- ✅ Simple to implement
- ❌ Must maintain GSI attributes on writes
- ❌ Eventually consistent reads

---

#### Strategy 2: Filtering by Attribute

**When:** Need to query items by a non-key attribute (e.g., Status, Type).

**Example:** Get all Open Orders across all customers

```typescript
// Main table
PK: CUSTOMER#<username>
SK: ORDER#<orderId>
Status: 'OPEN' | 'SHIPPED' | 'DELIVERED'

// GSI1: Filter by Status
GSI1PK: ORDER#<status>  // e.g., "ORDER#OPEN"
GSI1SK: <timestamp>      // Sorted by creation time
```

**Why composite sort key?**  
Groups all orders of same status together, sorted by time.

**Query code:**

```typescript
// Get all open orders, newest first
const result = await dynamodb.query({
  TableName: "AppTable",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :status",
  ExpressionAttributeValues: {
    ":status": "ORDER#OPEN",
  },
  ScanIndexForward: false, // Descending order
});
```

---

#### Strategy 3: Reverse Lookup (Many-to-Many)

**When:** Need to query relationships in both directions.

**Example:** User ↔ Repo access (adjacency list pattern)

```typescript
// Main table: User → Repos
PK: USER#<username>
SK: REPO#<owner>#<repo>

// GSI1: Repo → Users (reverse lookup)
GSI1PK: REPO#<owner>#<repo>
GSI1SK: USER#<username>
```

**Enables:**

- Main table: "Get all Repos for User"
- GSI1: "Get all Users with access to Repo"

---

#### Strategy 4: Sparse Index

**When:** Only some items have the attribute; need to query just those items.

**Example:** Get all Orders with tracking numbers

```typescript
// Main table
PK: CUSTOMER#<username>
SK: ORDER#<orderId>
TrackingNumber?: string  // Only set when shipped

// GSI1: Only shipped orders appear here
GSI1PK: ORDER#TRACKING
GSI1SK: <trackingNumber>
```

**Why sparse?**

- Only items with `GSI1PK` attribute appear in the index
- Items without it are automatically excluded
- Saves storage and throughput costs

**Benefits:**

- ✅ Natural filtering (no filter expressions needed)
- ✅ Lower costs (fewer items in index)
- ✅ Faster queries (smaller result sets)

---

#### Strategy 5: Overloaded GSI (Multiple Patterns)

**When:** Have multiple access patterns that can share a GSI.

**Example:** User profile lookups + Organization membership

```typescript
// Pattern 1: Get User by Email
User item:
  GSI1PK: EMAIL#<email>
  GSI1SK: USER#<username>

// Pattern 2: Get Users in Organization
OrgMember item:
  GSI1PK: ORG#<orgName>
  GSI1SK: USER#<username>
```

**Same GSI handles both patterns!**

- Query `GSI1PK = "EMAIL#alice@example.com"` → Returns User
- Query `GSI1PK = "ORG#acme"` → Returns all OrgMembers

**When to overload:**

- Patterns don't conflict (different partition keys)
- Need to conserve GSI slots (AWS limit: 20 GSIs per table)
- Want to reduce operational complexity

---

### Step 3: Design GSI Attributes

**Naming convention:**  
Use sequential generic names: `GSI1PK`, `GSI1SK`, `GSI2PK`, `GSI2SK`, etc..

**Why generic names?**

- Support multiple entity types in one index
- Enable GSI overloading
- Avoid confusion (same attribute name means different things per entity)

**CDK Example:**

```typescript
const table = new dynamodb.Table(this, "AppTable", {
  partitionKey: { name: "PK", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "SK", type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
});

// Add GSI1
table.addGlobalSecondaryIndex({
  indexName: "GSI1",
  partitionKey: { name: "GSI1PK", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "GSI1SK", type: dynamodb.AttributeType.STRING },
});

// Add GSI2 for another pattern
table.addGlobalSecondaryIndex({
  indexName: "GSI2",
  partitionKey: { name: "GSI2PK", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "GSI2SK", type: dynamodb.AttributeType.STRING },
});
```

### Step 4: Update Entity Chart

Document GSI attributes alongside primary key attributes:

**Main Table Entity Chart:**  
| Entity | PK | SK | GSI1PK | GSI1SK | Notes |  
|--------|----|----|--------|--------|-------|  
| Customer | CUSTOMER#<username> | CUSTOMER#<username> | EMAIL#<email> | CUSTOMER#<username> | Email lookup |  
| Order | CUSTOMER#<username> | ORDER#<orderId> | ORDER#<orderId> | ORDER#<orderId> | OrderId lookup |

**GSI1 Entity Chart:**  
| Entity | GSI1PK | GSI1SK | Access Pattern |  
|--------|--------|--------|----------------|  
| Customer | EMAIL#<email> | CUSTOMER#<username> | Get Customer by Email |  
| Order | ORDER#<orderId> | ORDER#<orderId> | Get Order by OrderId |

Save to: `docs/specs/jobs/<job>/entity-chart-gsi1.md`

### Step 5: Consider GSI Projection

**Projection types:**

1. **KEYS_ONLY**: Only keys are copied to GSI (cheapest)
2. **INCLUDE**: Keys + specified attributes
3. **ALL**: All attributes copied (most expensive, most flexible)

**Decision tree:**

```
Do you need non-key attributes in query results?
├─ No → Use KEYS_ONLY, fetch full item if needed
├─ Only a few → Use INCLUDE with specific attributes
└─ Most/all → Use ALL projection
```

**CDK Example with projection:**

```typescript
table.addGlobalSecondaryIndex({
  indexName: "GSI1",
  partitionKey: { name: "GSI1PK", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "GSI1SK", type: dynamodb.AttributeType.STRING },
  projectionType: dynamodb.ProjectionType.INCLUDE,
  nonKeyAttributes: ["Status", "CreatedAt", "Amount"], // Only these copied
});
```

**When to use KEYS_ONLY:**

- Access pattern just needs to identify items
- Will fetch full items via GetItem after query
- Minimizing storage costs is priority

**Example flow:**

```typescript
// 1. Query GSI1 (KEYS_ONLY) to get primary keys
const gsiResult = await dynamodb.query({
  TableName: "AppTable",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :orderId",
  ExpressionAttributeValues: { ":orderId": "ORDER#550e8400" },
});

// 2. Fetch full item from main table
const fullItem = await dynamodb.getItem({
  TableName: "AppTable",
  Key: {
    PK: gsiResult.Items[0].PK,
    SK: gsiResult.Items[0].SK,
  },
});
```

## Common Patterns

### Pattern 1: Entity Type Index

Create a GSI to query all items of a specific type:

```typescript
// All items have Type attribute
User: { Type: 'USER', ... }
Order: { Type: 'ORDER', ... }

// GSI1: Group by Type
GSI1PK: <Type>           // e.g., "USER", "ORDER"
GSI1SK: <createdAt>      // Sort by creation time

// Query: Get all recent Orders
query(GSI1PK = 'ORDER', ScanIndexForward = false, Limit = 20)
```

### Pattern 2: Time-Based Index

Enable time-range queries across entity types:

```typescript
// GSI1: All items sorted by time
GSI1PK: 'GLOBAL'              // Single partition
GSI1SK: <timestamp>#<type>    // e.g., "2024-01-15T10:30:00Z#ORDER"

// Query: Get all items in date range
query(
  GSI1PK = 'GLOBAL',
  GSI1SK BETWEEN '2024-01-01' AND '2024-01-31'
)
```

**Warning:** Single partition can become hot spot if high traffic. Consider sharding:

```typescript
GSI1PK: SHARD#<shardNumber>  // e.g., "SHARD#0", "SHARD#1", "SHARD#2"
```

### Pattern 3: Composite Filter Index

Combine two filters in sort key:

```typescript
// Pattern: Get Open Orders sorted by Priority
GSI1PK: ORDER
GSI1SK: <status>#<priority>#<orderId>
// e.g., "OPEN#HIGH#550e8400", "OPEN#LOW#660f9500"

// Query: Get high-priority open orders
query(
  GSI1PK = 'ORDER',
  GSI1SK begins_with 'OPEN#HIGH'
)
```

### Pattern 4: Multi-Tenant Index

Isolate data by tenant while enabling cross-tenant queries:

```typescript
// Main table: Data partitioned by Customer
PK: CUSTOMER#<customerId>
SK: ORDER#<orderId>

// GSI1: Cross-tenant view by Order status
GSI1PK: TENANT#<tenantId>
GSI1SK: STATUS#<status>#<orderId>

// Query: Get all Closed orders for Tenant A
query(
  GSI1PK = 'TENANT#a',
  GSI1SK begins_with 'STATUS#CLOSED'
)
```

## Validation Checklist

- [ ] Every unsatisfied access pattern assigned to a GSI
- [ ] GSI attribute names follow convention (GSI1PK, GSI1SK, etc.)
- [ ] Entity chart updated with GSI attributes for each entity
- [ ] Projection type chosen (KEYS_ONLY, INCLUDE, or ALL)
- [ ] Write operations updated to populate GSI attributes
- [ ] Query code tested against GSI
- [ ] Eventual consistency acceptable for these patterns
- [ ] GSI designs documented in `docs/specs/jobs/<job>/entity-chart-gsiX.md`

## Anti-Patterns to Avoid

❌ **Creating a GSI for every access pattern** → Use GSI overloading  
❌ **Using GSIs for rarely-used patterns** → Use Scan or normalization instead  
❌ **Assuming strong consistency** → GSIs are always eventually consistent  
❌ **Forgetting to add GSI attributes on writes** → Items won't appear in index  
❌ **Reusing attributes across indexes** → Keep them separate (GSI1PK ≠ GSI2PK)  
❌ **Over-projecting attributes** → KEYS_ONLY is often sufficient  
❌ **Not considering hot partitions** → Shard high-traffic GSI partition keys

## Cost Considerations

**GSI costs include:**

1. **Storage**: Size of all items + projected attributes
2. **Write throughput**: Every write to main table also writes to GSIs
3. **Read throughput**: Separate from main table reads

**Cost optimization tips:**

- Use KEYS_ONLY projection when possible
- Leverage sparse indexes to exclude unnecessary items
- Consider consolidating patterns into fewer GSIs via overloading
- Monitor GSI usage—remove unused indexes

## Migration Strategy

**Adding a GSI to existing table:**

1. **Add GSI to CDK stack** (does NOT backfill automatically)
2. **Deploy stack** (GSI created but empty)
3. **Backfill script**: Scan table and add GSI attributes:

```typescript
// Backfill script
const items = await scanTable("AppTable");

for (const item of items) {
  if (item.Type === "ORDER") {
    await dynamodb.updateItem({
      TableName: "AppTable",
      Key: { PK: item.PK, SK: item.SK },
      UpdateExpression: "SET GSI1PK = :gsi1pk, GSI1SK = :gsi1sk",
      ExpressionAttributeValues: {
        ":gsi1pk": `ORDER#${item.OrderId}`,
        ":gsi1sk": `ORDER#${item.OrderId}`,
      },
    });
  }
}
```

4. **Update application code** to populate GSI attributes on new writes
5. **Verify** GSI queries return expected results

**Removing a GSI:**

1. Stop populating GSI attributes in writes (application code)
2. Wait 24-48 hours (ensure no dependencies)
3. Remove GSI from CDK stack
4. Deploy

## Example: E-Commerce Orders

**Requirements:**

- Get Orders for Customer (primary key)
- Get Order by OrderId (need GSI)
- Get all Orders by Status (need GSI)

**Main Table:**

```typescript
| Entity | PK | SK | GSI1PK | GSI1SK | GSI2PK | GSI2SK |
|--------|----|----|--------|--------|--------|--------|
| Customer | CUSTOMER#<username> | CUSTOMER#<username> | - | - | - | - |
| Order | CUSTOMER#<username> | #ORDER#<orderId> | ORDER#<orderId> | ORDER#<orderId> | ORDER#<status> | <timestamp> |
```

**GSI1: Order Lookup**

- Query: `GSI1PK = "ORDER#550e8400"` → Returns Order

**GSI2: Orders by Status**

- Query: `GSI2PK = "ORDER#SHIPPED", ScanIndexForward = false` → Returns all shipped orders, newest first

**Access Patterns Satisfied:**  
| Pattern | Index | Query |  
|---------|-------|-------|  
| Get Orders for Customer | Main | PK = CUSTOMER#alice |  
| Get Order by OrderId | GSI1 | GSI1PK = ORDER#550e8400 |  
| Get Recent Shipped Orders | GSI2 | GSI2PK = ORDER#SHIPPED, DESC |
