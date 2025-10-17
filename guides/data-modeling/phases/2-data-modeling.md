# Phase 2: Data Modeling

## Overview

Phase 2 is where the rubber meets the road. You've gathered your requirements and access patterns in Phase 1. Now it's time to design your DynamoDB table to efficiently satisfy those patterns.

**Key Principle:** "You need to preassemble your data in the exact shape that is needed for a read operation".

## Objectives

By the end of Phase 2, you should have:

- [ ] Complete primary key design for all entities
- [ ] Entity charts for main table and all GSIs
- [ ] TypeScript entity schemas for all entities
- [ ] Documentation of relationship strategies
- [ ] Updated access patterns table with query details
- [ ] Validation that design avoids common anti-patterns

## Prerequisites

Before starting Phase 2:

- [ ] Phase 1 complete and approved
- [ ] ERD finalized
- [ ] All access patterns documented
- [ ] Stakeholder sign-off on requirements

## Steps

### Step 1: Design Primary Keys

**Duration:** 2-3 days

**Use Guide:** [Guide 3: Primary Key Design Guide](../guides/3-primary-key-design.md)

**Activities:**

1. Create entity chart with all entity types
2. Decide simple vs. composite primary key
3. Design PK/SK patterns for each entity
4. Validate uniqueness requirements
5. Map access patterns to primary key queries

**Decision Point: Simple vs. Composite Primary Key**

Use **Simple Primary Key** when:

- All access patterns fetch single items by ID
- No "fetch many" requirements
- No relationships between entities

Use **Composite Primary Key** when:

- Need to fetch multiple related items in one request
- Have one-to-many or many-to-many relationships
- Need hierarchical or time-series queries

**Primary Key Design Principles:**

