# Guide 3: Primary Key Design Guide

## When to Use This Guide

- [ ] Completed ERD
- [ ] Completed access patterns table
- [ ] Ready to design DynamoDB table structure

## Purpose

Translate your ERD and access patterns into the primary key structure and entity chart that drives all DynamoDB data access.

## The Three Key Questions

Before touching your entity chart, answer these:

### 1. Simple or Composite Primary Key?

**Choose Simple** (partition key only) if:

- Single entity type
- No "fetch many" patterns
- All access is by unique identifier

**Choose Composite** (partition + sort key) if:

- Multiple entity types
- Any "get all X for Y" patterns
- Need item collections for relationships

**For your serverless stack:** Default to composite unless you have a very simple use case.

### 2. What Are Your Interesting Requirements?

Identify constraints that affect primary key design:

**Uniqueness constraints:**

- Multiple attributes must be unique (e.g., username AND email)
- Shared identity across types (GitHub issue/PR numbers)

**Time-based access:**

- Latest N items
- Items in time range
- Expiration (TTL eligible)

**Unbounded relationships:**

- Relationships with potentially thousands of items
- Need pagination

**Reference counts:**

- Maintaining counts on parent entities

### 3. Which Entity to Start With?

**Priority order:**

1. Entities with uniqueness requirements → Model in primary key
2. Parent entities in one-to-many relationships → Build item collections around them
3. Core/frequently-accessed entities → Optimize for common patterns

## Step-by-Step Process

### Step 1: Create Entity Chart

Start with a blank chart tracking partition key (PK) and sort key (SK) patterns:

| Entity  | PK  | SK  | Notes |
| ------- | --- | --- | ----- |
| User    |     |     |       |
| Session |     |     |       |
| Order   |     |     |       |

Save to: `docs/specs/jobs/<job>/entity-chart.md`

### Step 2: Model for Uniqueness First

For each entity, identify the primary identifier(s):

**Single identifier:**

```
Entity: User
Identifier: username
PK: USER#<username>
SK: USER#<username>
```

**Multiple unique attributes:**  
Use tracking items:

```
Entity: Customer
Unique on: username, email

Main item:
PK: CUSTOMER#<username>
SK: CUSTOMER#<username>

Email tracking item:
PK: CUSTOMEREMAIL#<email>
SK: CUSTOMEREMAIL#<email>
Attributes: { username }
```

### Step 3: Design Composite Keys for Relationships

#### One-to-Many Strategy

Put parent and children in same item collection:

**Example: User → Sessions**

```
User item:
PK: USER#<username>
SK: USER#<username>

Session items:
PK: USER#<username>
SK: SESSION#<sessionId>
```

**Query:** PK = `USER#alice` returns User + all Sessions

#### Sorting Requirements

Use sort key structure to control order:

**Descending order (latest first):**

```
Order items:
PK: CUSTOMER#<username>
SK: #ORDER#<timestamp-or-KSUID>
```

Prefix with `#` to sort orders after the customer item.

**Ascending order:**

```
SK: ORDER#<incrementing-number>
```

### Step 4: Apply Prefixing Patterns

Use prefixes to distinguish entity types and prevent collisions:

**Standard format:**

```
<ENTITY_TYPE>#<identifier>
```

**Examples:**

- `USER#alexdebrie`
- `ORDER#550e8400`
- `REPO#owner#repo-name` (hierarchical)

**Benefits:**

- Prevents accidental overlaps
- Makes data explorable in console
- Enables filtering by Type attribute

### Step 5: Handle Special Cases

#### Shared Identifiers

When multiple entity types share numbering:

```
GitHub Repo:
PK: REPO#<owner>#<repo>
SK: REPO#<owner>#<repo>

Issue (shares numbers with PRs):
PK: REPO#<owner>#<repo>
SK: ISSUE#<zero-padded-number>

Pull Request:
PK: REPO#<owner>#<repo>
SK: PR#<zero-padded-number>
```

Both use the same item collection but different prefixes.

#### Singleton Items

Global state or meta-views:

```
Singleton:
PK: FRONTPAGE
SK: FRONTPAGE
```

All instances share same PK/SK (single item).

#### Zero-Padding Numbers

For sortable numeric IDs:

```
SK: ISSUE#00001
SK: ISSUE#00042
SK: ISSUE#00123
```

Ensures lexicographic sorting matches numeric order.

### Step 6: Update Entity Chart

Fill in your chart with the patterns:

