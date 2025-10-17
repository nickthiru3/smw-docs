# Guide 8: Migration Strategies for DynamoDB

## Overview

Your DynamoDB data model will evolve over time as your application grows and requirements change. This guide provides practical strategies for migrating your data model safely and efficiently.

**Key Principle:** "Migrations are intimidating at first, but they're entirely manageable".

## When You Need a Migration

Common scenarios requiring migrations:

- Adding new entity types to your application
- Adding new access patterns to existing entities
- Changing how you model relationships
- Fixing incorrect modeling assumptions discovered in production

## Migration Classification

The fundamental question: **Is your migration additive or does it require editing existing items?**

### Additive Migrations

- Start writing new attributes/items without changes to existing data
- Much easier to implement
- Lower risk
- Examples: New entity types, new optional attributes

### Non-Additive Migrations

- Require updating existing items with new attributes
- Usually requires batch ETL job
- Higher complexity and risk
- Examples: New required attributes for indexing, relationship restructuring

---

## Migration Strategy 1: Adding New Attributes to Existing Entities

**Use When:** Adding optional properties to existing entity types (e.g., Birthdate to User, FaxNumber to Business)

**Complexity:** ⭐ Easy

**Is Additive:** Yes ✅

### Why It Works

DynamoDB is schemaless, so you can add attributes without table changes. Handle missing attributes in your application's data access layer.

### Implementation Steps

**Step 1: Update Entity Schema**

Add the new attribute to your TypeScript interface:

```typescript
interface User {
  PK: string;
  SK: string;
  Type: "User";
  Username: string;
  Email: string;
  Name: string;
  Birthdate?: string; // New optional attribute
  CreatedAt: string;
  UpdatedAt: string;
}
```

````

**Step 2: Update Data Access Layer**

Handle missing attributes when reading items:

```typescript
function getUser(username: string): User {
  const response = await dynamodb.getItem({
    TableName: "AppTable",
    Key: {
      PK: { S: `USER#${username}` },
      SK: { S: `USER#${username}` },
    },
  });

  const item = response.Item;

  return {
    username: item.Username.S,
    email: item.Email.S,
    name: item.Name.S,
    birthdate: item.Birthdate?.S, // Handles missing values
    // ... other fields
  };
}
```

**Step 3: Update Write Operations**

Start including the new attribute in writes:

```typescript
function updateUser(user: User): void {
  const item: any = {
    PK: { S: `USER#${user.username}` },
    SK: { S: `USER#${user.username}` },
    Type: { S: "User" },
    Username: { S: user.username },
    Email: { S: user.email },
    Name: { S: user.name },
    UpdatedAt: { S: new Date().toISOString() },
  };

  // Only add Birthdate if provided
  if (user.birthdate) {
    item.Birthdate = { S: user.birthdate };
  }

  await dynamodb.putItem({
    TableName: "AppTable",
    Item: item,
  });
}
```

### When to Use

- ✅ Adding optional application attributes
- ✅ One-to-one relationships (embed as attribute)
- ✅ Bounded one-to-many relationships (list/map attributes)

### Limitations

- ❌ Not suitable for indexing attributes (PK, SK, GSI keys)
- ❌ Won't work for required attributes that affect queries
- ❌ Not for unbounded relationships

### Example: GitHub Code of Conduct

Adding a Code of Conduct to Repos (one-to-one relationship):

```typescript
interface Repo {
  PK: string;
  SK: string;
  Type: "Repo";
  RepoName: string;
  Owner: string;
  CodeOfConduct?: {
    // New attribute
    name: string;
    url: string;
  };
}
```

No ETL needed—just lazily add as users enable this feature.

---

## Migration Strategy 2: Adding New Entity Type Without Relations

**Use When:** Adding new entity type that doesn't need to be grouped with existing entities

**Complexity:** ⭐⭐ Moderate

**Is Additive:** Yes ✅

### Why It Works

New entity goes into separate item collection, so no existing items need modification.

### Implementation Steps

**Step 1: Design Primary Key**

Create PK/SK pattern for new entity:

```markdown
| Entity  | PK Pattern          | SK Pattern          | Notes      |
| ------- | ------------------- | ------------------- | ---------- |
| Project | PROJECT#<projectId> | PROJECT#<projectId> | New entity |
```

**Step 2: Add to Entity Schema**

```typescript
interface Project {
  PK: string; // PROJECT#<projectId>
  SK: string; // PROJECT#<projectId>
  Type: "Project";
  ProjectId: string;
  Name: string;
  Description: string;
  CreatedAt: string;
}
```

**Step 3: Implement CRUD Operations**

```typescript
async function createProject(project: Project): Promise<void> {
  await dynamodb.putItem({
    TableName: "AppTable",
    Item: {
      PK: { S: `PROJECT#${project.projectId}` },
      SK: { S: `PROJECT#${project.projectId}` },
      Type: { S: "Project" },
      ProjectId: { S: project.projectId },
      Name: { S: project.name },
      Description: { S: project.description },
      CreatedAt: { S: new Date().toISOString() },
    },
  });
}

