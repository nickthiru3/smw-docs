# DynamoDB Data Modeling Quick Reference Cards

## Overview

These quick reference cards provide at-a-glance guidance for common data modeling scenarios. Use these as a starting point, then refer to the detailed guides for full implementation.

---

## Card 1: When to Use Which Strategy

### One-to-Many Relationships

| Scenario                                 | Strategy                     | Guide                                                |
| ---------------------------------------- | ---------------------------- | ---------------------------------------------------- |
| Small, bounded list (<20 items)          | Complex attribute (List/Map) | [Guide 5](guides/5-relationship-strategies.md)       |
| Immutable data                           | Denormalize & duplicate      | [Guide 5](guides/5-relationship-strategies.md)       |
| Need to query parent + children together | Composite primary key        | [Guide 3](guides/3-primary-key-design.md)            |
| Primary key used for something else      | Secondary index              | [Guide 6](guides/6-secondary-index-strategies.md)    |
| Deep hierarchy with multi-level queries  | Composite sort key           | [Guide 5](guides/5-relationship-strategies.md)       |

### Many-to-Many Relationships

| Scenario                                       | Strategy                          | Guide                                                |
| ---------------------------------------------- | --------------------------------- | ---------------------------------------------------- |
| Only need limited attributes of related entity | Shallow duplication               | [Guide 5](guides/5-relationship-strategies.md)       |
| Need bidirectional queries                     | Adjacency list                    | [Guide 5](guides/5-relationship-strategies.md)       |
| Highly interconnected graph data               | Materialized graph                | [Guide 5](guides/5-relationship-strategies.md)       |
| Highly mutable relationship data               | Normalization (multiple requests) | [Guide 5](guides/5-relationship-strategies.md)       |

### Migrations

| Change Type                     | Strategy            | Additive? | Guide                                             |
| ------------------------------- | ------------------- | --------- | ------------------------------------------------- |
| Add optional attribute          | Lazy loading        | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md)       |
| Add independent entity          | New item collection | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md)       |
| Add related entity (co-located) | Share partition key | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md)       |
| Add entity needing GSI          | Create GSI + ETL    | ❌ No     | [Guide 8](guides/8-migration-strategies.md)       |
| Refactor existing pattern       | New indexes + ETL   | ❌ No     | [Guide 8](guides/8-migration-strategies.md)       |

---

## Card 2: Primary Key Design Checklist

### ✅ Good Primary Key Design

