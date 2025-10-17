# Guide 4: Entity Schema Guide

## When to Use This Guide

- [ ] Completed primary key design
- [ ] Need to document non-key attributes for your entities
- [ ] Ready to implement TypeScript types for your data model

## Purpose

Capture all non-key attributes of your entities in TypeScript types/interfaces. The DynamoDB Book explicitly states not to clutter your ERD with entity attributes, but these attributes still need to be documented and enforced somewhere—that somewhere is your application code.

## Why Separate from Primary Key Design?

The book recommends separating "application attributes from your indexing attributes". This separation serves multiple purposes:

**Indexing Attributes (PK, SK, GSI keys):**

- Drive data access patterns
- Determine how items are stored and queried
- Change infrequently

**Application Attributes (everything else):**

- Business data (Name, Email, CreatedAt, etc.)
- Can evolve more easily
- Don't affect how data is retrieved

## Step-by-Step Process

### Step 1: Extract Entities from ERD

Start with the entities you identified in your ERD (`docs/specs/jobs/<job>/erd.puml`). For each entity, you'll create a corresponding TypeScript type.

**Example from E-commerce ERD:**

```
Entities: Customer, Order, OrderItem, Brand
```

### Step 2: List All Attributes

For each entity, document every attribute beyond the primary key:

**Questions to ask:**

- What business data does this entity hold?
- What timestamps do I need to track? (CreatedAt, UpdatedAt, DeletedAt)
- What attributes will appear in UI displays?
- What attributes are needed for business logic?

**Common attribute patterns:**

- **Identity:** Username, Email, PhoneNumber
- **Timestamps:** CreatedAt, UpdatedAt, ExpiresAt
- **Status:** Status, State, IsActive
- **Metadata:** Type (for filtering by entity type)
- **Relationships:** References to other entities (handled carefully)

### Step 3: Choose Attribute Types

Map each attribute to a DynamoDB type:

**Scalar types:**

- `string` → DynamoDB String (S)
- `number` → DynamoDB Number (N)
- `boolean` → DynamoDB Boolean (BOOL)
- `null` → DynamoDB Null (NULL)
- `Buffer` → DynamoDB Binary (B)

**Complex types:**

- `array` → DynamoDB List (L)
- `object` → DynamoDB Map (M)

**Set types:**

- `Set<string>` → String Set (SS)
- `Set<number>` → Number Set (NS)
- `Set<Buffer>` → Binary Set (BS)

### Step 4: Create TypeScript Types

Create a file: `docs/specs/jobs/<job>/entity-schemas.ts`

**Basic structure:**

```typescript
// Base attributes common to all entities
interface BaseEntity {
  Type: string; // Entity type identifier
  CreatedAt: string; // ISO-8601 timestamp
  UpdatedAt?: string; // ISO-8601 timestamp
}

// Customer entity
interface Customer extends BaseEntity {
  Type: "CUSTOMER";
  Username: string;
  Email: string;
  Name: string;
  DateOfBirth?: string;
}

// Order entity
interface Order extends BaseEntity {
  Type: "ORDER";
  OrderId: string;
  CustomerId: string; // Reference to Customer
  OrderDate: string;
  TotalAmount: number;
  Status: "PENDING" | "SHIPPED" | "DELIVERED";
}
```

**Why include Type?**  
The book recommends adding a Type attribute to every item. This helps with:

- Filtering by entity type in queries
- Debugging and data exploration
- ETL processes for analytics

### Step 5: Document Complex Attributes

For nested structures (Maps and Lists), document their shape:

```typescript
interface Organization extends BaseEntity {
  Type: "ORGANIZATION";
  OrgName: string;

  // Nested map for payment plan
  PaymentPlan: {
    PlanType: "FREE" | "PRO" | "ENTERPRISE";
    MaxUsers: number;
    BillingDate?: string;
  };

  // Nested list for featured items
  FeaturedDeals: Array<{
    DealId: string;
    Position: number;
  }>;
}
```

### Step 6: Handle Uniqueness Tracking

