# Guide 7: Common Anti-Patterns Guide

## When to Use This Guide

- [ ] Reviewing your data model design
- [ ] Troubleshooting performance issues
- [ ] Conducting code reviews
- [ ] Onboarding new team members to DynamoDB

## Purpose

Identify and avoid common mistakes when modeling data in DynamoDB. This guide compiles anti-patterns from the DynamoDB Book and real-world experience to help you recognize problematic designs before they become production issues.

## Critical Anti-Patterns

### Anti-Pattern 1: Using Scan for Application Queries

**What it is:**  
Using the Scan operation to retrieve data during user-facing requests.

**Why it's bad:**

- Scans read your entire table (or partition)
- Performance degrades as table grows
- Consumes massive amounts of read capacity
- Cannot provide consistent response times

**Example:**

```typescript
// ❌ BAD: Scanning to find orders by status
const orders = await dynamodb.scan({
  TableName: "AppTable",
  FilterExpression: "OrderStatus = :status",
  ExpressionAttributeValues: { ":status": "SHIPPED" },
});
```

**Correct approach:**  
Model your primary key or GSI to support the access pattern:

```typescript
// ✅ GOOD: Query GSI by status
const orders = await dynamodb.query({
  TableName: "AppTable",
  IndexName: "GSI1",
  KeyConditionExpression: "GSI1PK = :status",
  ExpressionAttributeValues: { ":status": "ORDER#SHIPPED" },
});
```

**When Scan is acceptable:**

- Tables with very small datasets (< 1000 items)
- Background ETL jobs
- One-time data migrations
- Sparse index patterns (intentional full index scan)

---

### Anti-Pattern 2: Treating DynamoDB Like a Relational Database

**What it is:**  
Normalizing data, creating one table per entity type, and making multiple serial requests to "join" data.

**Why it's bad:**

- Multiple network round-trips tank performance
- Cannot take advantage of Query API
- Loses DynamoDB's scalability benefits
- More expensive (paying for multiple requests)

**Example:**

```typescript
// ❌ BAD: Normalized design requiring multiple requests
// Table 1: Customers
// Table 2: Orders

async function getCustomerWithOrders(username: string) {
  // Request 1: Get customer
  const customer = await getItem("CustomersTable", username);

  // Request 2: Get orders (serial, not parallel!)
  const orders = await query("OrdersTable", customer.customerId);

  return { customer, orders };
}
```

**Correct approach:**  
Pre-join data using item collections:

```typescript
// ✅ GOOD: Single table design with item collections
// Single request fetches both
const result = await dynamodb.query({
  TableName: "AppTable",
  KeyConditionExpression: "PK = :pk",
  ExpressionAttributeValues: {
    ":pk": "CUSTOMER#alexdebrie",
  },
});
// Returns: Customer item + all Order items
```

**Key principle:**

> "You need to preassemble your data in the exact shape that is needed for a read operation."

---

### Anti-Pattern 3: Not Defining Access Patterns Up Front

**What it is:**  
Starting to build your table without documenting every access pattern.

**Why it's bad:**

- Impossible to design an efficient data model
- Forces expensive migrations later
- Results in suboptimal query patterns
- "You can't design your table until you know how you'll use your data"

**Example:**

```typescript
// ❌ BAD: Building table structure first
const table = new dynamodb.Table(this, "MyTable", {
  partitionKey: { name: "id", type: STRING },
  // ... then figuring out access patterns later
});
```

**Correct approach:**  
Document patterns before modeling:

```markdown
## Access Patterns (defined FIRST)

1. Get User by username
2. Get Orders for User (last 20, DESC by date)
3. Get Order by OrderId
4. Get Orders by Status

## Then design keys to satisfy these patterns
```

**Quote from the book:**

> "I cannot express strongly enough how important this step is. You can handle almost any data model with DynamoDB provided that you design for your access patterns up front."

---

### Anti-Pattern 4: Using Timestamps as Primary Keys

**What it is:**  
Using `CreatedAt` or other timestamps as part of your partition or sort key when the client won't know that timestamp at read time.

**Why it's bad:**

- Client won't have the timestamp value when querying
- Makes direct item lookup impossible
- Forces inefficient queries or scans

**Example:**

```typescript
// ❌ BAD: Timestamp in primary key
PK: USER#<username>
SK: <timestamp>  // Client won't know this!

// How do you fetch a specific user?
// You can't without scanning all items for that user.
```

**Correct approach:**  
Use stable, client-known identifiers:

```typescript
// ✅ GOOD: Stable identifier in primary key
PK: USER#<username>
SK: USER#<username>

// For time-based queries, use timestamp in sort key
// when that's the actual access pattern:
PK: SENSOR#<deviceId>
SK: <timestamp>  // OK if you're querying by time range
```

