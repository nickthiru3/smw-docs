# Guide 2: Access Patterns Definition Guide

## When to Use This Guide

- [ ] Completed ERD
- [ ] Have UI mockups in `docs/specs/jobs/<job>/mockups/`
- [ ] Ready to define concrete data access requirements

## Purpose

Systematically extract and document every way your application will read/write data, following the book's approach of being "specific and thorough".

## Two Approaches to Pattern Discovery

### Approach 1: API-Centric (REST APIs)

Best when building traditional REST endpoints.

**Process:**

1. List all API endpoints from `docs/specs/jobs/<job>/api.yml` (or draft it)
2. For each endpoint, identify:
   - What entities are fetched/modified
   - What filters/sorting are needed
   - Expected response shape

**Example:**

```
GET /users/:username/orders
→ Access Pattern: "Fetch User and recent Orders for User"
→ Returns: User object + array of Order objects
```

### Approach 2: UI-Centric (BFF/SvelteKit)

Best for your serverless architecture with BFFs.

**Process:**

1. Walk through each screen in mockups
2. For each URL/route, identify data requirements
3. Map to entities from ERD

**Example:**

```
Route: /dashboard
Screen shows:
- User profile
- Unread notifications (last 10)
- Recent orders (last 5)

→ Access Patterns:
  1. "Get User by username"
  2. "Get unread Notifications for User"
  3. "Get recent Orders for User"
```

## Pattern Documentation Format

Use the table from `docs/guides/data-modeling/data-access-patterns-table.md`:

| Entity | Access Pattern       | Index | Parameters      | Notes                          |
| ------ | -------------------- | ----- | --------------- | ------------------------------ |
| User   | Get User by username | TBD   | username        | Simple lookup                  |
| Order  | Get Orders for User  | TBD   | username, limit | Latest 20, sorted by date DESC |

**Columns:**

- **Entity**: Primary entity being accessed
- **Access Pattern**: Human-readable description
- **Index**: Filled during primary key design (main table, GSI1, etc.)
- **Parameters**: What the client will provide
- **Notes**: Edge cases, sorting, pagination, filters

## Step-by-Step Process

### Step 1: Identify Write Patterns

Start with create/update/delete operations:

**Questions:**

- What triggers item creation? (User signup, checkout, etc.)
- What attributes change over time?
- What requires transactional guarantees?

**Common patterns:**

- Create X
- Update X
- Delete X (soft vs hard delete)
- Bulk create (e.g., OrderItems in a transaction)

**When to include:**

- If uniqueness is required (e.g., unique email)
- If transactions span multiple items
- If writes have complex business rules

**When to exclude:**

- Simple PutItem operations with no constraints
- Standard CRUD unless there's something interesting

### Step 2: Identify Read Patterns

#### Single-Item Reads

Pattern: "Get X by Y"

**Questions:**

- What identifier will the client have?
- Is this identifier stable? (username vs session token)

**Examples:**

- Get User by username
- Get Session by sessionToken
- Get Order by orderId

#### Multi-Item Reads (Critical!)

Pattern: "Get all X for Y" or "Get latest X"

**Questions:**

- What's the parent entity?
- How many items (bounded <100 or unbounded)?
- Sorting required? Ascending or descending?
- Any filters beyond the parent relationship?

**Examples:**

- Get all Orders for User (unbounded, DESC by date)
- Get active Sessions for User (bounded, no sorting)
- Get last 10 Notifications for User (bounded, DESC by timestamp)

#### Aggregation/Search Patterns

Pattern: "Find X where Y" or "Get all X"

**Questions:**

- Is this a common query or rare analytics?
- Can it be pre-computed?
- Does it need real-time results?

**Examples:**

- Get all Brands (small, stable list)
- Search Products by category
- Count Orders by status

### Step 3: Map to BFF Actions vs Queries

Reference your `docs/specs/jobs/<job>/actions-queries.md`:

**Actions (writes):**

- Return success/failure
- May trigger events
- Often use transactions

**Queries (reads):**

- Return data
- Must be fast
- Drive most of your modeling decisions

### Step 4: Add Time-Based Patterns

Don't forget patterns driven by background jobs:

**Examples:**

- Delete expired Sessions (TTL)
- Archive old Orders
- Send daily digest emails

### Step 5: Validate Completeness

**Checklist:**

- [ ] Every screen in mockups has patterns
- [ ] Every API endpoint has patterns
- [ ] Reviewed with frontend team
- [ ] Reviewed with backend team
- [ ] Edge cases documented in Notes column

**Red flags:**

- "We'll figure this out later" → No, define it now
- Vague patterns like "search functionality" → Be specific
- Missing sorting/pagination details → Document them

## Integration with Your Stack

### SvelteKit Route Mapping

```typescript
// src/routes/users/[username]/+page.server.ts
export async function load({ params }) {
  // Access Pattern: "Get User by username"
  // Access Pattern: "Get recent Orders for User"
}
```

Map each `load` function to patterns.

### Sequence Diagram Cross-Reference

Your sequence diagram should show which patterns are called when:

```
User → BFF → Microservice
     [Get User by username]
     [Get Orders for User]
```

## Common Access Patterns by Relationship Type

### One-to-Many

- Get parent by ID → Primary key lookup
- Get parent + all children → Query with composite key
- Get parent + recent N children → Query with LIMIT

### Many-to-Many

- Get entity + related entities (both directions)
- Check if relationship exists → GetItem with composite key

### Time-Series

- Get latest N items → Query with `ScanIndexForward=false`
- Get items in time range → Query with `BETWEEN` on sort key

## Example: E-Commerce Patterns

```markdown
| Entity    | Access Pattern                | Index | Parameters        | Notes                           |
| --------- | ----------------------------- | ----- | ----------------- | ------------------------------- |
| Customer  | Create Customer               | Main  | username, email   | Must enforce uniqueness on both |
| Customer  | Get Customer by username      | Main  | username          |                                 |
| Customer  | View Customer & Recent Orders | Main  | username          | Last 20 orders, DESC by date    |
| Order     | Create Order                  | Main  | username, orderId | Transaction with OrderItems     |
| Order     | Get Order by orderId          | GSI1  | orderId           | Need index for direct lookup    |
| Order     | Update Order status           | GSI1  | orderId           |                                 |
| OrderItem | Get OrderItems for Order      | GSI1  | orderId           | Part of Order view              |
```

Save to: `docs/specs/jobs/<job>/data-access-patterns.md`

## Validation Checklist

- [ ] All job story scenarios covered
- [ ] All UI screens mapped to patterns
- [ ] All API endpoints mapped to patterns
- [ ] Write patterns identified (especially unique constraints)
- [ ] Multi-item fetch patterns specify sorting
- [ ] Parameters list what client will provide
- [ ] Notes capture edge cases
- [ ] Patterns reviewed with team
- [ ] Table saved to `docs/specs/jobs/<job>/data-access-patterns.md`

## Anti-Patterns to Avoid

❌ "We can just scan the table" → No, define the pattern  
❌ Assuming patterns can be added later → They can't, not easily  
❌ Forgetting sorting requirements → Causes rework later  
❌ Vague filters like "search by keyword" → Be specific

## Next Steps

1. Prioritize patterns (mark "must have" vs "nice to have")
2. Begin primary key design (see Primary Key Design Guide)
3. Update table as you model (fill in Index column)
