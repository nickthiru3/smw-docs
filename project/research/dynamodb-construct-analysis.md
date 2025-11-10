# DynamoDB Construct Analysis: Single-Table vs Faux-SQL

**Date**: 2025-11-10  
**Context**: Story 001 Implementation - Database Layer  
**Decision**: Refactor database construct to support Faux-SQL approach

---

## Current State: Single-Table Design

### Existing Implementation (`lib/db/construct.ts`)

**Table Schema**:

```typescript
{
  PK: string,          // Partition Key (generic)
  SK: string,          // Sort Key (generic)
  GSI1PK: string,      // GSI partition key (generic)
  GSI1SK: string       // GSI sort key (generic)
}
```

**Characteristics**:

- ✅ Generic key names (PK, SK, GSI1PK, GSI1SK)
- ✅ Single table for all entities
- ✅ One GSI (GSI1) for alternate access patterns
- ✅ Overloaded keys (e.g., `PK=USER#123`, `SK=PROFILE#456`)
- ✅ Point-in-time recovery enabled
- ✅ Environment-based deletion protection

**Use Case**: High-performance applications with well-defined access patterns

---

## Required State: Faux-SQL Design

### SMW Application Requirements

**Data Modeling Approach**: Faux-SQL (per `docs/guides/data-modeling/faux-sql-design.md`)

**Characteristics**:

- ✅ **Multiple tables**: One table per entity type (Merchants, Reviews, etc.)
- ✅ **Descriptive keys**: Meaningful names (MerchantId, CategoryId) instead of generic (PK, SK)
- ✅ **Normalized data**: Follow relational normalization principles
- ✅ **Simple GSIs**: One index per access pattern, no overloading
- ✅ **Multiple requests**: Accept 2-3 serial requests for related data

**Benefits for SMW**:

- ✅ **Fast development**: Familiar patterns, quick iteration
- ✅ **Easy to understand**: SQL-like structure, clear naming
- ✅ **Flexible**: Easy to add entities and access patterns
- ✅ **Analytics-friendly**: Normalized data, easy exports
- ✅ **AI-compatible**: Standard patterns for code generation
- ✅ **Lower cognitive load**: No complex key overloading

**Trade-offs Accepted**:

- ⚠️ Higher latency (50-100ms vs 10ms) - acceptable for MVP
- ⚠️ More expensive at scale - acceptable at <10K req/sec
- ⚠️ No atomic transactions across tables - not needed for current use cases

---

## Comparison: Single-Table vs Faux-SQL

### Single-Table Design (Current)

**Example**: Users and Orders in one table

```typescript
// UsersTable (single table)
{
  PK: "USER#123",
  SK: "PROFILE",
  Email: "alice@example.com",
  Name: "Alice"
}
{
  PK: "USER#123",
  SK: "ORDER#456",
  OrderDate: "2024-01-15",
  TotalAmount: 99.99
}
{
  PK: "ORDER#456",
  SK: "METADATA",
  CustomerId: "USER#123",
  Status: "shipped"
}
```

**GSI1**: Overloaded for multiple access patterns

```typescript
{
  GSI1PK: "ORDER#STATUS#shipped",
  GSI1SK: "2024-01-15"
}
```

**Queries**:

- Get user: `PK=USER#123 AND SK=PROFILE` (1 request)
- Get user's orders: `PK=USER#123 AND SK begins_with ORDER#` (1 request)
- Get orders by status: `GSI1PK=ORDER#STATUS#shipped` (1 request)

**Complexity**: High - requires understanding of key overloading patterns

---

### Faux-SQL Design (Required)

**Example**: Merchants and Reviews in separate tables

```typescript
// MerchantsTable
{
  MerchantId: "M123",      // Partition Key (descriptive)
  Name: "Green Recyclers",
  Email: "contact@greenrecyclers.com",
  Category: "plastic",
  Location: {...}
}

// ReviewsTable
{
  ReviewId: "R456",        // Partition Key
  MerchantId: "M123",      // Foreign key
  Rating: 5,
  Comment: "Great service!",
  CreatedAt: "2024-01-15"
}
```

**GSI: CategoryIndex** (on MerchantsTable)

```typescript
{
  Category: "plastic",     // GSI Partition Key (descriptive)
  MerchantId: "M123"       // GSI Sort Key (descriptive)
}
```

**GSI: MerchantIndex** (on ReviewsTable)

```typescript
{
  MerchantId: "M123",      // GSI Partition Key
  CreatedAt: "2024-01-15"  // GSI Sort Key
}
```

**Queries**:

- Get merchant: `MerchantId=M123` (1 request)
- Get merchants by category: `CategoryIndex: Category=plastic` (1 request)
- Get reviews for merchant: `MerchantIndex: MerchantId=M123` (1 request)