- [ ] Client knows the key at read time
- [ ] Uses generic names (PK, SK, not CustomerId)
- [ ] Includes entity type prefix (USER#, ORDER#)
- [ ] Enables grouping of related items in collections
- [ ] No timestamps unless client knows them

### ❌ Bad Primary Key Design

- ❌ Descriptive key names (CustomerId, OrderDate)
- ❌ CreatedAt timestamp in key
- ❌ Client doesn't know the value at query time
- ❌ No entity prefixes to distinguish types
- ❌ Unbounded lists as attributes

### Common Patterns

**Simple Primary Key:**

```

PK: USER#<username>
SK: USER#<username>

```

Use when: Single-item lookups only

**Composite Primary Key (Hierarchical):**

```

Parent: PK: USER#<username>, SK: USER#<username>
Child: PK: USER#<username>, SK: ORDER#<timestamp>#<orderId>

```

Use when: Need to fetch parent + children together

**Composite Primary Key (Many-to-Many):**

```

PK: USER#<userId>, SK: ORG#<orgId>
PK: ORG#<orgId>, SK: USER#<userId>

```

Use when: Adjacency list pattern for M:N relationships

---

## Card 3: Secondary Index Quick Guide

### When You Need a GSI

- Query by non-primary-key attribute
- Reverse lookup (Order → Customer when table is Customer → Order)
- Filter by status, category, or other enum
- Sort by different attribute than primary SK
- Handle access pattern not satisfied by primary key

### GSI Design Principles

1. **Overload GSIs**

   - Use generic names: GSI1PK, GSI1SK, GSI2PK, GSI2SK
   - Handle multiple entity types per GSI
   - Don't create one GSI per access pattern

2. **Separate Attributes**

   - Never reuse SK for GSI1SK
   - Use dedicated attributes per index
   - Makes evolution easier

3. **Projection Strategy**
   - KEYS_ONLY: Minimal storage, requires GetItem
   - INCLUDE: Specify needed attributes
   - ALL: Copy everything (highest cost)

### GSI Patterns

**Pattern 1: Direct Lookup**

```typescript
// Get Order by OrderId
GSI1PK: ORDER#<orderId>
GSI1SK: ORDER#<orderId>
```

**Pattern 2: Grouped by Attribute**

```typescript
// Get Orders by Status
GSI1PK: STATUS#<status>
GSI1SK: <timestamp>#<orderId>
```

**Pattern 3: Sparse Index**

```typescript
// Only index items where Status = 'PENDING'
GSI1PK: PENDING#ORDER  (only when Status = 'PENDING')
GSI1SK: <timestamp>     (only when Status = 'PENDING')
```

---

## Card 4: Common Anti-Patterns to Avoid

### Access Patterns

- ❌ Scan operations in application code
- ❌ Filter expressions as primary access mechanism
- ❌ Multiple serial requests for related data
- ❌ "We'll just Scan and filter for now"

### Key Design

- ❌ Timestamps in keys unless client knows them
- ❌ Descriptive key names (CustomerId, OrderDate)
- ❌ Reusing attributes across indexes
- ❌ Missing Type attribute on items
- ❌ No entity prefixes on keys

### Relationships

- ❌ Not pre-joining data in item collections
- ❌ Unbounded lists stored in single items
- ❌ Normalizing data to save storage
- ❌ Multiple serial requests for related entities

### Architecture

- ❌ Using an ORM
- ❌ Data access not at boundary of application
- ❌ Not separating indexing from application attributes
- ❌ Multiple tables for related entities

---

## Card 5: Entity Chart Template

### Main Table Entity Chart

| Entity  | PK Pattern              | SK Pattern                      | Notes               |
| ------- | ----------------------- | ------------------------------- | ------------------- |
| User    | USER#\<username\>       | USER#\<username\>               | Simple lookup       |
| Order   | CUSTOMER#\<customerId\> | ORDER#\<timestamp\>#\<orderId\> | Grouped by customer |
| Product | PRODUCT#\<productId\>   | PRODUCT#\<productId\>           | Simple lookup       |

### GSI1 Entity Chart

| Entity | GSI1PK Pattern    | GSI1SK Pattern            | Access Pattern       | Notes          |
| ------ | ----------------- | ------------------------- | -------------------- | -------------- |
| Order  | ORDER#\<orderId\> | ORDER#\<orderId\>         | Get Order by ID      | Direct lookup  |
| Order  | STATUS#\<status\> | \<timestamp\>#\<orderId\> | Get Orders by Status | Sorted by date |
| User   | EMAIL#\<email\>   | EMAIL#\<email\>           | Get User by Email    | Unique lookup  |

### Access Patterns Table

| Entity | Access Pattern          | Index | Key Condition                | Parameters        | Frequency |
| ------ | ----------------------- | ----- | ---------------------------- | ----------------- | --------- |
| User   | Get User by username    | Main  | PK = USER#\<username\>       | username          | High      |
| Order  | Get Orders for Customer | Main  | PK = CUSTOMER#\<customerId\> | customerId, limit | High      |
| Order  | Get Order by OrderId    | GSI1  | GSI1PK = ORDER#\<orderId\>   | orderId           | Medium    |

---

## Card 6: TypeScript Entity Schema Template

```typescript
// Base interface for all items
interface BaseItem {
  PK: string;
  SK: string;
  Type: string; // Always include Type attribute
  CreatedAt: string; // ISO-8601 timestamp
  UpdatedAt: string;
}

// Entity-specific interface
interface User extends BaseItem {
  Type: "User"; // Literal type
  Username: string;
  Email: string;
  Name: string;
  // Application attributes
}

interface Order extends BaseItem {
  Type: "Order";
  OrderId: string;
  CustomerId: string;
  Status: "PENDING" | "SHIPPED" | "DELIVERED";
  TotalAmount: number;
  Items: OrderItem[]; // Complex attribute
  // GSI attributes
  GSI1PK?: string; // ORDER#<orderId>
  GSI1SK?: string; // ORDER#<orderId>
}

// Nested complex type
interface OrderItem {
  ProductId: string;
  Quantity: number;
  PriceAtPurchase: number;
}
```

### Key Points

- Always include `Type` attribute
- Use ISO-8601 for timestamps
- Complex attributes for nested data
- Optional GSI attributes
- Separate indexing from application attributes

---

## Card 7: Implementation Checklist

### Before You Start Coding

- [ ] ERD created and approved
- [ ] All access patterns documented with parameters
- [ ] Entity charts created (main table + GSIs)
- [ ] Relationship strategies decided
- [ ] Anti-pattern review completed
- [ ] Peer review sign-off

### During Implementation

- [ ] Data access layer at application boundary
- [ ] Type attribute on all items
- [ ] Generic key names (PK, SK, GSI1PK, etc.)
- [ ] Separate attributes per index
- [ ] Entity prefixes on all keys
- [ ] No ORM usage

### After Implementation

- [ ] All access patterns tested
- [ ] Performance testing completed
- [ ] Scripts written for debugging access patterns
- [ ] Documentation updated with entity charts
- [ ] Access patterns table in repository

---

## Card 8: Migration Decision Tree

```
Does the migration require updating existing items?
│
├─ NO (Additive) → Low risk, no ETL required
│   │
│   ├─ Adding optional attribute?
│   │   └─ Strategy 1: Lazy loading
│   │
│   ├─ Adding independent entity?
│   │   └─ Strategy 2: New item collection
│   │
│   └─ Adding related entity (co-located)?
│       └─ Strategy 3: Share partition key
│
└─ YES (Non-Additive) → Higher risk, requires ETL
    │
    ├─ Need new GSI for existing items?
    │   └─ Strategy 4: Create GSI + ETL
    │
    └─ Refactoring existing pattern?
        └─ Strategy 5: New indexes + full ETL
```

### ETL Best Practices

1. **Use Type attribute for filtering**
2. **Parallel scans for speed**
3. **Handle LastEvaluatedKey properly**
4. **Batch updates when possible**
5. **Monitor progress and errors**
6. **Test on copy of data first**
7. **Have rollback plan ready**

---

## Card 9: Debugging Checklist

### When Access Pattern Isn't Working

**Check Primary Key:**

- [ ] Does client know the key at query time?
- [ ] Are entity prefixes consistent?
- [ ] Is partition key grouping items correctly?

**Check Sort Key:**

- [ ] Is sort order what you expect?
- [ ] Are items in same item collection?
- [ ] Is begins_with condition correct?

**Check GSI:**

- [ ] Are GSI attributes populated on items?
- [ ] Is projection strategy correct?
- [ ] Is eventual consistency acceptable?

**Check Query Conditions:**

- [ ] KeyConditionExpression syntax correct?
- [ ] FilterExpression not primary mechanism?
- [ ] Parameters match key patterns?

### Performance Issues

- [ ] Using Query instead of Scan?
- [ ] Item size under 400KB?
- [ ] Not hitting partition throughput limits?
- [ ] GSIs properly overloaded?
- [ ] Avoiding hot partitions?

---

## Card 10: Resource Quick Links

### Process Guides

- [Phase 1: Experience & Domain Design](phases/1-experience-and-domain-design.md)
- [Phase 2: Data Modeling](phases/2-data-modeling.md)
- [Phase 3: Implementation](phases/3-implementation.md)

### Strategy Guides

- [Guide 1: ERD Creation](guides/1-erd-creation.md)
- [Guide 2: Access Patterns](guides/2-access-patterns-definition.md)
- [Guide 3: Primary Key Design](guides/3-primary-key-design.md)
- [Guide 4: Entity Schema](guides/4-entity-schema.md)
- [Guide 5: Relationship Strategies](guides/5-relationship-strategies.md)
- [Guide 6: Secondary Indexes](guides/6-secondary-index-strategies.md)
- [Guide 7: Anti-Patterns](guides/7-common-anti-patterns.md)
- [Guide 8: Migration Strategies](guides/8-migration-strategies.md)

### Reference Materials

- [Artifact Mapping Table](artifact-mapping-table.md)
- DynamoDB Book Chapter 7: Data Modeling Approach
- DynamoDB Book Chapters 11-12: Relationship Strategies
- DynamoDB Book Chapter 15: Migration Strategies

---

## Printable Cheat Sheet

### Golden Rules

1. **Know your access patterns first**
2. **Pre-join data in item collections**
3. **Use generic key names (PK, SK)**
4. **Overload GSIs, don't create one per pattern**
5. **Separate indexing from application attributes**
6. **Always include Type attribute**
7. **Don't use Scan in application code**
8. **Storage is cheap, compute is expensive**
9. **Plan migrations before making changes**
10. **Test on copy of production data**

### Common Patterns

| Need                        | Solution                         |
| --------------------------- | -------------------------------- |
| Unique identifier           | Use in PK                        |
| Query parent + children     | Composite PK, shared partition   |
| Reverse lookup              | Create GSI                       |
| Filter by status            | Sparse GSI or composite sort key |
| Many-to-many                | Adjacency list                   |
| Sort by different attribute | Secondary index                  |
| Bounded list                | Complex attribute (List/Map)     |
| Unbounded collection        | Separate items, shared PK        |

### Quick Syntax

**Query:**

```typescript
KeyConditionExpression: "PK = :pk AND begins_with(SK, :sk)";
```

**Sparse Index:**

```typescript
// Only set GSI attributes when condition met
GSI1PK: condition ? "VALUE" : undefined;
```

**Composite Sort Key:**

```typescript
SK: "LEVEL1#<val1>#LEVEL2#<val2>#LEVEL3#<val3>";
```

---

## Next Steps

1. **Starting New Model:** Begin with [Guide 1: ERD Creation](guides/1-erd-creation.md)
2. **Defining Queries:** Use [Guide 2: Access Patterns](guides/2-access-patterns-definition.md)
3. **Designing Keys:** Apply [Guide 3: Primary Key Design](guides/3-primary-key-design.md)
4. **Need Help:** Check [Guide 7: Anti-Patterns](guides/7-common-anti-patterns.md)
5. **Making Changes:** Consult [Guide 8: Migration Strategies](guides/8-migration-strategies.md)