async function getProject(projectId: string): Promise<Project> {
  const response = await dynamodb.getItem({
    TableName: "AppTable",
    Key: {
      PK: { S: `PROJECT#${projectId}` },
      SK: { S: `PROJECT#${projectId}` },
    },
  });

  // Transform to Project object
  return transformToProject(response.Item);
}
```

**Step 4: Update Access Patterns Table**

Add new patterns to your documentation:

```markdown
| Entity  | Access Pattern    | Index | Key Condition              | Parameters       |
| ------- | ----------------- | ----- | -------------------------- | ---------------- |
| Project | Get Project by ID | Main  | PK = PROJECT#<id>          | projectId        |
| Project | List all Projects | Main  | Scan with FilterExpression | Type = 'Project' |
```

### When to Use

- ✅ New feature with independent data
- ✅ No need to fetch with other entities
- ✅ Separate lifecycle from existing entities

### Limitations

- ❌ If you later need relationships, requires another migration
- ❌ Not efficient for "fetch Project with related items" patterns

### Example: SaaS Projects

Adding Projects to a SaaS application with Organizations and Users:

```
Before:
ORG#acme    | ORG#acme       (Organization)
ORG#acme    | USER#alice     (User in org)

After (no changes to existing items):
ORG#acme    | ORG#acme       (Organization)
ORG#acme    | USER#alice     (User in org)
PROJECT#p1  | PROJECT#p1     (New Project - separate collection)
```

---

## Migration Strategy 3: Adding Entity Into Existing Item Collection

**Use When:** New entity has relationship with existing entity and needs to be co-located

**Complexity:** ⭐⭐ Moderate

**Is Additive:** Yes ✅

### Why It Works

Model new items to share partition key with existing items, creating unified item collection. No existing items need updates.

### Implementation Steps

**Step 1: Design Keys to Share Partition**

```markdown
| Entity | PK Pattern      | SK Pattern                | Notes                     |
| ------ | --------------- | ------------------------- | ------------------------- |
| User   | USER#<username> | USER#<username>           | Existing                  |
| Gist   | USER#<username> | GIST#<timestamp>#<gistId> | New - shares PK with User |
```

**Step 2: Add Entity Schema**

```typescript
interface Gist {
  PK: string; // USER#<username>
  SK: string; // GIST#<timestamp>#<gistId>
  Type: "Gist";
  GistId: string;
  Username: string;
  Title: string;
  Content: string;
  CreatedAt: string;
}
```

**Step 3: Implement Access Pattern**

```typescript
async function getGistsForUser(
  username: string,
  limit: number = 10
): Promise<{ user: User; gists: Gist[] }> {
  const response = await dynamodb.query({
    TableName: "AppTable",
    KeyConditionExpression: "PK = :pk AND begins_with(SK, :sk)",
    ExpressionAttributeValues: {
      ":pk": { S: `USER#${username}` },
      ":sk": { S: "GIST#" },
    },
    Limit: limit,
    ScanIndexForward: false, // Most recent first
  });

  const user = response.Items.find((item) => item.Type.S === "User");
  const gists = response.Items.filter((item) => item.Type.S === "Gist");

  return {
    user: transformToUser(user),
    gists: gists.map(transformToGist),
  };
}
```

**Step 4: Update Entity Chart**

```markdown
| Entity | PK Pattern      | SK Pattern                | Notes                         |
| ------ | --------------- | ------------------------- | ----------------------------- |
| User   | USER#<username> | USER#<username>           | Parent entity                 |
| Gist   | USER#<username> | GIST#<timestamp>#<gistId> | Child entity - sorted by time |
```

### When to Use

- ✅ One-to-many relationship between new and existing entity
- ✅ Need to fetch parent + children together
- ✅ Children naturally sorted by time or sequence

### Limitations

- ❌ If existing item collection already large (>10GB), consider separate collection
- ❌ Not suitable if new entity needs to be queried independently often

### Example: GitHub Gists

Adding Gists to Users without modifying existing User items:

```
Before:
USER#alice  | USER#alice     (User)