**Exception:**  
Timestamps are fine in sort keys when time-range queries are your actual access pattern (e.g., sensor readings, logs).

---

### Anti-Pattern 5: Over-Using Filter Expressions

**What it is:**  
Relying on FilterExpression to handle query logic instead of modeling it into your keys.

**Why it's bad:**

- Filter applied AFTER reading items (you pay for filtered-out data)
- Subject to 1MB limit before filtering
- Cannot save a bad data model
- Wastes read capacity

**Example:**

```typescript
// ❌ BAD: Using filter for core access pattern
const result = await dynamodb.query({
  TableName: "AppTable",
  KeyConditionExpression: "PK = :pk",
  FilterExpression: "OrderStatus = :status", // Applied AFTER read!
  ExpressionAttributeValues: {
    ":pk": "CUSTOMER#alice",
    ":status": "SHIPPED",
  },
});
```

**Correct approach:**  
Model status into your keys:

```typescript
// ✅ GOOD: Status in GSI partition key
GSI1PK: ORDER#<status>
GSI1SK: <timestamp>

// Query directly by status
const result = await dynamodb.query({
  IndexName: 'GSI1',
  KeyConditionExpression: 'GSI1PK = :status',
  ExpressionAttributeValues: {
    ':status': 'ORDER#SHIPPED'
  }
});
```

**When FilterExpression is acceptable:**

- Reducing response payload size (network optimization)
- Minor filtering after key-based query already narrowed results
- One-off admin queries

---

### Anti-Pattern 6: Creating One GSI Per Access Pattern

**What it is:**  
Creating a separate GSI for every access pattern that can't be satisfied by the main table.

**Why it's bad:**

- AWS limits you to 20 GSIs per table
- Increases operational complexity
- Higher costs (write capacity multiplied by number of GSIs)
- Misses GSI overloading opportunities

**Example:**

```typescript
// ❌ BAD: Separate GSI for each pattern
GSI1: Get Order by OrderId
GSI2: Get Order by TrackingNumber
GSI3: Get Orders by Status
GSI4: Get Orders by CustomerEmail
// ... runs out of GSIs quickly!
```

**Correct approach:**  
Overload GSIs to handle multiple patterns:

```typescript
// ✅ GOOD: Single GSI handles multiple patterns
GSI1PK: ORDER#<orderId>        // Pattern 1: Direct order lookup
GSI1PK: TRACKING#<number>      // Pattern 2: Tracking lookup
GSI1PK: EMAIL#<email>          // Pattern 3: Email lookup
GSI1PK: STATUS#<status>        // Pattern 4: Status filtering
```

**Key principle:**

> "New users often want to add a secondary index for each read pattern. This is overkill and will cost more. Instead, you can overload your secondary indexes."

---

### Anti-Pattern 7: Reusing Attributes Across Multiple Indexes

**What it is:**  
Using the same attribute (e.g., `SK`) for both your main table's sort key and a GSI's sort key.

**Why it's bad:**

- Creates tight coupling between indexes
- Makes future changes extremely difficult
- Limits flexibility for different entity types
- "You'll tie yourself in knots"

**Example:**

```typescript
// ❌ BAD: Reusing SK in GSI
Main Table:
  PK, SK

GSI1:
  GSI1PK, SK  // ← Reusing SK attribute!
```

**Correct approach:**  
Use separate attributes for each index:

```typescript
// ✅ GOOD: Dedicated attributes per index
Main Table:
  PK, SK

GSI1:
  GSI1PK, GSI1SK  // ← Dedicated attribute
```

**Why the extra storage cost is worth it:**

- Simpler modeling logic
- Easier migrations
- Independent evolution of access patterns
- Clearer intent in code

---

### Anti-Pattern 8: Using Descriptive Attribute Names for Keys

**What it is:**  
Using meaningful attribute names like `CustomerId`, `OrderDate` for primary keys instead of generic names.

**Why it's bad:**

- Doesn't support multiple entity types in one table
- Creates confusion (same attribute means different things per entity)
- Prevents key overloading
- Hard to evolve data model

**Example:**

```typescript
// ❌ BAD: Descriptive key names
{
  CustomerId: 'C123',      // Partition key
  OrderId: 'O456',         // Sort key
  // What about Users? Products? Sessions?
}
```

**Correct approach:**  
Use generic key names:

```typescript
// ✅ GOOD: Generic key names support any entity
{
  PK: 'CUSTOMER#C123',    // Can be anything
  SK: 'ORDER#O456',       // Can be anything
  Type: 'Order',          // Actual entity type
  CustomerId: 'C123',     // Application attribute
  OrderId: 'O456'         // Application attribute
}
```

---

### Anti-Pattern 9: Forgetting to Add Type Attribute

