# Guide 5: Relationship Strategies Quick Reference

## When to Use This Guide

- [ ] Modeling relationships between entities in your data model
- [ ] Deciding between different one-to-many or many-to-many patterns
- [ ] Need quick reference for common relationship patterns from the DynamoDB Book

## Purpose

Provide quick decision trees and implementation patterns for the most common relationship types in DynamoDB. This guide distills the strategies from Chapters 11-12 of the DynamoDB Book into actionable patterns for your serverless microservices.

## Decision Framework

When modeling a relationship, ask yourself:

1. **How many related items?** (Bounded vs Unbounded)
2. **Do I need to query in both directions?**
3. **How frequently does this data change?**
4. **What's the read/write ratio?**

---

## One-to-Many Relationships

### Pattern 1: Denormalization with Complex Attribute

**When to use:**

- Relationship is one-to-few (typically < 5-10 items)
- Related items are always accessed together with parent
- Related items rarely change independently
- Related items don't need to be queried separately

**Implementation:**  
Store related items as a List or Map attribute on the parent item.

```typescript
interface Organization extends BaseEntity {
  Type: "ORGANIZATION";
  OrgId: string;
  Name: string;

  // Denormalized as Map
  PaymentPlan: {
    PlanType: "FREE" | "PRO" | "ENTERPRISE";
    MaxUsers: number;
    BillingDate?: string;
  };

  // Denormalized as List
  FeaturedDeals: Array<{
    DealId: string;
    Position: number;
  }>;
}
```

**Trade-offs:**

- ✅ Single request to fetch parent + children
- ✅ Strongly consistent reads
- ✅ Transactional updates (parent + children updated together)
- ❌ Cannot query children independently
- ❌ Limited by 400KB item size
- ❌ All children retrieved even if you only need one

**CDK Example:**  
No special index needed—just use main table's primary key.

---

### Pattern 2: Denormalization by Duplicating Data

**When to use:**

- Need to display parent info when querying children
- Parent data rarely changes
- Willing to accept eventual consistency on parent updates
- Read-heavy workload

**Implementation:**  
Copy frequently-accessed parent attributes onto child items.

```typescript
// Parent item
interface Customer extends BaseEntity {
  Type: "CUSTOMER";
  CustomerId: string;
  CustomerName: string;
  Email: string;
}

// Child item with duplicated parent data
interface Order extends BaseEntity {
  Type: "ORDER";
  OrderId: string;
  CustomerId: string;

  // Duplicated from Customer
  CustomerName: string; // ← Denormalized

  OrderDate: string;
  TotalAmount: number;
}
```

**Trade-offs:**

- ✅ Avoid additional request to fetch parent details
- ✅ Faster reads (no secondary query needed)
- ❌ Parent updates require updating all children (eventual consistency)
- ❌ Increased storage costs
- ❌ Risk of stale data if parent changes

**When to avoid:**

- Parent data changes frequently
- Strong consistency required between parent and children

---

### Pattern 3: Composite Primary Key + Query API

**When to use:**

- Relationship is one-to-many (potentially unbounded)
- Need to fetch parent and all children together
- Children should be stored separately from parent
- This is the **most common pattern** for one-to-many relationships

**Implementation:**  
Use same partition key for parent and children; differentiate with sort key.

```typescript
// Entity chart
| Entity    | PK                      | SK                      |
|-----------|-------------------------|-------------------------|
| Customer  | CUSTOMER#<customerId>   | CUSTOMER#<customerId>   |
| Order     | CUSTOMER#<customerId>   | ORDER#<orderId>         |
```

**Access pattern:**

```typescript
// Get Customer + all Orders
const result = await dynamodb.query({
  TableName: "AppTable",
  KeyConditionExpression: "PK = :pk",
  ExpressionAttributeValues: {
    ":pk": { S: "CUSTOMER#c123" },
  },
});
// Returns: Customer item + all Order items
```

**Trade-offs:**

- ✅ Efficient fetching of parent + children in one query
- ✅ No item size limits (can have thousands of children)
- ✅ Pagination support for large result sets
- ✅ Children can be updated independently
- ❌ Cannot query children without knowing parent
- ❌ No built-in sorting by child attributes (see Pattern 5)