| Entity        | PK                    | SK                    | Notes                          |
| ------------- | --------------------- | --------------------- | ------------------------------ |
| User          | USER#<username>       | USER#<username>       | Simple lookup                  |
| Session       | USER#<username>       | SESSION#<sessionId>   | In User item collection        |
| Order         | CUSTOMER#<username>   | #ORDER#<orderId>      | Latest orders first (# prefix) |
| CustomerEmail | CUSTOMEREMAIL#<email> | CUSTOMEREMAIL#<email> | Uniqueness tracking            |

## CDK Implementation Pattern

```typescript
// lib/stacks/service-stack.ts
const table = new dynamodb.Table(this, "AppTable", {
  partitionKey: {
    name: "PK",
    type: dynamodb.AttributeType.STRING,
  },
  sortKey: {
    name: "SK",
    type: dynamodb.AttributeType.STRING,
  },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
  pointInTimeRecovery: true,
  stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES,
});
```

**Why generic names (PK/SK)?**

- Support multiple entity types in one table
- Avoid confusion when PK means different things per entity
- Enable primary key overloading

### Step 7: Validate Against Access Patterns

Go through your access patterns table and mark which can be satisfied:

| Entity  | Access Pattern          | Index       | ✅ Satisfied?                   |
| ------- | ----------------------- | ----------- | ------------------------------- |
| User    | Get User by username    | Main        | ✅ Direct lookup via PK         |
| Session | Get Sessions for User   | Main        | ✅ Query PK = USER#username     |
| Order   | Get Orders for Customer | Main        | ✅ Query PK = CUSTOMER#username |
| Order   | Get Order by orderId    | ❌ Need GSI | ❌ Not in primary key           |

Patterns not satisfied will need secondary indexes (next phase).

## Key Design Principles

### Client Knowledge Test

> "Will the client have this value at read time?"

If fetching a User profile at `/users/:username`, username is safe for primary key.  
If fetching "my profile" at `/me`, you need to resolve `username` first (from auth context).

### Avoid Timestamp Primary Keys

❌ Don't use:

```
PK: USER#<username>
SK: <timestamp>
```

Client won't know timestamp at read time.

✅ Do use:

```
PK: USER#<username>
SK: USER#<username>
```

Or if timestamp is truly how you access it:

```
PK: READINGS#<deviceId>
SK: <timestamp>
```

### Keep Parent Items Accessible

If using `#ORDER#` prefix to sort children after parent:

```
User item:     PK=USER#alice, SK=USER#alice
Order items:   PK=USER#alice, SK=#ORDER#123
```

Query `PK=USER#alice AND SK BETWEEN USER# AND !` to get just the parent.

## Common Patterns

### Pattern 1: Hierarchical Keys

```
PK: ORG#<orgName>
SK: PROJECT#<projectId>#TASK#<taskId>
```

Enables querying at any level.

### Pattern 2: Composite Sort Keys

```
SK: <Country>#<State>#<City>#<Zip>
```

Query with `begins_with` for different levels.

### Pattern 3: Inverted Indexes

Put child as PK to enable lookups in both directions:

```
Main table:
Parent: PK=PARENT#1, SK=PARENT#1
Child:  PK=PARENT#1, SK=CHILD#A

GSI1:
Child:  GSI1PK=CHILD#A, GSI1SK=PARENT#1
```

## Validation Checklist

- [ ] Primary key ensures uniqueness for all entities
- [ ] All "get by ID" patterns satisfied by primary key
- [ ] Item collections designed for "fetch many" patterns
- [ ] Sort keys handle ordering requirements (ASC/DESC)
- [ ] Prefixes prevent collisions between entity types
- [ ] Client will know PK values at read time
- [ ] Entity chart filled out completely
- [ ] Patterns marked as satisfied or needing GSI
- [ ] Entity chart saved to `docs/specs/jobs/<job>/entity-chart.md`

## Anti-Patterns to Avoid

❌ Using attribute names like `CustomerId`, `OrderId` → Use generic `PK`, `SK`  
❌ Reusing same attribute across multiple indexes → Keep them separate  
❌ CreatedAt timestamps in primary keys unless truly accessed that way  
❌ Forgetting to prefix different entity types → Causes collisions  
❌ Not considering client knowledge at read time → Unusable keys

## Next Steps

1. Identify patterns not satisfied → These need secondary indexes
2. Create TypeScript entity schemas (see Entity Schema Guide)
3. Design secondary indexes (see Secondary Index Strategies Guide)
4. Document data model spec (consolidate entity chart + access patterns)