1. **Client Knowledge**: Client must know the key at read time

   - ✅ Good: `PK: USER#<username>` (username from URL path)
   - ❌ Bad: `PK: USER#<timestamp>` (client doesn't know timestamp)

2. **Entity Prefixes**: Use prefixes to distinguish entity types

```

PK: CUSTOMER#C123 (not just "C123")
 SK: METADATA#C123 (not just "C123")

```

3. **Generic Names**: Use PK/SK, not descriptive names

- ✅ Good: PK, SK, GSI1PK, GSI1SK
- ❌ Bad: CustomerId, OrderDate

**Entity Chart Template:**

| Entity  | PK Pattern              | SK Pattern                      | Notes                               |
| ------- | ----------------------- | ------------------------------- | ----------------------------------- |
| User    | USER#\<username\>       | USER#\<username\>               | Simple lookup                       |
| Order   | CUSTOMER#\<customerId\> | ORDER#\<timestamp\>#\<orderId\> | Grouped by customer, sorted by time |
| Product | PRODUCT#\<productId\>   | PRODUCT#\<productId\>           | Simple lookup                       |

**Access Pattern Mapping:**

For each access pattern, document how it maps to your key design:

| Access Pattern          | API Call      | Key Condition                | Notes              |
| ----------------------- | ------------- | ---------------------------- | ------------------ |
| Get User by username    | GetItem       | PK = USER#\<username\>       | Direct lookup      |
| Get Orders for Customer | Query         | PK = CUSTOMER#\<customerId\> | Returns all orders |
| Get Order by OrderId    | GetItem (GSI) | GSI1PK = ORDER#\<orderId\>   | Requires GSI       |

**Validation Checklist:**

- [ ] Each entity has unique PK/SK combination
- [ ] All "Get by ID" patterns satisfied
- [ ] All "Get many" patterns grouped in item collections
- [ ] No timestamps in keys unless client knows them
- [ ] Generic key names used (PK, SK)
- [ ] Entity prefixes consistent across all entities

**Common Mistakes:**

- Using CreatedAt in primary key
- Descriptive key names (CustomerId, OrderDate)
- Not considering client knowledge at read time

**Output:**

- `docs/specs/jobs/<job>/entity-chart-main.md`
- Updated access patterns table

---

### Step 2: Model Relationships

**Duration:** 1-2 days

**Use Guide:** [Guide 5: Relationship Strategies Quick Reference](../guides/5-relationship-strategies.md)

**Activities:**

1. Review each relationship in ERD
2. Classify relationship type (1:1, 1:N, M:N)
3. Choose appropriate strategy for each relationship
4. Update entity chart with relationship items (if needed)
5. Document strategy decisions

**Relationship Strategy Decision Tree:**

```

Is relationship 1:1, 1:N, or M:N?
│
├─ 1:1 → Denormalize into single item
│
├─ 1:N → Is "N" bounded or unbounded?
│ ├─ Bounded (<20 items) → Denormalize with complex attribute
│ └─ Unbounded → Item collection with composite key
│
└─ M:N → Choose strategy:
 ├─ Shallow duplication (copy limited attributes)
 ├─ Adjacency list (same table, multiple item collections)
 ├─ Materialized graph (GSI for reverse lookups)
 └─ Normalization (separate requests)

```

**Common Relationship Patterns:**

**One-to-One:**

```typescript
// Denormalize: Store related data on same item
interface User {
  PK: "USER#alexdebrie";
  SK: "USER#alexdebrie";
  Username: "alexdebrie";
  Profile: {
    // 1:1 relationship embedded
    Bio: "...";
    AvatarUrl: "...";
  };
}
```

**One-to-Many (Bounded):**

```typescript
// Denormalize: Use List or Map attribute
interface Customer {
  PK: "CUSTOMER#C123";
  SK: "CUSTOMER#C123";
  Name: "Acme Corp";
  Addresses: [
    // Max 5 addresses
    { Street: "123 Main"; City: "Seattle" },
    { Street: "456 Oak"; City: "Portland" }
  ];
}
```

**One-to-Many (Unbounded):**

```typescript
// Item collection: Separate items, same PK
// Customer item:
{ PK: 'CUSTOMER#C123', SK: 'CUSTOMER#C123', Name: 'Acme' }

// Order items (share PK):
{ PK: 'CUSTOMER#C123', SK: 'ORDER#2020-01-15#O456' }
{ PK: 'CUSTOMER#C123', SK: 'ORDER#2020-01-20#O789' }
```

**Many-to-Many:**

```typescript
// Adjacency list: Items represent relationships
// User item:
{ PK: 'USER#alice', SK: 'USER#alice', Name: 'Alice' }

// Membership items:
{ PK: 'USER#alice', SK: 'ORG#acme', Role: 'Admin' }
{ PK: 'USER#alice', SK: 'ORG#widgets', Role: 'Member' }

// Organization items:
{ PK: 'ORG#acme', SK: 'ORG#acme', Name: 'Acme Corp' }
{ PK: 'ORG#acme', SK: 'USER#alice', Role: 'Admin' }
```

**Relationship Documentation Template:**

Create `docs/specs/jobs/<job>/relationship-decisions.md`:

```markdown
# Relationship Strategy Decisions

## User → Orders (1:N, Unbounded)

**Strategy:** Item collection with composite primary key

**Rationale:**

- User can have unlimited orders (unbounded)
- Need to query "Get recent orders for user"
- Orders sorted by timestamp

**Implementation:**

- User: `PK: USER#<username>`, `SK: USER#<username>`
- Order: `PK: USER#<username>`, `SK: ORDER#<timestamp>#<orderId>`

**Query:**
```

KeyConditionExpression: PK = USER#alice AND SK begins_with ORDER#

```

## User → Profile (1:1)

**Strategy:** Denormalization into single item

**Rationale:**
- Always accessed together
- Profile data is small
- Simpler model

**Implementation:**
User item contains ProfileData map attribute
```

**Validation Checklist:**

- [ ] All ERD relationships have documented strategies
- [ ] Unbounded 1:N not stored as lists
- [ ] Bounded 1:N lists have reasonable limits (<20)
- [ ] M:N relationships support both access directions
- [ ] Relationship items added to entity chart (if applicable)

**Output:**

- `docs/specs/jobs/<job>/relationship-decisions.md`
- Updated entity chart

---

### Step 3: Define Entity Schemas

**Duration:** 1 day

**Use Guide:** [Guide 4: Entity Schema Guide](../guides/4-entity-schema.md)

**Activities:**

1. Create TypeScript interfaces for each entity
2. Document all application attributes (non-key attributes)
3. Define complex types (nested objects, lists)
4. Add Type attribute to all entities
5. Document attribute purposes and constraints

**Schema Structure:**

```typescript
// Base interface for all items
interface BaseItem {
  PK: string;
  SK: string;
  Type: string; // Always include Type
  CreatedAt: string; // ISO-8601 timestamp
  UpdatedAt: string; // ISO-8601 timestamp
}

// Entity-specific interface
interface User extends BaseItem {
  Type: "User"; // Literal type
  Username: string;
  Email: string;
  Name: string;
  // ... other attributes
}

interface Order extends BaseItem {
  Type: "Order";
  OrderId: string;
  CustomerId: string;
  TotalAmount: number;
  Status: "PENDING" | "SHIPPED" | "DELIVERED";
  Items: OrderItem[]; // Nested complex type
}

interface OrderItem {
  ProductId: string;
  Quantity: number;
  PriceAtPurchase: number;
}
```

**Attribute Guidelines:**

1. **Always Include Type**

   - Makes debugging easier
   - Enables ETL filtering
   - Clarifies console views

2. **Use Timestamps Appropriately**

   - ISO-8601 format for human readability
   - Unix timestamps for range queries
   - Never in primary key unless client knows it

3. **Complex Attributes**

   - Use maps for structured nested data
   - Use lists for ordered collections (bounded size)
   - Use sets for unique collections

4. **Avoid Over-Nesting**
   - Keep nesting depth reasonable (≤3 levels)
   - Consider separate items for deeply nested data

**Type Attribute Pattern:**

```typescript
// Filtering by Type in Scan operations
const users = await dynamodb.scan({
  TableName: "AppTable",
  FilterExpression: "Type = :type",
  ExpressionAttributeValues: {
    ":type": "User",
  },
});
```

**Validation Checklist:**

- [ ] All entities have TypeScript interfaces
- [ ] Type attribute included on every entity
- [ ] Complex attributes (maps, lists) properly defined
- [ ] No unbounded lists as attributes
- [ ] Timestamps in appropriate format
- [ ] Attribute names not DynamoDB reserved words

**Output:**

- `docs/specs/jobs/<job>/entity-schemas.ts`

---

### Step 4: Design Secondary Indexes

**Duration:** 2-3 days

**Use Guide:** [Guide 6: Secondary Index Strategies](../guides/6-secondary-index-strategies.md)

**Activities:**

1. Identify access patterns not satisfied by primary key
2. Group patterns by common access characteristics
3. Design GSI key structures to satisfy patterns
4. Create entity charts for each GSI
5. Update access patterns table with GSI queries
6. Validate GSI overloading opportunities

**When You Need a GSI:**

- Query by non-primary-key attribute
- Reverse lookup (e.g., Order → Customer when table is Customer → Order)
- Filter by attribute values (status, category, etc.)
- Sort by different attribute than primary SK

**GSI Design Principles:**

1. **Overload GSIs**

   - Don't create one GSI per pattern
   - Use generic names: GSI1PK, GSI1SK, GSI2PK, GSI2SK
   - Handle multiple entity types per GSI

2. **Dedicated Attributes**

   - Never reuse SK for GSI1SK
   - Use separate attributes per index
   - Easier to maintain and evolve

3. **Projection Strategy**
   - KEYS_ONLY: Minimal storage, requires table lookup
   - INCLUDE: Specify needed attributes
   - ALL: Copy all attributes (highest cost)

**GSI Overloading Example:**

```typescript
// Single GSI handles multiple patterns
// GSI1: GSI1PK + GSI1SK

// Pattern 1: Get Order by OrderId
{
  PK: 'CUSTOMER#C123',
  SK: 'ORDER#O456',
  GSI1PK: 'ORDER#O456',  // Direct lookup
  GSI1SK: 'ORDER#O456'
}

// Pattern 2: Get Orders by Status
{
  PK: 'CUSTOMER#C123',
  SK: 'ORDER#O456',
  GSI1PK: 'STATUS#SHIPPED',  // Group by status
  GSI1SK: '2020-01-15#O456'  // Sort by date
}

// Pattern 3: Get User by Email
{
  PK: 'USER#alice',
  SK: 'USER#alice',
  GSI1PK: 'EMAIL#alice@example.com',  // Unique lookup
  GSI1SK: 'EMAIL#alice@example.com'
}
```

**GSI Entity Chart Template:**

Create `docs/specs/jobs/<job>/entity-chart-gsi1.md`:

| Entity            | GSI1PK Pattern    | GSI1SK Pattern            | Access Pattern       | Notes          |
| ----------------- | ----------------- | ------------------------- | -------------------- | -------------- |
| Order (by ID)     | ORDER#\<orderId\> | ORDER#\<orderId\>         | Get Order by OrderId | Direct lookup  |
| Order (by Status) | STATUS#\<status\> | \<timestamp\>#\<orderId\> | Get Orders by Status | Sorted by date |
| User (by Email)   | EMAIL#\<email\>   | EMAIL#\<email\>           | Get User by Email    | Unique lookup  |

**Sparse Index Pattern:**

```typescript
// Only include items in GSI when certain condition met
interface Order {
  PK: "CUSTOMER#C123";
  SK: "ORDER#O456";
  Status: "PENDING";
  GSI1PK: "PENDING#ORDER"; // Only set when Status=PENDING
  GSI1SK: "2020-01-15"; // Only set when Status=PENDING
}

// When Order ships, remove GSI attributes:
UpdateExpression: "SET Status = :status REMOVE GSI1PK, GSI1SK";
```

**Access Patterns Table Update:**

| Entity | Access Pattern       | Index | Key Condition              | Parameters    | Notes         |
| ------ | -------------------- | ----- | -------------------------- | ------------- | ------------- |
| Order  | Get Order by OrderId | GSI1  | GSI1PK = ORDER#\<orderId\> | orderId       | Direct lookup |
| Order  | Get Orders by Status | GSI1  | GSI1PK = STATUS#\<status\> | status, limit | Paginated     |
| User   | Get User by Email    | GSI1  | GSI1PK = EMAIL#\<email\>   | email         | Unique        |

**Validation Checklist:**

- [ ] All unsatisfied patterns have GSI strategy
- [ ] GSIs overloaded where possible
- [ ] Separate attributes per index
- [ ] Eventual consistency acceptable for all GSI queries
- [ ] Projection strategy chosen for each GSI
- [ ] Entity charts created for each GSI
- [ ] Access patterns table updated with query details

**Common Mistakes:**

- Creating one GSI per pattern
- Reusing attributes across indexes
- Not considering sparse index opportunities

**Output:**

- `docs/specs/jobs/<job>/entity-chart-gsi1.md`
- `docs/specs/jobs/<job>/entity-chart-gsi2.md` (if needed)
- Updated access patterns table

---

### Step 5: Validate Against Anti-Patterns

**Duration:** 1 day

**Use Guide:** [Guide 7: Common Anti-Patterns](../guides/7-common-anti-patterns.md)

**Activities:**

1. Review design against anti-pattern checklist
2. Validate all access patterns efficient at scale
3. Confirm no relational database patterns
4. Check for proper separation of concerns
5. Get peer review from team

**Validation Checklist:**

**Access Patterns:**

- [ ] No Scan operations in application code
- [ ] All patterns use Query or GetItem
- [ ] Filter expressions not primary access mechanism
- [ ] Each pattern documented with specific parameters

**Key Design:**

- [ ] Generic key names (PK, SK, GSI1PK, etc.)
- [ ] No timestamps in keys unless client knows them
- [ ] Separate attributes per index (no reuse)
- [ ] Type attribute on every item
- [ ] Entity prefixes consistent

**Relationships:**

- [ ] Related items pre-joined in item collections
- [ ] No multiple serial requests for fetching relations
- [ ] Unbounded lists not in single items
- [ ] Denormalization accepted for performance

**Architecture:**

- [ ] Data access layer at boundary
- [ ] Indexing separate from application attributes
- [ ] No ORM usage
- [ ] Single table for related entities

**Cost & Performance:**

- [ ] GSI overloading maximized
- [ ] Storage optimization not prioritized over compute
- [ ] No full-table scans expected

**Red Flags:**

- ❌ "We'll just Scan and filter for now"
- ❌ "Let's normalize this to save storage"
- ❌ "We can add patterns later without planning"
- ❌ "One GSI per entity type is cleaner"

**Peer Review:**  
Schedule review with:

- [ ] Senior engineer familiar with DynamoDB
- [ ] Team member from Phase 1 discussions
- [ ] Someone not involved in modeling (fresh perspective)

**Output:**

- Validated, peer-reviewed data model
- Anti-pattern review document

---

## Phase 2 Deliverables

Upon completion of Phase 2, you should have:

1. **Primary Key Design**

   - `docs/specs/jobs/<job>/entity-chart-main.md`
   - Complete PK/SK patterns for all entities

2. **Relationship Documentation**

   - `docs/specs/jobs/<job>/relationship-decisions.md`
   - Strategy for each relationship in ERD

3. **Entity Schemas**

   - `docs/specs/jobs/<job>/entity-schemas.ts`
   - TypeScript interfaces for all entities

4. **Secondary Index Designs**

   - `docs/specs/jobs/<job>/entity-chart-gsi1.md`
   - `docs/specs/jobs/<job>/entity-chart-gsi2.md` (if needed)
   - Complete GSI key patterns

5. **Updated Access Patterns Table**

   - All patterns mapped to specific queries
   - Index and parameters documented

6. **Anti-Pattern Review**
   - Validation checklist completed
   - Peer review sign-off

## Common Pitfalls

### 1. Skipping relationship strategy decisions

**Problem:** Jumping straight to keys without thinking through relationship patterns  
**Solution:** Use Guide 5 to explicitly choose strategy for each relationship

### 2. Creating too many GSIs

**Problem:** One GSI per access pattern, exhausting 20 GSI limit  
**Solution:** Overload GSIs with generic attribute names

### 3. Reusing attributes across indexes

**Problem:** Using SK for both table and GSI sort key  
**Solution:** Dedicated attributes (SK, GSI1SK, GSI2SK)

### 4. Not including Type attribute

**Problem:** Can't filter by entity type in Scan/export operations  
**Solution:** Add Type: 'EntityName' to every item

### 5. Descriptive key names

**Problem:** Using CustomerId, OrderDate as attribute names  
**Solution:** Use PK, SK, and store semantics in values

## Moving to Phase 3

Before moving to Phase 3, ensure:

- [ ] All Phase 2 deliverables complete
- [ ] Entity charts validated
- [ ] Access patterns table completely filled out
- [ ] Anti-pattern review passed
- [ ] Peer review completed
- [ ] Team consensus on design

**Next Step:** [Phase 3: Implementation](phase-3.md)

---

## Additional Resources

- [Guide 3: Primary Key Design Guide](../guides/3-primary-key-design.md)
- [Guide 4: Entity Schema Guide](../guides/4-entity-schema.md)
- [Guide 5: Relationship Strategies Quick Reference](../guides/5-relationship-strategies.md)
- [Guide 6: Secondary Index Strategies](../guides/6-secondary-index-strategies.md)
- [Guide 7: Common Anti-Patterns Guide](../guides/7-common-anti-patterns.md)
- DynamoDB Book Chapter 7: "How to approach data modeling"
- DynamoDB Book Chapters 11-12: "Relationship strategies"