After (User unchanged, Gists added):
USER#alice  | USER#alice             (User)
USER#alice  | GIST#2020-01-15#g1    (Gist)
USER#alice  | GIST#2020-01-20#g2    (Gist)
```

Query to get User + Gists: Single Query operation.

---

## Migration Strategy 4: Adding Entity Into New Item Collection with ETL

**Use When:** New entity needs separate item collection AND existing items need new attributes for indexing

**Complexity:** ⭐⭐⭐⭐ Complex

**Is Additive:** No ❌ (Requires ETL)

### Why It Works

Create new GSI with new attributes, then run ETL job to decorate existing items.

### Implementation Steps

**Step 1: Design New GSI**

Add GSI to support new access pattern:

```markdown
**GSI2: AccountApps Index**

- GSI2PK: ACCOUNT#<accountId>
- GSI2SK: APP#<appId>
```

**Step 2: Update Table Definition (CDK)**

```typescript
table.addGlobalSecondaryIndex({
  indexName: "GSI2",
  partitionKey: {
    name: "GSI2PK",
    type: AttributeType.STRING,
  },
  sortKey: {
    name: "GSI2SK",
    type: AttributeType.STRING,
  },
  projectionType: ProjectionType.ALL,
});
```

**Step 3: Create ETL Script**

Scan table and add new attributes to existing items:

```typescript
async function migrateAccountsForApps(): Promise<void> {
  let lastEvaluatedKey: any = undefined;

  do {
    // Scan for items needing migration
    const response = await dynamodb.scan({
      TableName: "AppTable",
      FilterExpression: "#type IN (:user, :org)",
      ExpressionAttributeNames: {
        "#type": "Type",
      },
      ExpressionAttributeValues: {
        ":user": { S: "User" },
        ":org": { S: "Organization" },
      },
      ExclusiveStartKey: lastEvaluatedKey,
    });

    // Update each item
    for (const item of response.Items) {
      const accountId = item.Type.S === "User" ? item.Username.S : item.OrgId.S;

      await dynamodb.updateItem({
        TableName: "AppTable",
        Key: {
          PK: item.PK,
          SK: item.SK,
        },
        UpdateExpression: "SET GSI2PK = :gsi2pk, GSI2SK = :gsi2sk",
        ExpressionAttributeValues: {
          ":gsi2pk": { S: `ACCOUNT#${accountId}` },
          ":gsi2sk": { S: `ACCOUNT#${accountId}` },
        },
      });
    }

    lastEvaluatedKey = response.LastEvaluatedKey;
  } while (lastEvaluatedKey);

  console.log("Migration complete");
}
```

**Step 4: Add New Entity Type**

```typescript
interface GitHubApp {
  PK: string; // APP#<appId>
  SK: string; // APP#<appId>
  Type: "GitHubApp";
  AppId: string;
  Name: string;
  // ... other fields
}