**What it is:**  
Not including a `Type` attribute on every item to identify the entity type.

**Why it's bad:**

- Difficult to distinguish entity types in console
- Hard to filter during ETL migrations
- Complicates analytics exports
- Makes debugging painful

**Example:**

```typescript
// ❌ BAD: No Type attribute
{
  PK: 'USER#alexdebrie',
  SK: 'USER#alexdebrie',
  Username: 'alexdebrie',
  Email: 'alex@example.com'
  // Hard to tell this is a User!
}
```

**Correct approach:**  
Always include Type:

```typescript
// ✅ GOOD: Explicit Type attribute
{
  PK: 'USER#alexdebrie',
  SK: 'USER#alexdebrie',
  Type: 'User',           // ← Clear entity type
  Username: 'alexdebrie',
  Email: 'alex@example.com'
}
```

**Bonus benefit:**  
Use in filter expressions during Scans:

```typescript
FilterExpression: 'Type = :type',
ExpressionAttributeValues: { ':type': 'Order' }
```

---

### Anti-Pattern 10: Storing Unbounded Lists in Items

**What it is:**  
Denormalizing unbounded one-to-many relationships into a single item as a List attribute.

**Why it's bad:**

- DynamoDB items limited to 400KB
- Will eventually fail as list grows
- All-or-nothing retrieval (can't paginate)
- Update complexity

**Example:**

```typescript
// ❌ BAD: Unbounded list in item
interface Customer {
  CustomerId: string;
  Orders: Array<Order>; // Could be thousands!
}
```

**Correct approach:**  
Use item collections for unbounded relationships:

```typescript
// ✅ GOOD: Separate items, same item collection
Customer item:
  PK: CUSTOMER#C123
  SK: CUSTOMER#C123

Order items:
  PK: CUSTOMER#C123
  SK: ORDER#O456
  PK: CUSTOMER#C123
  SK: ORDER#O789
  // ... can have unlimited orders
```

**When denormalization IS okay:**

- Relationship is bounded (e.g., max 20 addresses)
- Related items always accessed together
- Related items rarely change

---

### Anti-Pattern 11: Not Separating Indexing from Application Attributes

**What it is:**  
Mixing DynamoDB indexing logic with application business logic throughout your codebase.

**Why it's bad:**

- Tightly couples data access to business logic
- Makes testing difficult
- Hard to change data model
- Pollutes domain models with infrastructure concerns

**Example:**

```typescript
// ❌ BAD: DynamoDB logic everywhere
async function createOrder(order: Order) {
  return dynamodb.putItem({
    Item: {
      PK: { S: `CUSTOMER#${order.customerId}` },
      SK: { S: `ORDER#${order.orderId}` },
      GSI1PK: { S: `ORDER#${order.orderId}` },
      // ... indexing logic mixed with business logic
    },
  });
}
```

**Correct approach:**  
Implement data model at the boundary:

```typescript
// ✅ GOOD: Separation of concerns
// Domain layer
interface Order {
  orderId: string;
  customerId: string;
  amount: number;
}

// Data access layer (boundary)
class OrderRepository {
  async save(order: Order): Promise<void> {
    const item = this.toDynamoDBItem(order);
    await dynamodb.putItem({ Item: item });
  }

  private toDynamoDBItem(order: Order) {
    return {
      PK: { S: `CUSTOMER#${order.customerId}` },
      SK: { S: `ORDER#${order.orderId}` },
      // ... all indexing logic contained here
    };
  }
}
```

---

### Anti-Pattern 12: Using ORM/ODMs

**What it is:**  
Using Object-Relational Mappers or Object-Document Mappers that abstract away DynamoDB specifics.

**Why it's bad:**

- ORMs designed for relational thinking
- Hide the very things you need to understand
- Can't leverage DynamoDB-specific patterns
- Performance anti-patterns baked in

**Example:**

```typescript
// ❌ BAD: Generic ORM
@Entity()
class Order {
  @PrimaryKey()
  id: string;

  @ManyToOne(() => Customer)
  customer: Customer; // ORM tries to "join" these
}
```

**Correct approach:**  
Use DynamoDB-aware helpers (not magic ORMs):

```typescript
// ✅ GOOD: Lightweight DynamoDB tooling
import { Entity } from "dynamodb-toolbox";