**Primary key design tips:**

- Use prefix on sort key to control sort order: `#ORDER#<orderId>` sorts after `CUSTOMER#<customerId>`
- Include timestamp for time-based queries: `ORDER#<iso-timestamp>`

---

### Pattern 4: Secondary Index + Query API

**When to use:**

- Need to query children without knowing parent
- Need to query by child attribute (e.g., OrderStatus)
- Already using Pattern 3 for parent → children access

**Implementation:**  
Create GSI with child's identifier as partition key.

```typescript
// Main table (Pattern 3)
| Entity    | PK                      | SK                    |
|-----------|-------------------------|-----------------------|
| Customer  | CUSTOMER#<customerId>   | CUSTOMER#<customerId> |
| Order     | CUSTOMER#<customerId>   | ORDER#<orderId>       |

// GSI1: Query orders by OrderId
| Entity    | GSI1PK              | GSI1SK                |
|-----------|---------------------|------------------------|
| Order     | ORDER#<orderId>     | ORDER#<orderId>        |
```

**Access patterns:**

```typescript
// 1. Get Customer + all Orders (main table)
query(PK = 'CUSTOMER#c123')

// 2. Get Order by OrderId (GSI1)
query(IndexName: 'GSI1', GSI1PK = 'ORDER#o456')
```

**Trade-offs:**

- ✅ Bi-directional queries (parent → children AND child lookup)
- ✅ Additional filtering on child attributes via GSI
- ❌ Eventually consistent reads from GSI
- ❌ Additional throughput costs for GSI
- ❌ Must maintain GSI attributes on writes

---

### Pattern 5: Composite Sort Keys with Hierarchical Data

**When to use:**

- Hierarchical data with 3+ levels
- Need to query at different hierarchy levels
- Example: `Organization → Project → Task`

**Implementation:**  
Use composite sort key with delimited structure.

```typescript
// Entity chart
| Entity       | PK                      | SK                                    |
|--------------|-------------------------|---------------------------------------|
| Org          | ORG#<orgId>            | ORG#<orgId>                           |
| Project      | ORG#<orgId>            | PROJECT#<projectId>                   |
| Task         | ORG#<orgId>            | PROJECT#<projectId>#TASK#<taskId>     |
```

**Access patterns:**

```typescript
// Get Org only
query(PK = 'ORG#o1', SK begins_with 'ORG#')

// Get Org + all Projects
query(PK = 'ORG#o1', SK begins_with 'ORG#' OR 'PROJECT#')

// Get Project + all Tasks
query(PK = 'ORG#o1', SK begins_with 'PROJECT#p1')

// Get specific Task
query(PK = 'ORG#o1', SK = 'PROJECT#p1#TASK#t1')
```

**Trade-offs:**

- ✅ Query at any hierarchy level with `begins_with()`
- ✅ Flexible traversal of tree structure
- ❌ Sort key becomes complex
- ❌ Refactoring hierarchy is difficult
- ❌ Deep nesting can hit sort key size limits

---

## Many-to-Many Relationships

### Decision Tree for Many-to-Many

```
Is the relationship sparse (few connections per item)?
├─ Yes → Use Adjacency List (Pattern 1)
└─ No → Continue

Do you need to query in both directions frequently?
├─ Yes → Use Adjacency List with GSI (Pattern 1 + GSI)
└─ No → Continue

Do you need metadata on the relationship itself?
├─ Yes → Use Adjacency List (Pattern 1)
└─ No → Continue

Is denormalization acceptable?
├─ Yes → Use Shallow Duplication (Pattern 2)
└─ No → Use Normalization (Pattern 4)
```

---

### Pattern 1: Adjacency List

**When to use:**

- Classic many-to-many relationship
- Need metadata on the relationship (e.g., GitHub User → Repo access level)
- Need to query in both directions

**Implementation:**  
Create items representing the edges between entities.