interface AppInstallation {
  PK: string; // ACCOUNT#<accountId>
  SK: string; // APP#<appId>
  GSI1PK: string; // APP#<appId>
  GSI1SK: string; // ACCOUNT#<accountId>
  Type: "AppInstallation";
  // ... relationship details
}
```

**Step 5: Implement Access Patterns**

```typescript
// Get Apps installed by Account
async function getAppsForAccount(accountId: string): Promise<GitHubApp[]> {
  const response = await dynamodb.query({
    TableName: "AppTable",
    IndexName: "GSI2",
    KeyConditionExpression: "GSI2PK = :pk AND begins_with(GSI2SK, :sk)",
    ExpressionAttributeValues: {
      ":pk": { S: `ACCOUNT#${accountId}` },
      ":sk": { S: "APP#" },
    },
  });

  return response.Items.map(transformToApp);
}

// Get Accounts that installed App
async function getAccountsForApp(appId: string): Promise<Account[]> {
  const response = await dynamodb.query({
    TableName: "AppTable",
    IndexName: "GSI1",
    KeyConditionExpression: "GSI1PK = :pk AND begins_with(GSI1SK, :sk)",
    ExpressionAttributeValues: {
      ":pk": { S: `APP#${appId}` },
      ":sk": { S: "ACCOUNT#" },
    },
  });

  return response.Items.map(transformToAccount);
}
```

### ETL Best Practices

**1. Use Type Attribute for Filtering**

```typescript
FilterExpression: '#type = :userType',
ExpressionAttributeNames: { '#type': 'Type' },
ExpressionAttributeValues: { ':userType': { S: 'User' } }
```

**2. Parallel Scans for Speed**

```typescript
const TOTAL_SEGMENTS = 10;

async function parallelMigration(segment: number): Promise<void> {
  const response = await dynamodb.scan({
    TableName: "AppTable",
    FilterExpression: "#type = :type",
    TotalSegments: TOTAL_SEGMENTS,
    Segment: segment,
    // ... other parameters
  });

  // Process items...
}

// Run in parallel
Promise.all(
  Array.from({ length: TOTAL_SEGMENTS }, (_, i) => parallelMigration(i))
);
```

**3. Handle LastEvaluatedKey**

```typescript
let lastKey = undefined;
do {
  const response = await dynamodb.scan({
    ExclusiveStartKey: lastKey,
    // ... other parameters
  });

  // Process items

  lastKey = response.LastEvaluatedKey;
} while (lastKey);
```

**4. Batch Updates**

```typescript
const batchSize = 25;
const batches = chunk(itemsToUpdate, batchSize);

for (const batch of batches) {
  await dynamodb.batchWriteItem({
    RequestItems: {
      AppTable: batch.map((item) => ({
        PutRequest: { Item: item },
      })),
    },
  });
}
```

### When to Use

- ✅ Many-to-many relationships requiring new GSI
- ✅ New access pattern needs existing items in new index
- ✅ Relationship requires bidirectional queries

### Limitations

- ❌ Requires downtime or careful coordination
- ❌ Can be slow for large tables
- ❌ Risk of errors during migration
- ❌ May impact table performance during execution

### Example: GitHub Apps

Adding GitHub Apps with many-to-many relationship to Repos:

1. Create GSI2 for Account → App relationship
2. ETL: Add GSI2PK/GSI2SK to existing User and Organization items
3. Create AppInstallation items with adjacency list pattern
4. Support bidirectional queries via main table and GSI1

---

## Migration Strategy 5: Refactoring Existing Access Patterns

**Use When:** Initial modeling assumptions proved incorrect in production

**Complexity:** ⭐⭐⭐⭐⭐ Very Complex

**Is Additive:** No ❌ (Major refactor)

### Why It Works

Complete redesign of how data is accessed, requiring new indexes and full ETL.

### Implementation Steps

**Step 1: Identify the Problem**

Example: Issues and Pull Requests initially modeled together, but need separate filtering by status:

```
Problem:
- Can't efficiently filter "Open Issues" vs "Closed Issues"
- Status changes require delete + recreate workflow
```

**Step 2: Design New Approach**

```markdown
**Before:**

- PK: REPO#<owner>#<repo>
- SK: ISSUE#<number>

**After:**

- PK: REPO#<owner>#<repo>
- SK: ISSUE#<status>#<number>

**New GSI1:**