const OrderEntity = new Entity({
  name: "Order",
  attributes: {
    PK: { partitionKey: true },
    SK: { sortKey: true },
    orderId: { type: "string" },
    // ... explicit about DynamoDB structure
  },
});
```

**Acceptable tools:**

- AWS SDK v3 (low-level)
- DynamoDB Toolbox (DynamoDB-specific helper)
- Custom thin wrappers around SDK

---

### Anti-Pattern 13: Assuming Access Patterns Can't Change

**What it is:**  
Believing that because you model for specific patterns, you can never add new ones.

**Why it's bad:**

- Leads to avoidance of DynamoDB
- Misses the point: you CAN migrate, you just plan differently
- Self-fulfilling prophecy of inflexibility

**Reality:**  
Access patterns DO evolve, and DynamoDB handles it:

- Many changes are additive (new GSI, new attributes)
- Migration strategies exist (lazy writes, ETL processes)
- Single-table design is flexible once you understand it

**Correct mindset:**

> "You can evolve your DynamoDB data model just like you can evolve with other databases. The same general DynamoDB principles apply—you must know your new access patterns before modeling them out—but many changes are additive."

---

### Anti-Pattern 14: Optimizing for Storage Over Compute

**What it is:**  
Normalizing data to avoid duplication, prioritizing storage savings over query performance.

**Why it's bad:**

- Storage is cheap (\$0.10/GB/month)
- Compute is expensive (multiple requests, joins)
- Defeats the purpose of using DynamoDB
- Forces relational patterns on NoSQL database

**Example:**

```typescript
// ❌ BAD: Normalized to save storage
Table 1: Customers
Table 2: CustomerAddresses (to avoid storing addresses on Customer)

// Now requires 2 requests to get Customer + Addresses
```

**Correct approach:**  
Denormalize for performance:

```typescript
// ✅ GOOD: Duplicate data to optimize queries
interface Customer {
  Type: "Customer";
  CustomerId: string;
  Addresses: Address[]; // Denormalized (bounded)
}

// Single request gets everything
```

**Key principle:**

> "Storage is cheap and plentiful... it makes sense to optimize for compute by denormalizing your data."

---

## Validation Checklist

Before deploying your DynamoDB data model, verify:

**Access Patterns:**

- [ ] All access patterns documented BEFORE modeling
- [ ] No reliance on Scan for application queries
- [ ] Each pattern satisfied by primary key or GSI (not Filter)

**Key Design:**

- [ ] Generic key names (PK, SK, GSI1PK, etc.)
- [ ] No timestamps in keys unless client knows them
- [ ] Separate attributes per index (no reuse)
- [ ] Type attribute on every item

**Relationships:**

- [ ] Related items pre-joined in item collections
- [ ] No multiple serial requests for relationships
- [ ] Unbounded lists not stored in single items

**Architecture:**

- [ ] Data access layer at application boundary
- [ ] Indexing attributes separated from application attributes
- [ ] No relational-style ORM in use

**Cost & Performance:**

- [ ] GSI overloading used (not one GSI per pattern)
- [ ] Denormalization accepted for query performance
- [ ] Single-table design for related entities

## Summary Table

| Anti-Pattern                  | Impact                               | Correct Approach                |
| ----------------------------- | ------------------------------------ | ------------------------------- |
| Using Scan for queries        | Performance degrades at scale        | Model keys/GSIs for patterns    |
| Treating like RDBMS           | Multiple requests, slow              | Single-table, item collections  |
| No defined patterns           | Bad data model, expensive migrations | Document patterns first         |
| Timestamps in keys            | Can't query by identifier            | Use stable, client-known IDs    |
| Over-using filters            | Wasted capacity, 1MB limits          | Model filters into keys         |
| One GSI per pattern           | Runs out of GSIs, expensive          | Overload GSIs                   |
| Reusing attributes            | Tight coupling, hard to change       | Dedicated attributes per index  |
| Descriptive key names         | Can't support multiple entities      | Generic PK/SK names             |
| No Type attribute             | Hard to debug, filter, export        | Always include Type             |
| Unbounded lists in items      | Hits 400KB limit                     | Use item collections            |
| Mixed indexing/business logic | Tight coupling, hard to test         | Data access at boundary         |
| Using ORMs                    | Hides DynamoDB patterns              | Use SDK or DynamoDB-aware tools |
| Fear of migrations            | Avoids DynamoDB unnecessarily        | Understand migration strategies |
| Storage over compute          | Slow queries                         | Denormalize for performance     |

---

That's the complete set of seven guides! Here's a summary of what we created:

## Complete Guide Set

1. **ERD Creation Guide** - Foundation for entities and relationships
2. **Access Patterns Definition Guide** - Bridging requirements to data model
3. **Primary Key Design Guide** - Core table structure decisions
4. **Entity Schema Guide** - TypeScript types for non-key attributes
5. **Relationship Strategies Quick Reference** - Pattern library for relationships
6. **Secondary Index Strategies** - When and how to use GSIs
7. **Common Anti-Patterns Guide** - What to avoid

All guides are designed to be:

- AI-model friendly (instructional, decision-tree format)
- DynamoDB Book-aligned (references best practices)
- Practical (includes code examples for your TypeScript/CDK stack)
- Progressive (build on each other in logical order)