**Complexity**: Low - SQL-like queries with descriptive names

---

## DynamoDB Construct Requirements

### Common Features (Both Approaches)

Both single-table and faux-SQL constructs need:

1. **Environment-based protection**:

   ```typescript
   const shouldProtectFromDeletion = envName !== "local" && envName !== "dev";
   deletionProtection: shouldProtectFromDeletion;
   removalPolicy: shouldProtectFromDeletion
     ? RemovalPolicy.RETAIN
     : RemovalPolicy.DESTROY;
   ```

2. **Point-in-time recovery**:

   ```typescript
   pointInTimeRecoverySpecification: {
     pointInTimeRecoveryEnabled: true;
   }
   ```

3. **CloudFormation outputs**:

   ```typescript
   new CfnOutput(this, "TableName", {
     value: tableName,
     exportName: `${serviceName}-TableName`,
   });
   ```

4. **Public table property**:
   ```typescript
   public readonly table: TableV2;
   ```

---

### Single-Table Specific Features

**Key Schema**:

```typescript
partitionKey: { name: "PK", type: AttributeType.STRING },
sortKey: { name: "SK", type: AttributeType.STRING }
```

**GSI Schema** (generic, overloaded):

```typescript
globalSecondaryIndexes: [
  {
    indexName: "GSI1",
    partitionKey: { name: "GSI1PK", type: AttributeType.STRING },
    sortKey: { name: "GSI1SK", type: AttributeType.STRING },
    projectionType: ProjectionType.ALL,
  },
];
```

**Use Case**: High-performance apps with stable access patterns

---

### Faux-SQL Specific Features

**Key Schema** (descriptive names):

```typescript
// MerchantsTable
partitionKey: { name: "MerchantId", type: AttributeType.STRING }
// No sort key needed for simple lookup

// ReviewsTable (composite key for hierarchy)
partitionKey: { name: "ReviewId", type: AttributeType.STRING },
sortKey: { name: "MerchantId", type: AttributeType.STRING }
```

**GSI Schema** (descriptive names, one per access pattern):

```typescript
// MerchantsTable - CategoryIndex
globalSecondaryIndexes: [
  {
    indexName: "CategoryIndex",
    partitionKey: { name: "Category", type: AttributeType.STRING },
    sortKey: { name: "MerchantId", type: AttributeType.STRING },
    projectionType: ProjectionType.ALL,
  },
];

// ReviewsTable - MerchantIndex
globalSecondaryIndexes: [
  {
    indexName: "MerchantIndex",
    partitionKey: { name: "MerchantId", type: AttributeType.STRING },
    sortKey: { name: "CreatedAt", type: AttributeType.STRING },
    projectionType: ProjectionType.ALL,
  },
];
```

**Use Case**: MVP/early-stage apps, analytics-heavy, GraphQL, AI-assisted development

---

## Proposed Refactoring

### Directory Structure

```
lib/db/
├── single-table/
│   └── construct.ts          # Existing single-table construct (moved)
├── faux-sql/
│   └── construct.ts          # New faux-SQL construct
└── construct.ts              # Removed (replaced by approach-specific constructs)
```

### Why Separate Constructs?

**Rationale**:

1. **Different philosophies**: Single-table and faux-SQL are fundamentally different approaches
2. **Different schemas**: Generic keys (PK/SK) vs descriptive keys (MerchantId/Category)
3. **Different GSI patterns**: Overloaded GSIs vs one-per-access-pattern
4. **Template reusability**: Both patterns should be available in template
5. **Clear separation**: Developers choose approach explicitly

**Alternative Considered**: Single construct with configuration flag

- ❌ **Rejected**: Too complex, mixes concerns, harder to maintain

---

### Faux-SQL Construct Design

**File**: `lib/db/faux-sql/construct.ts`

**Interface**:

```typescript
export interface IFauxSqlDatabaseConstructProps {
  readonly config: IConfig;
  readonly tables: ITableDefinition[];
}

export interface ITableDefinition {
  readonly tableName: string;
  readonly partitionKey: IKeyAttribute;
  readonly sortKey?: IKeyAttribute;
  readonly globalSecondaryIndexes?: IGsiDefinition[];
}

export interface IKeyAttribute {
  readonly name: string;
  readonly type: AttributeType;
}

export interface IGsiDefinition {
  readonly indexName: string;
  readonly partitionKey: IKeyAttribute;
  readonly sortKey?: IKeyAttribute;
  readonly projectionType?: ProjectionType;
}
```

**Features**:

- ✅ **Multiple tables**: Create multiple tables from config
- ✅ **Descriptive keys**: Use meaningful attribute names
- ✅ **Simple GSIs**: One index per access pattern
- ✅ **Flexible schema**: Easy to add new tables/indexes
- ✅ **Same protections**: Environment-based deletion protection, PITR