- GSI1PK: ISSUE#<issueId>
- GSI1SK: ISSUE#<issueId>
```

**Step 3: Add New Indexes**

```typescript
table.addGlobalSecondaryIndex({
  indexName: "IssueById",
  partitionKey: { name: "GSI1PK", type: AttributeType.STRING },
  sortKey: { name: "GSI1SK", type: AttributeType.STRING },
});

table.addGlobalSecondaryIndex({
  indexName: "IssueByStatus",
  partitionKey: { name: "GSI2PK", type: AttributeType.STRING },
  sortKey: { name: "GSI2SK", type: AttributeType.STRING },
});
```

**Step 4: Create Migration Script**

```typescript
async function migrateIssues(): Promise<void> {
  let lastKey = undefined;

  do {
    const response = await dynamodb.scan({
      TableName: "AppTable",
      FilterExpression: "#type IN (:issue, :pr)",
      ExpressionAttributeNames: { "#type": "Type" },
      ExpressionAttributeValues: {
        ":issue": { S: "Issue" },
        ":pr": { S: "PullRequest" },
      },
      ExclusiveStartKey: lastKey,
    });

    for (const item of response.Items) {
      const oldSK = item.SK.S;
      const status = item.Status.S; // 'OPEN' or 'CLOSED'
      const number = oldSK.split("#")[1];

      // Delete old item
      await dynamodb.deleteItem({
        TableName: "AppTable",
        Key: {
          PK: item.PK,
          SK: item.SK,
        },
      });

      // Create new item with updated keys
      await dynamodb.putItem({
        TableName: "AppTable",
        Item: {
          ...item,
          SK: { S: `ISSUE#${status}#${number}` },
          GSI1PK: { S: `ISSUE#${item.IssueId.S}` },
          GSI1SK: { S: `ISSUE#${item.IssueId.S}` },
          GSI2PK: { S: `REPO#${item.RepoId.S}#${status}` },
          GSI2SK: { S: `ISSUE#${number}` },
        },
      });
    }

    lastKey = response.LastEvaluatedKey;
  } while (lastKey);
}
```

**Step 5: Update Access Patterns**

```typescript
// Get Open Issues for Repo
async function getOpenIssues(owner: string, repo: string): Promise<Issue[]> {
  const response = await dynamodb.query({
    TableName: "AppTable",
    KeyConditionExpression: "PK = :pk AND begins_with(SK, :sk)",
    ExpressionAttributeValues: {
      ":pk": { S: `REPO#${owner}#${repo}` },
      ":sk": { S: "ISSUE#OPEN#" },
    },
  });

  return response.Items.map(transformToIssue);
}

// Get Issue by ID
async function getIssueById(issueId: string): Promise<Issue> {
  const response = await dynamodb.query({
    TableName: "AppTable",
    IndexName: "IssueById",
    KeyConditionExpression: "GSI1PK = :pk",
    ExpressionAttributeValues: {
      ":pk": { S: `ISSUE#${issueId}` },
    },
  });

  return transformToIssue(response.Items[0]);
}
```

### Migration Execution Plan

**1. Preparation Phase**

- Design new schema thoroughly
- Create new indexes
- Test migration script on copy of data
- Plan rollback strategy

**2. Execution Phase**

- Enable maintenance mode (if possible)
- Run migration script
- Monitor for errors
- Validate data integrity

**3. Verification Phase**

- Test all access patterns
- Compare before/after data counts
- Performance testing
- Gradual traffic ramp-up

### When to Use

- ✅ Fundamental modeling error discovered
- ✅ Access patterns significantly changed
- ✅ Performance issues with current design
- ✅ Have capacity for significant engineering effort

### Limitations

- ❌ High risk of data loss or corruption
- ❌ Requires significant downtime or dual-write period
- ❌ Expensive in terms of engineering time
- ❌ May require client updates

---

## Migration Decision Matrix

| Scenario                        | Strategy   | Additive? | ETL Required? | Complexity | Risk   |
| ------------------------------- | ---------- | --------- | ------------- | ---------- | ------ |
| Add optional attribute          | Strategy 1 | ✅ Yes    | ❌ No         | Low        | Low    |
| Add independent entity          | Strategy 2 | ✅ Yes    | ❌ No         | Low        | Low    |
| Add related entity (co-located) | Strategy 3 | ✅ Yes    | ❌ No         | Medium     | Low    |
| Add entity (new collection)     | Strategy 4 | ❌ No     | ✅ Yes        | High       | Medium |
| Refactor existing pattern       | Strategy 5 | ❌ No     | ✅ Yes        | Very High  | High   |

---

## Best Practices

### 1. Plan Migrations Carefully

- Document current and target state
- List all affected access patterns
- Create rollback plan
- Test on non-production data first

### 2. Use Type Attribute for Filtering

Always include Type attribute on items—makes ETL filtering much easier:

```typescript
FilterExpression: '#type = :type',
ExpressionAttributeNames: { '#type': 'Type' },
ExpressionAttributeValues: { ':type': { S: 'User' } }
```

### 3. Implement Monitoring

```typescript
let processedCount = 0;
let errorCount = 0;