```typescript
// Entity chart
| Entity          | PK                    | SK                    | Attributes        |
|-----------------|-----------------------|-----------------------|-------------------|
| User            | USER#<username>       | USER#<username>       | Name, Email, ...  |
| Repo            | REPO#<owner>#<repo>   | REPO#<owner>#<repo>   | Description, ...  |
| UserRepoAccess  | USER#<username>       | REPO#<owner>#<repo>   | Role: 'ADMIN'     |
```

**Access patterns:**

```typescript
// Get User + all Repos they have access to
query(PK = 'USER#alexdebrie', SK begins_with 'REPO#')

// Get Repo + all Users with access
// Requires GSI with PK=Repo, SK=User
query(IndexName: 'GSI1', GSI1PK = 'REPO#aws#dynamodb')
```

**CDK Example:**

```typescript
// GSI for reverse lookup
table.addGlobalSecondaryIndex({
  indexName: "GSI1",
  partitionKey: { name: "GSI1PK", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "GSI1SK", type: dynamodb.AttributeType.STRING },
});
```

**Trade-offs:**

- ✅ Supports querying in both directions (with GSI)
- ✅ Can store relationship metadata
- ✅ Add/remove relationships without touching core entities
- ❌ Extra items increase storage costs
- ❌ Three items involved in some transactions (User, Repo, UserRepoAccess)

---

### Pattern 2: Shallow Duplication

**When to use:**

- Need basic info about related entities
- Related entities rarely change
- Read-heavy workload

**Implementation:**  
Store limited attributes from related entity directly on parent.

```typescript
interface Actor extends BaseEntity {
  Type: "ACTOR";
  ActorName: string;
  Bio: string;

  // Shallow duplication of movies
  Movies: Array<{
    MovieTitle: string;
    Year: number;
    Role: string;
  }>;
}

interface Movie extends BaseEntity {
  Type: "MOVIE";
  MovieTitle: string;
  Director: string;

  // Shallow duplication of actors
  Actors: Array<{
    ActorName: string;
    Role: string;
  }>;
}
```

**Trade-offs:**

- ✅ Query in both directions without GSI
- ✅ Single request to get entity + related summaries
- ❌ Stale data if related entities change
- ❌ Item size limits (400KB)
- ❌ Update complexity when shared data changes

**When to avoid:**

- Entities change frequently
- Need real-time consistency

---

### Pattern 3: Normalization (Multiple Requests)

**When to use:**

- Strong consistency required
- Relationships rarely queried together
- Simplicity preferred over performance

**Implementation:**  
Store entities separately; make multiple requests.

```typescript
// Request 1: Get User
const user = await getItem({ PK: "USER#alexdebrie" });

// Request 2: Get related Repos
const repoIds = user.RepoIds; // ['repo1', 'repo2']
const repos = await batchGetItem(repoIds);
```

**Trade-offs:**

- ✅ No data duplication
- ✅ Strong consistency
- ✅ Simple to understand
- ❌ Multiple round-trips to DynamoDB
- ❌ Cannot use transactions across both requests
- ❌ Slower performance

**When acceptable:**

- Background jobs (not latency-sensitive)
- Admin interfaces
- Low-traffic access patterns

---

## Summary Table

| Relationship | Pattern              | Use When                                | Avoid When               |
| ------------ | -------------------- | --------------------------------------- | ------------------------ |
| One-to-Few   | Complex Attribute    | Always accessed together, rarely change | Items queried separately |
| One-to-Many  | Composite PK + Query | Most common case                        | Need child-first queries |
| One-to-Many  | GSI                  | Need bi-directional queries             | Only parent-first access |
| Hierarchy    | Composite Sort Key   | Multi-level traversal needed            | Shallow hierarchy        |
| Many-to-Many | Adjacency List       | Need relationship metadata              | Simple connections only  |
| Many-to-Many | Shallow Duplication  | Read-heavy, stable data                 | Frequent updates         |

---

## Anti-Patterns to Avoid

❌ **Creating a GSI for every relationship** → Overload GSIs instead  
❌ **Making multiple serial requests** → Use composite keys and Query  
❌ **Storing unbounded arrays in items** → Will hit 400KB limit  
❌ **Not prefixing sort keys** → Makes queries ambiguous  
❌ **Denormalizing frequently-changing data** → Stale data risks
