# Faux-SQL DynamoDB Modeling: Complete Guide

## Table of Contents

1. [Overview & Introduction](#1-overview--introduction)
2. [Quick Start](#2-quick-start)
3. [Phase 1: Experience & Domain Design](#3-phase-1-experience--domain-design)
4. [Phase 2: Data Modeling](#4-phase-2-data-modeling)
5. [Phase 3: Implementation](#5-phase-3-implementation)
6. [Advanced Topics](#6-advanced-topics)
7. [Migration & Evolution](#7-migration--evolution)
8. [Complete Examples](#8-complete-examples)
9. [Reference](#9-reference)

---

## 1. Overview & Introduction

### 1.1 What is Faux-SQL DynamoDB Modeling?

Faux-SQL DynamoDB modeling is an approach where you use DynamoDB but structure your data **like a relational database**. Instead of cramming multiple entity types into a single table with overloaded keys (single-table design), you create **separate tables for each entity type**, similar to traditional SQL databases.

**Core Principles:**

- **Multiple tables**: One table per entity type (Customers, Orders, Products, etc.)
- **Descriptive keys**: Meaningful names (CustomerId, OrderId) instead of generic (PK, SK)
- **Normalized data**: Follow relational normalization principles
- **Simple GSIs**: One index per access pattern, no overloading
- **Multiple requests**: Accept 2-3 serial requests for related data

**Visual Comparison:**

```
Single-Table Design:
└── AppTable
    ├── CUSTOMER#123 | CUSTOMER#123 (Customer)
    ├── CUSTOMER#123 | ORDER#456 (Order)
    └── ORDER#456 | ITEM#789 (OrderItem)
    → 1 request fetches all

Faux-SQL Design:
├── CustomersTable (PK: CustomerId)
├── OrdersTable (PK: OrderId, GSI: CustomerId)
└── OrderItemsTable (PK: OrderId, SK: ItemId)
    → 2-3 requests fetch all
```

---

### 1.2 When to Use Faux-SQL

**Use Faux-SQL when:**

#### A. Developer Agility Over Performance

**Context**: "In new applications where developer agility is more important than application performance"

When building new applications, especially at startups, you often:

- Don't know your exact access patterns yet
- Need to iterate quickly on your data model
- May pivot the entire application direction
- Can tolerate 100ms response times instead of 10ms

**Quote from the book**: "You may decide that the performance characteristics of a single-table design are not worth the loss of flexibility and more difficult analytics"

#### B. Easier Analytics and Reporting

Single-table design makes analytics difficult because:

- Data is denormalized and "twisted into a pretzel"
- You need to "unwind" the table to make it analytics-friendly
- Multiple entity types are mixed together

With Faux-SQL:

- Each entity is in its own table (easy to export)
- Data is normalized (standard SQL queries work)
- Business analysts can understand the structure

**Quote**: "A well-optimized single-table DynamoDB layout looks more like machine code than a simple spreadsheet" — Forrest Brazeal

#### C. Compatibility with Serverless Architecture

DynamoDB works perfectly with serverless because:

- No connection pooling issues (HTTP-based API)
- Pay-per-use pricing model
- Seamless integration with AWS Lambda
- Infrastructure-as-code friendly

Traditional relational databases have problems with serverless due to connection management.

#### D. AI/LLM Assistant Compatibility

While not explicitly stated in the book, Faux-SQL is dramatically easier for AI coding assistants because:

- **Standard CRUD patterns**: AI models are trained on millions of SQL examples
- **Descriptive naming**: Clear attribute names help with code generation
- **No complex overloading**: No need to understand partition key patterns like `CUSTOMER#<Id>#ORDER#<OrderId>`
- **Familiar patterns**: One-to-many relationships work like foreign keys

**Use Single-Table when:**

✅ Performance is critical (sub-10ms response times)
✅ High throughput (>10K requests/second)
✅ Access patterns are well-defined and stable
✅ Team has DynamoDB expertise
✅ Cost optimization at scale is priority

**Decision Matrix:**

| Factor                         | Use Faux-SQL                         | Use Single-Table                        |
| ------------------------------ | ------------------------------------ | --------------------------------------- |
| **Application Stage**          | MVP, early-stage, uncertain patterns | Production, known patterns              |
| **Response Time Needs**        | 50-200ms acceptable                  | Sub-10ms required                       |
| **Access Pattern Knowledge**   | Patterns still evolving              | Patterns well-defined                   |
| **Team Experience**            | Junior devs, new to DynamoDB         | Experienced DynamoDB modelers           |
| **Analytics Requirements**     | Frequent ad-hoc queries needed       | OLTP focus, separate analytics pipeline |
| **Scale Requirements**         | < 10K requests/sec                   | Millions of requests/sec                |
| **GraphQL Usage**              | Yes                                  | No (REST/RPC)                           |
| **Development Speed Priority** | High - need to ship fast             | Lower - performance critical            |

### Explicit Guidance from the Book

**When NOT to use single-table design** (i.e., when Faux-SQL is appropriate):

> "There are two occasions where [single-table benefits] don't outweigh the costs:
>
> - In new applications where developer agility is more important than application performance
> - In applications using GraphQL"

**Important caveat**:

> "But first I want to emphasize that these are exceptions, not general guidance. When modeling with DynamoDB, you should be following best practices... And even if you opt into a multi-table design, you should understand single-table design to know why it's not a good fit for your specific application."

---

### 1.3 Trade-offs & Considerations

**✅ Advantages of Faux-SQL:**

- **Fast development**: Familiar patterns, quick iteration
- **Easy to understand**: SQL-like structure, clear naming
- **Flexible**: Easy to add entities and access patterns
- **Analytics-friendly**: Normalized data, easy exports
- **AI-compatible**: Standard patterns for code generation
- **Lower cognitive load**: No complex key overloading

**⚠️ Disadvantages of Faux-SQL:**

- **Higher latency**: Multiple requests (50-100ms vs 10ms)
- **More expensive at scale**: More RCUs/WCUs consumed
- **No atomic transactions across tables**: Can't update Customer + Order atomically
- **Potential hot partitions**: If not careful with key design
- **Less efficient**: Can't pre-join related data

**Performance Comparison:**

```
Single-Table: 1 request → 10ms
Faux-SQL: 3 requests → 50-100ms

At 1K req/sec:
Single-Table: ~$10/month
Faux-SQL: ~$30/month

At 100K req/sec:
Single-Table: ~$1,000/month
Faux-SQL: ~$3,000/month
```

**When Trade-offs Are Acceptable:**

- MVP/early-stage (speed > performance)
- < 10K requests/second
- Analytics requirements high
- Team learning DynamoDB
- Using GraphQL

---

### 1.4 Migration Path (Faux-SQL → Single-Table)

**When to Migrate:**

- Traffic exceeds 10K requests/second
- Latency becomes user-facing issue
- Cost optimization becomes priority
- Access patterns have stabilized

**Migration Strategy:**

```
Phase 1: Design single-table schema
├─ Map Faux-SQL tables to single table
├─ Design GSIs for access patterns
└─ Document transformations

Phase 2: Dual-write implementation
├─ Write to both old and new tables
├─ Validate data parity
└─ Monitor for errors

Phase 3: Switch reads gradually
├─ Use feature flags
├─ Route % of traffic to new table
└─ Monitor performance

Phase 4: Deprecate old tables
├─ Stop dual-writes
├─ Archive old data
└─ Delete old tables
```

**Timeline**: Typically 2-4 weeks for complete migration

See [Section 7.3](#73-migrating-to-single-table-design) for detailed migration guide.

---

## 2. Quick Start

### 2.1 Prerequisites

Before starting with Faux-SQL modeling:

- [ ] Basic understanding of DynamoDB concepts (tables, items, keys)
- [ ] Familiarity with TypeScript/JavaScript
- [ ] AWS account with DynamoDB access
- [ ] Node.js and AWS CDK installed

**Recommended Reading:**
- AWS DynamoDB Developer Guide (basics)
- Understanding of NoSQL vs SQL differences

---

### 2.2 Tools Required

**Development:**
- **AWS CDK** - Infrastructure as Code for table definitions
- **AWS SDK v3** - DynamoDB Document Client for data operations
- **TypeScript** - Type-safe entity definitions

**Optional:**
- **NoSQL Workbench** - Visual table design and testing
- **PlantUML** - ERD diagrams
- **DynamoDB Local** - Local development environment

---

### 2.3 Your First Faux-SQL Model (5-minute example)

Let's build a simple blog application with Posts and Comments.

**Step 1: Define Entities**

```typescript
// entities/post.ts
interface Post {
  PostId: string;
  Title: string;
  Content: string;
  AuthorId: string;
  CreatedAt: string;
}

// entities/comment.ts
interface Comment {
  CommentId: string;
  PostId: string; // Foreign key
  AuthorId: string;
  Content: string;
  CreatedAt: string;
}
```

**Step 2: Create Tables (CDK)**

```typescript
// infrastructure/tables.ts
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";

// Posts Table
const postsTable = new dynamodb.Table(this, "PostsTable", {
  tableName: "Posts",
  partitionKey: { name: "PostId", type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
});

// Add GSI for querying by author
postsTable.addGlobalSecondaryIndex({
  indexName: "AuthorIndex",
  partitionKey: { name: "AuthorId", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "CreatedAt", type: dynamodb.AttributeType.STRING },
});

// Comments Table
const commentsTable = new dynamodb.Table(this, "CommentsTable", {
  tableName: "Comments",
  partitionKey: { name: "PostId", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "CommentId", type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
});
```

**Step 3: Implement Data Access**

```typescript
// repositories/posts.ts
import { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand } from "@aws-sdk/lib-dynamodb";

export async function createPost(post: Post): Promise<Post> {
  await docClient.send(
    new PutCommand({
      TableName: "Posts",
      Item: post,
    })
  );
  return post;
}

export async function getPost(postId: string): Promise<Post | null> {
  const result = await docClient.send(
    new GetCommand({
      TableName: "Posts",
      Key: { PostId: postId },
    })
  );
  return result.Item as Post | null;
}

export async function getPostsByAuthor(authorId: string): Promise<Post[]> {
  const result = await docClient.send(
    new QueryCommand({
      TableName: "Posts",
      IndexName: "AuthorIndex",
      KeyConditionExpression: "AuthorId = :authorId",
      ExpressionAttributeValues: { ":authorId": authorId },
    })
  );
  return result.Items as Post[];
}
```

**Step 4: Query Related Data**

```typescript
// Get post with comments (2 requests)
export async function getPostWithComments(postId: string) {
  // Request 1: Get post
  const post = await getPost(postId);
  
  // Request 2: Get comments for post
  const comments = await docClient.send(
    new QueryCommand({
      TableName: "Comments",
      KeyConditionExpression: "PostId = :postId",
      ExpressionAttributeValues: { ":postId": postId },
    })
  );

  return {
    post,
    comments: comments.Items as Comment[],
  };
}
```

**That's it!** You've created a working Faux-SQL DynamoDB model in 5 minutes.

**Key Takeaways:**
- ✅ Two separate tables (Posts, Comments)
- ✅ Descriptive key names (PostId, CommentId)
- ✅ Simple GSI for alternate access pattern
- ✅ Foreign key relationship (PostId in Comments)
- ✅ 2 requests to fetch related data

---

## 3. Phase 1: Experience & Domain Design

**Objectives:**

By the end of Phase 1, you should have:
- [ ] Entity list scoped to current job/sprint
- [ ] Entity-Relationship Diagram (ERD)
- [ ] Access patterns documented for current features
- [ ] Understanding of relationships between entities

**Key Difference from Single-Table:**
- Don't need ALL access patterns upfront
- Focus on current job/sprint requirements
- Can add patterns incrementally later

---

### 3.1 Step 1: Understand Your Application

**What:** Gather requirements from job cards, mockups, and user stories

**Why (Faux-SQL context):** Unlike single-table design which requires comprehensive upfront planning, Faux-SQL allows incremental discovery. Focus on understanding the current job/sprint rather than the entire application.

**How:**

1. **Review job card** - Identify core business objects
2. **Analyze mockups** - What data appears on each screen?
3. **Read acceptance criteria** - What operations are required?
4. **Identify entities** - List the nouns (Customer, Order, Product)
5. **Scope to current job** - Don't model future features yet

**Output:** Entity list scoped to current job/sprint

**Example: "Search by Category" Job**

```markdown
## Entities Identified:
- Merchant (business listings)
- Category (Repair, Refill, Recycling, Donate)
- Location (GPS coordinates, address)
- Review (ratings, comments)

## Current Job Workflows:
- Consumer searches by category + filters
- Consumer views search results (map + list)
- Consumer opens listing detail page

## Future Jobs (Not Modeling Yet):
- Merchant creates/edits listing
- Consumer saves favorites
```

**Common Pitfalls:**

❌ **Trying to model entire application**: Leads to analysis paralysis
✅ **Solution**: Focus only on current job/sprint entities

❌ **Skipping mockup analysis**: Missing key entities
✅ **Solution**: Walk through each screen systematically

---

### 3.2 Step 2: Create Entity-Relationship Diagram (ERD)

**What:** Visual representation of entities and their relationships

**Why (Faux-SQL context):** ERD translates **directly** to table structure. Each entity box = one DynamoDB table. No need to "twist" the model for single-table patterns.

**How:**

1. **Document each entity** with key attributes
2. **Identify relationships** between entities
3. **Classify relationship types** (1:1, 1:N, M:N)
4. **Create visual ERD** using PlantUML or Mermaid
5. **Keep it simple** - Focus on relationships, not all attributes

**Output:** `docs/project/specs/jobs/<job-id>/erd.puml`

**Example ERD (E-commerce):**

```
Customer ──────< Orders ──────< OrderItems
│
└──────< Addresses

Relationships:
- Customer → Orders (1:N)
- Order → OrderItems (1:N)
- Customer → Addresses (1:N)
```

**Example ERD (SMW Search):**

```
Merchant ──────< Categories
│
├──────── Location (1:1)
│
└──────< Reviews

Relationships:
- Merchant → Categories (M:N)
- Merchant → Location (1:1, denormalized)
- Merchant → Reviews (1:N)
```

**Common Pitfalls:**

❌ **Too many attributes**: ERD cluttered with every field
✅ **Solution**: Show only key identifiers and relationships

❌ **Missing cardinality**: Unclear if 1:1, 1:N, or M:N
✅ **Solution**: Always mark relationship type explicitly

---

### 3.3 Step 3: Define Access Patterns

**What:** Document how data will be queried

**Why (Faux-SQL context):** You don't need ALL patterns upfront. Document current job patterns and add more incrementally as features require them.

**How:**

1. **UI-centric approach** - For each screen, what data is needed?
2. **List access patterns** for current job
3. **Document in entity files** (not centralized table)
4. **Track status** - Implemented vs. Planned vs. Unknown
5. **Add GSIs as needed** - Can evolve over time

**Output:** Access patterns documented in entity files

**Example: Merchants Entity**

```markdown
## MerchantsTable

### Implemented Access Patterns (Current Job)
✅ Get merchant by MerchantId (direct lookup)
✅ Search merchants by category within radius (CategoryIndex GSI)

### Planned Access Patterns (Known Future Jobs)
🔲 Get merchant by email (merchant portal login)
🔲 Filter merchants by verification status (admin dashboard)

### Potential Future Patterns (Unknown)
? Analytics: Merchants by signup date range
? Admin: View all merchants across categories
```

**Common Pitfalls:**

❌ **Trying to define all patterns upfront**: Delays development
✅ **Solution**: Start with current job, add patterns incrementally

❌ **Vague pattern descriptions**: "Get merchant data"
✅ **Solution**: Be specific - "Get merchant by MerchantId for detail page"

---

## 4. Phase 2: Data Modeling

**Objectives:**

By the end of Phase 2, you should have:
- [ ] Primary key design for each table
- [ ] Entity-key tables documenting key patterns
- [ ] GSI definitions for alternate access patterns
- [ ] Relationship strategies documented

**Key Difference from Single-Table:**
- Descriptive key names (not PK/SK)
- One table per entity
- Simple GSIs (no overloading)

---

### 4.1 Step 4: Design Primary Keys

**What:** Structure table keys for efficient access

**Why (Faux-SQL context):** Use descriptive, meaningful key names. Simple keys for single-item lookups, composite keys for hierarchies or one-to-many relationships within same table.

**How:**

1. **Choose key type** - Simple (PK only) or Composite (PK + SK)
2. **Use descriptive names** - CustomerId, OrderId (not PK, SK)
3. **Document in entity-key table** format
4. **Add GSIs** for alternate lookups

**Simple Key Pattern:**

```typescript
// MerchantsTable
{
  MerchantId: "M123",  // Partition Key
  Name: "Green Repair Shop",
  CategoryId: "REPAIR"
}

**Composite Key Pattern:**

```typescript
// ReviewsTable (hierarchical)
{
  MerchantId: "M123",      // Partition Key
  ReviewId: "R456",        // Sort Key
  Rating: 5,
  Comment: "Great service!"
}

// Query all reviews for a merchant
KeyConditionExpression: "MerchantId = :merchantId"
```

**Entity-Key Table Template:**

```markdown
## MerchantsTable

| Attribute | Key Type | Notes |
|-----------|----------|-------|
| MerchantId | HASH (PK) | Primary identifier |

### GSI: CategoryIndex
| Attribute | Key Type | Notes |
|-----------|----------|-------|
| CategoryId | HASH | For category browsing |
| MerchantId | RANGE | Sort by merchant |
```

**Output:** Entity-key tables for each table

**Example: E-commerce Tables**

```markdown
## CustomersTable
- PK: CustomerId

## OrdersTable
- PK: OrderId
- GSI1: CustomerId (for "my orders")

## OrderItemsTable
- PK: OrderId
- SK: ItemId
```

**Common Pitfalls:**

❌ **Using generic names**: PK, SK, GSI1PK
✅ **Solution**: Use descriptive names - CustomerId, OrderId

❌ **Composite key when simple suffices**: Overcomplicating
✅ **Solution**: Use composite only for hierarchies or 1:N in same table

---

### 4.2 Step 5: Model Relationships

**What:** Choose strategies for connecting entities

**Why (Faux-SQL context):** Relationships use foreign keys and multiple requests, similar to SQL. Three main strategies: foreign key attributes, denormalization (bounded), or composite keys (hierarchies).

**How:**

1. **Review each relationship** in ERD
2. **Classify type** (1:1, 1:N, M:N)
3. **Choose strategy** based on pattern
4. **Document decision** with rationale

**Strategy 1: Foreign Key Attributes (Most Common)**

```typescript
// OrdersTable
{
  OrderId: "O123",
  CustomerId: "C456",  // Foreign key
  Total: 99.99
}

// Query: Get orders for customer
// Use GSI on CustomerId
QueryCommand({
  TableName: "Orders",
  IndexName: "CustomerIndex",
  KeyConditionExpression: "CustomerId = :customerId"
})
```

**Strategy 2: Denormalization (Bounded 1:N)**

```typescript
// CustomerTable (with embedded addresses)
{
  CustomerId: "C456",
  Name: "Alice",
  Addresses: [  // Bounded list (<5 addresses)
    { Street: "123 Main St", City: "Seattle" },
    { Street: "456 Oak Ave", City: "Portland" }
  ]
}
```

**Strategy 3: Composite Keys (Hierarchical 1:N)**

```typescript
// ReviewsTable
{
  MerchantId: "M123",  // PK
  ReviewId: "R456",    // SK
  Rating: 5
}

// Query all reviews for merchant
KeyConditionExpression: "MerchantId = :merchantId"
```

**Output:** Relationship decisions documented

**Example Documentation:**

```markdown
## Merchant → Reviews (1:N, Unbounded)

**Strategy:** Composite key (MerchantId + ReviewId)

**Rationale:**
- Unbounded (could have 1000s of reviews)
- Always queried together ("show merchant with reviews")
- Reviews don't need independent access

**Implementation:**
- ReviewsTable: PK=MerchantId, SK=ReviewId
- Query: KeyConditionExpression="MerchantId = :id"
```

**Common Pitfalls:**

❌ **Unbounded lists as attributes**: Hits 400KB item limit
✅ **Solution**: Use separate table with composite key or foreign key + GSI

❌ **No documentation**: Unclear why strategy chosen
✅ **Solution**: Document rationale for each relationship

---

### 4.3 Step 6: Design Secondary Indexes (GSIs)

**What:** Add alternate access patterns via GSIs

**Why (Faux-SQL context):** Simple, straightforward GSIs - one per access pattern. No overloading or complex patterns needed.

**How:**

1. **Identify alternate lookups** - What queries can't use primary key?
2. **Create GSI per pattern** - Descriptive names (CategoryIndex, EmailIndex)
3. **Use descriptive attribute names** - Not GSI1PK/GSI1SK
4. **Document in entity files**

**Pattern: Unique Alternate Key**

```typescript
// CustomersTable
// Primary: CustomerId
// GSI: EmailIndex (for login by email)

{
  CustomerId: "C123",
  Email: "alice@example.com",  // GSI partition key
  Name: "Alice"
}

// Query by email
QueryCommand({
  TableName: "Customers",
  IndexName: "EmailIndex",
  KeyConditionExpression: "Email = :email"
})
```

**Pattern: Foreign Key Lookup**

```typescript
// OrdersTable
// Primary: OrderId
// GSI: CustomerIndex (for "my orders")

{
  OrderId: "O456",
  CustomerId: "C123",  // GSI partition key
  OrderDate: "2024-01-15",  // GSI sort key
  Total: 99.99
}

// Query orders by customer
QueryCommand({
  TableName: "Orders",
  IndexName: "CustomerIndex",
  KeyConditionExpression: "CustomerId = :customerId"
})
```

**Output:** GSI definitions in entity files

**Example:**

```markdown
## OrdersTable

### Primary Key
- OrderId (HASH)

### GSI: CustomerIndex
- CustomerId (HASH)
- OrderDate (RANGE)
- Purpose: Get orders for customer, sorted by date
- Access Pattern: "My Orders" page
```

**Common Pitfalls:**

❌ **GSI overloading**: Trying to use one GSI for multiple patterns
✅ **Solution**: Create separate GSI per pattern (Faux-SQL simplicity)

❌ **Generic GSI names**: GSI1, GSI2
✅ **Solution**: Descriptive names - CustomerIndex, CategoryIndex

---

## 5. Phase 3: Implementation

**Objectives:**

By the end of Phase 3, you should have:
- [ ] CDK table definitions deployed
- [ ] Data access layer implemented
- [ ] CRUD operations tested
- [ ] Query patterns validated

**Key Difference from Single-Table:**
- Multiple table definitions (one per entity)
- Simpler data access code (no key overloading)
- Standard CRUD patterns

---

### 5.1 Step 7: Create Infrastructure (CDK)

**What:** Define tables in Infrastructure as Code

**Why (Faux-SQL context):** Each entity gets its own table definition. Simple, straightforward CDK code with descriptive names.

**How:**

1. **Create CDK stack** for data layer
2. **Define table per entity** with descriptive names
3. **Add GSIs** as needed
4. **Use PAY_PER_REQUEST** billing for flexibility
5. **Deploy to AWS**

**Output:** CDK stack deployed to AWS

**Example: E-commerce Tables**

```typescript
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import * as cdk from "aws-cdk-lib";

export class DataStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    // Customers Table
    const customersTable = new dynamodb.Table(this, "CustomersTable", {
      tableName: "Customers",
      partitionKey: { name: "CustomerId", type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
    });

    customersTable.addGlobalSecondaryIndex({
      indexName: "EmailIndex",
      partitionKey: { name: "Email", type: dynamodb.AttributeType.STRING },
    });

    // Orders Table
    const ordersTable = new dynamodb.Table(this, "OrdersTable", {
      tableName: "Orders",
      partitionKey: { name: "OrderId", type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
    });

    ordersTable.addGlobalSecondaryIndex({
      indexName: "CustomerIndex",
      partitionKey: { name: "CustomerId", type: dynamodb.AttributeType.STRING },
      sortKey: { name: "OrderDate", type: dynamodb.AttributeType.STRING },
    });

    // OrderItems Table
    const orderItemsTable = new dynamodb.Table(this, "OrderItemsTable", {
      tableName: "OrderItems",
      partitionKey: { name: "OrderId", type: dynamodb.AttributeType.STRING },
      sortKey: { name: "ItemId", type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
    });
  }
}
```

**Common Pitfalls:**

❌ **Provisioned capacity**: Hard to predict, causes throttling
✅ **Solution**: Use PAY_PER_REQUEST for flexibility

❌ **Missing GSIs**: Forgetting alternate access patterns
✅ **Solution**: Review entity files before deploying

---

### 5.2 Step 8: Implement Data Access Layer

**What:** CRUD operations and query patterns

**Why (Faux-SQL context):** Simple, standard patterns. Each table gets its own repository/service with descriptive method names.

**How:**

1. **Create TypeScript interfaces** for entities
2. **Implement repository per table**
3. **Use Document Client** for simpler code
4. **Handle multiple requests** for related data
5. **Add error handling**

**Output:** Data access modules

**Example: Customers Repository**

```typescript
import { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand } from "@aws-sdk/lib-dynamodb";

export interface Customer {
  CustomerId: string;
  Email: string;
  Name: string;
  CreatedAt: string;
}

export async function createCustomer(customer: Customer): Promise<Customer> {
  await docClient.send(
    new PutCommand({
      TableName: "Customers",
      Item: customer,
    })
  );
  return customer;
}

export async function getCustomer(customerId: string): Promise<Customer | null> {
  const result = await docClient.send(
    new GetCommand({
      TableName: "Customers",
      Key: { CustomerId: customerId },
    })
  );
  return result.Item as Customer | null;
}

export async function getCustomerByEmail(email: string): Promise<Customer | null> {
  const result = await docClient.send(
    new QueryCommand({
      TableName: "Customers",
      IndexName: "EmailIndex",
      KeyConditionExpression: "Email = :email",
      ExpressionAttributeValues: { ":email": email },
    })
  );
  return result.Items?.[0] as Customer | null;
}
```

**Example: Fetching Related Data**

```typescript
// Get customer with their orders (2 requests)
export async function getCustomerWithOrders(customerId: string) {
  // Request 1: Get customer
  const customer = await getCustomer(customerId);
  
  // Request 2: Get orders for customer
  const orders = await docClient.send(
    new QueryCommand({
      TableName: "Orders",
      IndexName: "CustomerIndex",
      KeyConditionExpression: "CustomerId = :customerId",
      ExpressionAttributeValues: { ":customerId": customerId },
    })
  );

  return {
    customer,
    orders: orders.Items as Order[],
  };
}
```

**Common Pitfalls:**

❌ **Not using Document Client**: Dealing with AttributeValue types
✅ **Solution**: Use DynamoDBDocumentClient for simpler code

❌ **Serial requests in loops**: N+1 query problem
✅ **Solution**: Use Promise.all() for parallel requests when possible

---

### 5.3 Step 9: Testing & Debugging

**What:** Validate implementation works correctly

**Why (Faux-SQL context):** Test each table independently, then test cross-table queries.

**How:**

1. **Unit tests** for each repository method
2. **Integration tests** for cross-table queries
3. **Debugging scripts** for data inspection
4. **Validate access patterns** match requirements

**Output:** Test suite and debugging tools

**Example: Unit Test**

```typescript
describe("CustomersRepository", () => {
  it("should create and retrieve customer", async () => {
    const customer: Customer = {
      CustomerId: "C123",
      Email: "test@example.com",
      Name: "Test User",
      CreatedAt: new Date().toISOString(),
    };

    await createCustomer(customer);
    const retrieved = await getCustomer("C123");

    expect(retrieved).toEqual(customer);
  });

  it("should find customer by email", async () => {
    const customer = await getCustomerByEmail("test@example.com");
    expect(customer?.Email).toBe("test@example.com");
  });
});
```

**Example: Debugging Script**

```typescript
// scripts/inspect-customer.ts
async function inspectCustomer(customerId: string) {
  console.log("=== Customer ===");
  const customer = await getCustomer(customerId);
  console.log(JSON.stringify(customer, null, 2));

  console.log("\n=== Orders ===");
  const orders = await getOrdersByCustomer(customerId);
  console.log(JSON.stringify(orders, null, 2));
}

inspectCustomer(process.argv[2]).catch(console.error);
```

**Common Pitfalls:**

❌ **No integration tests**: Missing cross-table query bugs
✅ **Solution**: Test complete user workflows end-to-end

❌ **Testing only happy path**: Missing error cases
✅ **Solution**: Test not-found, validation errors, etc.

---

## 6. Advanced Topics

### 6.1 Handling Complex Queries

**Filtering After Query:**

```typescript
// Get active orders for customer
const result = await docClient.send(
  new QueryCommand({
    TableName: "Orders",
    IndexName: "CustomerIndex",
    KeyConditionExpression: "CustomerId = :customerId",
    FilterExpression: "#status = :status",
    ExpressionAttributeNames: { "#status": "Status" },
    ExpressionAttributeValues: {
      ":customerId": customerId,
      ":status": "ACTIVE",
    },
  })
);
```

**Note:** FilterExpression still scans all items, just filters results. For frequent filters, consider adding GSI.

---

### 6.2 Pagination Strategies

```typescript
export async function getOrdersPaginated(
  customerId: string,
  lastKey?: Record<string, any>
) {
  const result = await docClient.send(
    new QueryCommand({
      TableName: "Orders",
      IndexName: "CustomerIndex",
      KeyConditionExpression: "CustomerId = :customerId",
      ExpressionAttributeValues: { ":customerId": customerId },
      Limit: 20,
      ExclusiveStartKey: lastKey,
    })
  );

  return {
    items: result.Items as Order[],
    nextKey: result.LastEvaluatedKey,
    hasMore: !!result.LastEvaluatedKey,
  };
}
```

---

### 6.3 Optimistic Locking

```typescript
export async function updateCustomer(customer: Customer) {
  await docClient.send(
    new PutCommand({
      TableName: "Customers",
      Item: { ...customer, Version: (customer.Version || 0) + 1 },
      ConditionExpression: "Version = :expectedVersion",
      ExpressionAttributeValues: {
        ":expectedVersion": customer.Version || 0,
      },
    })
  );
}
```

---

### 6.4 Batch Operations

```typescript
// Batch get multiple customers
export async function getCustomersBatch(customerIds: string[]) {
  const result = await docClient.send(
    new BatchGetCommand({
      RequestItems: {
        Customers: {
          Keys: customerIds.map((id) => ({ CustomerId: id })),
        },
      },
    })
  );

  return result.Responses?.Customers as Customer[];
}
```

---

## 7. Migration & Evolution

### 7.1 Adding New Entities

**Process:**
1. Create new table with CDK
2. Implement repository
3. Deploy infrastructure
4. No impact on existing tables

**Example:** Adding Products table to existing e-commerce app

```typescript
const productsTable = new dynamodb.Table(this, "ProductsTable", {
  tableName: "Products",
  partitionKey: { name: "ProductId", type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
});
```

---

### 7.2 Adding New Access Patterns

**Process:**
1. Add GSI to existing table
2. Backfill GSI attributes on existing items (if needed)
3. Implement new query method
4. Deploy

**Example:** Adding "Get orders by status" pattern

```typescript
// Add GSI to OrdersTable
ordersTable.addGlobalSecondaryIndex({
  indexName: "StatusIndex",
  partitionKey: { name: "Status", type: dynamodb.AttributeType.STRING },
  sortKey: { name: "OrderDate", type: dynamodb.AttributeType.STRING },
});

// Backfill script (if Status attribute missing on old items)
async function backfillStatus() {
  // Scan and update items...
}
```

---

### 7.3 Migrating to Single-Table Design

See [Section 1.4](#14-migration-path-faux-sql--single-table) for overview.

**Detailed Migration Steps:**

**Phase 1: Design Single-Table Schema**
- Map each Faux-SQL table to single-table patterns
- Design GSIs for all access patterns
- Create entity-key tables

**Phase 2: Dual-Write**
- Write to both old and new tables
- Validate data parity
- Monitor for errors

**Phase 3: Backfill Historical Data**
- Scan old tables
- Transform to new format
- Write to new table

**Phase 4: Switch Reads**
- Use feature flags
- Route traffic gradually
- Monitor performance

**Phase 5: Deprecate**
- Stop dual-writes
- Archive old tables
- Delete after safety period

---

## 8. Complete Examples

### 8.1 E-commerce Application

**Tables:**
- CustomersTable (PK: CustomerId, GSI: EmailIndex)
- OrdersTable (PK: OrderId, GSI: CustomerIndex)
- OrderItemsTable (PK: OrderId, SK: ItemId)
- ProductsTable (PK: ProductId)

**Access Patterns:**
1. Get customer by ID
2. Get customer by email (login)
3. Get orders for customer
4. Get order by ID
5. Get items for order
6. Get product by ID

---

### 8.2 SaaS Multi-Tenant Application

**Tables:**
- OrganizationsTable (PK: OrgId)
- UsersTable (PK: UserId, GSI: EmailIndex, GSI: OrgIndex)
- SubscriptionsTable (PK: OrgId)

**Access Patterns:**
1. Get organization
2. Get user by ID
3. Get user by email
4. Get users for organization
5. Get subscription for organization

---

## 9. Reference

### 9.1 Entity File Template

```markdown
# [EntityName]Table

## Primary Key
- [AttributeName] (HASH)
- [AttributeName] (RANGE) - if composite

## GSI: [IndexName]
- [AttributeName] (HASH)
- [AttributeName] (RANGE) - if needed
- Purpose: [Description]
- Access Pattern: [Pattern description]

## Implemented Access Patterns
✅ [Pattern 1]
✅ [Pattern 2]

## Planned Access Patterns
🔲 [Pattern 3]
🔲 [Pattern 4]
```

---

### 9.2 CDK Template

See Section 5.1 for complete CDK examples.

---

### 9.3 Glossary

**Faux-SQL**: DynamoDB modeling approach using multiple tables with descriptive names

**Foreign Key**: Attribute referencing primary key of another table

**GSI**: Global Secondary Index for alternate access patterns

**Composite Key**: Primary key with partition key + sort key

**Denormalization**: Embedding related data to avoid additional requests

---

### 9.4 Further Reading

- **The DynamoDB Book** by Alex DeBrie (Chapter 8: Multi-Table Design)
- [AWS DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/)
- [Single-Table Design Guide](single-table-design.md) - For when you need to migrate

---

## Conclusion

Faux-SQL DynamoDB modeling provides a pragmatic approach for teams prioritizing development velocity over maximum performance. It's ideal for MVPs, early-stage applications, and teams learning DynamoDB.

**Key Takeaways:**

1. **One table per entity** - Familiar, SQL-like structure
2. **Descriptive names** - CustomerId, OrderId (not PK, SK)
3. **Simple GSIs** - One per access pattern, no overloading
4. **Incremental patterns** - Don't need all patterns upfront
5. **Easy to evolve** - Add tables and GSIs as needed
6. **Migration path exists** - Can move to single-table when scale demands

**When to Use:**
- MVP/early-stage applications
- Teams learning DynamoDB
- Analytics requirements high
- Using GraphQL
- Traffic < 10K req/sec

**When to Migrate:**
- Traffic > 10K req/sec
- Latency becomes critical
- Cost optimization needed
- Access patterns stabilized

**Good luck with your Faux-SQL DynamoDB modeling!** 🚀