try {
  // Process item
  processedCount++;
} catch (error) {
  errorCount++;
  console.error("Migration error:", error);
}

console.log(`Processed: ${processedCount}, Errors: ${errorCount}`);
```

### 4. Handle Partial Failures

```typescript
async function migrateWithRetry(item: any, maxRetries = 3): Promise<void> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await updateItem(item);
      return;
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      await sleep(1000 * Math.pow(2, attempt)); // Exponential backoff
    }
  }
}
```

### 5. Consider Dual-Write Period

For critical systems, implement dual-write to both old and new patterns during migration:

```typescript
async function saveDuringMigration(item: Item): Promise<void> {
  // Write to old pattern
  await saveOldPattern(item);

  // Write to new pattern
  try {
    await saveNewPattern(item);
  } catch (error) {
    console.error("New pattern write failed:", error);
    // Don't fail - we're in transition
  }
}
```

---

## Summary Table

| Migration Type              | Strategy            | Example          | Additive | ETL | References |
| --------------------------- | ------------------- | ---------------- | -------- | --- | ---------- |
| New application attribute   | Add lazily          | User birthdate   | ✅       | ❌  |            |
| New entity (no relations)   | Separate collection | Projects         | ✅       | ❌  |            |
| New entity (co-located)     | Share partition key | Gists with Users | ✅       | ❌  |            |
| New entity (new collection) | GSI + ETL           | GitHub Apps      | ❌       | ✅  |            |
| Refactor pattern            | New indexes + ETL   | Issues by status | ❌       | ✅  |            |

---

## Anti-Patterns to Avoid

### 1. Not Planning for Migrations

❌ **Bad:** "We'll figure out migrations when we need them"
✅ **Good:** Understand migration strategies during initial design

### 2. Modifying Items Without Type Filtering

❌ **Bad:** Scan all items and update everything
✅ **Good:** Use FilterExpression with Type attribute

### 3. Not Testing Migration Scripts

❌ **Bad:** Run script directly on production
✅ **Good:** Test on copy of production data first

### 4. Not Monitoring Progress

❌ **Bad:** Start migration and walk away
✅ **Good:** Log progress, errors, and performance metrics

### 5. Not Having Rollback Plan

❌ **Bad:** "We'll deal with failures if they happen"
✅ **Good:** Document rollback procedure before starting

---

## Additional Resources

- [DynamoDB Book Chapter 15: Migration Strategies](reference)
- [DynamoDB Book Chapter 22: GitHub Migration Example](reference)
- [Guide 1: ERD Creation](guide-1-erd-creation.md)
- [Guide 2: Access Patterns Definition](guide-2-access-patterns.md)
- [Guide 7: Anti-Patterns](guide-7-anti-patterns.md)

---

## Checklist: Before Running Migration

- [ ] Current data model documented
- [ ] Target data model documented
- [ ] All affected access patterns identified
- [ ] Migration script written and reviewed
- [ ] Test run completed on copy of data
- [ ] Rollback procedure documented
- [ ] Monitoring and logging implemented
- [ ] Team informed of migration schedule
- [ ] Backup of current data created
- [ ] Post-migration validation plan ready
````