Some entities require additional tracking items for uniqueness. Document these separately:

```typescript
// Main Customer entity
interface Customer extends BaseEntity {
  Type: "CUSTOMER";
  Username: string; // Unique
  Email: string; // Also must be unique
  Name: string;
}

// Email tracking item (enforces email uniqueness)
interface CustomerEmailTracker {
  Type: "CUSTOMER_EMAIL_TRACKER";
  Email: string;
  Username: string; // Points back to main Customer
}
```

### Step 7: Cross-Reference with Access Patterns

Review your access patterns table (`docs/specs/jobs/<job>/data-access-patterns.md`). Ensure every attribute needed to satisfy access patterns is captured in your schemas.

**Validation checklist:**

- [ ] All attributes displayed in UI mockups are included
- [ ] All filter/sort requirements are represented
- [ ] All attributes needed for business logic are documented
- [ ] Timestamp fields for time-based queries are included

## Integration with Your CDK Stack

Once you have TypeScript types, you can use them throughout your application:

```typescript
// In your Lambda handler
import { Customer, Order } from "@/types/entities";

async function createOrder(
  customer: Customer,
  items: OrderItem[]
): Promise<Order> {
  const order: Order = {
    Type: "ORDER",
    OrderId: generateId(),
    CustomerId: customer.Username,
    OrderDate: new Date().toISOString(),
    TotalAmount: calculateTotal(items),
    Status: "PENDING",
    CreatedAt: new Date().toISOString(),
  };

  await dynamodb.put({
    TableName: process.env.TABLE_NAME,
    Item: marshall(order), // Type-safe!
  });

  return order;
}
```

## Common Patterns

### Pattern 1: Soft Deletes

Instead of deleting items, mark them as deleted:

```typescript
interface BaseEntity {
  Type: string;
  CreatedAt: string;
  UpdatedAt?: string;
  DeletedAt?: string; // Soft delete marker
  IsDeleted?: boolean; // Quick filter flag
}
```

### Pattern 2: Versioning

Track schema versions for migrations:

```typescript
interface BaseEntity {
  Type: string;
  SchemaVersion: number; // e.g., 1, 2, 3
  CreatedAt: string;
  UpdatedAt?: string;
}
```

### Pattern 3: Audit Trail

Track who made changes:

```typescript
interface AuditableEntity extends BaseEntity {
  CreatedBy: string; // Username who created
  UpdatedBy?: string; // Username who last updated
}
```

## Validation Checklist

- [ ] Every entity from ERD has a TypeScript type
- [ ] All tracking items (for uniqueness, etc.) are documented
- [ ] Type attribute included on all entities
- [ ] Timestamp fields use ISO-8601 strings
- [ ] Nested structures (Maps/Lists) are fully typed
- [ ] Cross-referenced with access patterns—no missing attributes
- [ ] Schema file saved to `docs/specs/jobs/<job>/entity-schemas.ts`
- [ ] Types exported for use in Lambda handlers

## Anti-Patterns to Avoid

❌ Including PK/SK in entity types → Keep indexing separate  
❌ Using `any` or overly generic types → Defeats the purpose  
❌ Forgetting the Type attribute → Makes debugging harder  
❌ Not documenting tracking items → Uniqueness logic gets lost  
❌ Inconsistent timestamp formats → Use ISO-8601 everywhere

## Example: Session Store

From the ERD we have:

- User entity
- Session entity

**Entity schemas:**

```typescript
interface User {
  Type: "USER";
  Username: string;
  Email: string;
  PasswordHash: string;
  CreatedAt: string;
  UpdatedAt?: string;
}

interface Session {
  Type: "SESSION";
  SessionToken: string;
  Username: string; // Reference to User
  ExpiresAt: string; // ISO-8601 timestamp for TTL
  CreatedAt: string;
  LastAccessedAt?: string;
}
```

Note: The Session has an `ExpiresAt` field. This is for TTL functionality, which will automatically delete expired sessions.

Output: `docs/specs/jobs/session-store/entity-schemas.ts`