**Example Usage**:

```typescript
// In ServiceStack
const db = new FauxSqlDatabaseConstruct(this, "DatabaseConstruct", {
  config,
  tables: [
    {
      tableName: "Merchants",
      partitionKey: { name: "MerchantId", type: AttributeType.STRING },
      globalSecondaryIndexes: [
        {
          indexName: "CategoryIndex",
          partitionKey: { name: "Category", type: AttributeType.STRING },
          sortKey: { name: "MerchantId", type: AttributeType.STRING },
        },
      ],
    },
    {
      tableName: "Reviews",
      partitionKey: { name: "ReviewId", type: AttributeType.STRING },
      sortKey: { name: "MerchantId", type: AttributeType.STRING },
      globalSecondaryIndexes: [
        {
          indexName: "MerchantIndex",
          partitionKey: { name: "MerchantId", type: AttributeType.STRING },
          sortKey: { name: "CreatedAt", type: AttributeType.STRING },
        },
      ],
    },
  ],
});

// Access tables
const merchantsTable = db.tables.get("Merchants");
const reviewsTable = db.tables.get("Reviews");
```

---

## Implementation Plan

### Phase 1: Refactor Existing Construct

1. **Create directory structure**:

   ```bash
   mkdir -p lib/db/single-table
   mkdir -p lib/db/faux-sql
   ```

2. **Move existing construct**:

   ```bash
   mv lib/db/construct.ts lib/db/single-table/construct.ts
   ```

3. **Update imports** in `lib/service-stack.ts`:

   ```typescript
   // Old
   import DatabaseConstruct from "#lib/db/construct";

   // New
   import DatabaseConstruct from "#lib/db/single-table/construct";
   ```

4. **Verify** existing functionality still works

---

### Phase 2: Create Faux-SQL Construct

1. **Create** `lib/db/faux-sql/construct.ts`
2. **Implement** multi-table support with descriptive keys
3. **Add** CloudFormation outputs for each table
4. **Test** with Story 001 Merchants table

---

### Phase 3: Update ServiceStack

1. **Switch** to faux-SQL construct:

   ```typescript
   import DatabaseConstruct from "#lib/db/faux-sql/construct";
   ```

2. **Configure** Merchants table:

   ```typescript
   const db = new DatabaseConstruct(this, "DatabaseConstruct", {
     config,
     tables: [
       {
         tableName: "Merchants",
         partitionKey: { name: "MerchantId", type: AttributeType.STRING },
         globalSecondaryIndexes: [
           {
             indexName: "CategoryIndex",
             partitionKey: { name: "Category", type: AttributeType.STRING },
           },
         ],
       },
     ],
   });
   ```

3. **Update** ApiConstruct to use new table structure

---

## Migration Path (Future)

If SMW grows and needs single-table design:

1. **Keep faux-SQL tables** for analytics
2. **Create single-table** for high-performance queries
3. **Dual-write** to both during migration
4. **Switch reads** gradually with feature flags
5. **Deprecate faux-SQL** tables after validation

**Timeline**: 2-4 weeks for complete migration

---

## Recommendations

### For Story 001

1. ✅ **Use faux-SQL approach**: Aligns with SMW data modeling guide
2. ✅ **Create separate construct**: Keep single-table for template
3. ✅ **Start with Merchants table**: Simple schema, one GSI
4. ✅ **Add tables incrementally**: Reviews, Categories as needed

### For Template

1. ✅ **Keep both constructs**: Developers choose based on use case
2. ✅ **Document trade-offs**: Clear guidance on when to use each
3. ✅ **Provide examples**: Working code for both approaches

### For Implementation

1. ✅ **Bottom-up approach**: Database → Lambda → API Gateway
2. ✅ **Test with DynamoDB Local**: Validate schema before deployment
3. ✅ **Document decisions**: Capture in story implementation log
4. ✅ **Update guides**: Reflect actual patterns used

---

## Next Steps

1. ✅ **Create faux-SQL construct** following existing code style
2. ✅ **Implement Merchants table** with CategoryIndex GSI
3. ✅ **Test locally** with DynamoDB Local
4. ✅ **Update ServiceStack** to use new construct
5. ✅ **Document patterns** in implementation guides
6. ✅ **Proceed to Lambda handler** implementation

---

## References

- **Faux-SQL Guide**: `docs/guides/data-modeling/faux-sql-design.md`
- **Single-Table Guide**: `docs/guides/data-modeling/single-table-design.md`
- **Story 001 Data Model**: `docs/project/specs/stories/consumers/browse-providers-by-waste-category/data-model.md`
- **Existing Construct**: `lib/db/single-table/construct.ts` (after refactor)
- **DynamoDB Book**: "The DynamoDB Book" by Alex DeBrie
