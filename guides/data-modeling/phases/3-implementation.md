# Phase 3: Implementation

## Overview

Phase 3 is where you translate your carefully designed data model into production code. You've completed your ERD, documented all access patterns, designed your primary keys and secondary indexes, and validated against anti-patterns. Now it's time to build.

**Key Principle:** "Implement your data model at the very boundary of your application".

## Objectives

By the end of Phase 3, you should have:

- [ ] DynamoDB table(s) created via infrastructure-as-code
- [ ] Data access layer implemented
- [ ] All access patterns coded and tested
- [ ] Scripts for debugging and data exploration
- [ ] Monitoring and alerting configured
- [ ] Documentation updated with implementation details

## Prerequisites

Before starting Phase 3:

- [ ] Phase 2 complete and approved
- [ ] Entity charts finalized for all indexes
- [ ] Access patterns table completely filled out
- [ ] Anti-pattern review passed
- [ ] Team sign-off on design

## Steps

### Step 1: Create Table with Infrastructure-as-Code

**Duration:** 1 day

**Use Guide:** Infrastructure-as-code tools documentation

**Activities:**

1. Define table in CloudFormation/CDK/Terraform
2. Configure primary key
3. Add secondary indexes
4. Set up billing mode
5. Configure streams (if needed)
6. Add TTL configuration (if needed)

**CDK Example:**

```typescript
import * as dynamodb from "aws-cdk-lib/aws-dynamodb";
import * as cdk from "aws-cdk-lib";

export class DataStack extends cdk.Stack {
  public readonly table: dynamodb.Table;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create table with composite primary key
    this.table = new dynamodb.Table(this, "AppTable", {
      tableName: "AppTable",
      partitionKey: {
        name: "PK",
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: "SK",
        type: dynamodb.AttributeType.STRING,
      },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // Start with on-demand
      pointInTimeRecovery: true, // Enable backups
      stream: dynamodb.StreamViewType.NEW_AND_OLD_IMAGES, // Enable streams
      removalPolicy: cdk.RemovalPolicy.RETAIN, // Protect against accidental deletion
    });

    // Add GSI1
    this.table.addGlobalSecondaryIndex({
      indexName: "GSI1",
      partitionKey: {
        name: "GSI1PK",
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: "GSI1SK",
        type: dynamodb.AttributeType.STRING,
      },
      projectionType: dynamodb.ProjectionType.ALL,
    });

    // Add GSI2 (if needed)
    this.table.addGlobalSecondaryIndex({
      indexName: "GSI2",
      partitionKey: {
        name: "GSI2PK",
        type: dynamodb.AttributeType.STRING,
      },
      sortKey: {
        name: "GSI2SK",
        type: dynamodb.AttributeType.STRING,
      },
      projectionType: dynamodb.ProjectionType.ALL,
    });

    // Configure TTL
    this.table.addTimeToLive({
      attributeName: "TTL",
    });
  }
}
```

**CloudFormation Example:**

```yaml
Resources:
  AppTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: AppTable
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: PK
          AttributeType: S
        - AttributeName: SK
          AttributeType: S
        - AttributeName: GSI1PK
          AttributeType: S
        - AttributeName: GSI1SK
          AttributeType: S
      KeySchema:
        - AttributeName: PK
          KeyType: HASH
        - AttributeName: SK
          KeyType: RANGE
      GlobalSecondaryIndexes:
        - IndexName: GSI1
          KeySchema:
            - AttributeName: GSI1PK
              KeyType: HASH
            - AttributeName: GSI1SK
              KeyType: RANGE
          Projection:
            ProjectionType: ALL
      StreamSpecification:
        StreamViewType: NEW_AND_OLD_IMAGES
      TimeToLiveSpecification:
        AttributeName: TTL
        Enabled: true
      PointInTimeRecoverySpecification:
        PointInTimeRecoveryEnabled: true
```

**Validation Checklist:**

- [ ] Table name matches documentation
- [ ] Primary key attributes match entity chart
- [ ] All GSIs defined with correct key schema
- [ ] Billing mode appropriate for workload
- [ ] Streams enabled if needed
- [ ] TTL configured if needed
- [ ] Backups enabled
- [ ] Deletion protection configured

**Output:**

- Infrastructure-as-code files committed to repository
- Table deployed to development environment

---

### Step 2: Implement Data Access Layer

**Duration:** 3-5 days

**Use Guide:** [Guide 4: Entity Schema Guide](../guides/4-entity-schema.md)

**Activities:**

1. Create data access module/package
2. Implement TypeScript interfaces for entities
3. Create helper functions for key generation
4. Implement CRUD operations for each entity
5. Implement query operations for access patterns
6. Add proper error handling
7. Write unit tests

**Architecture Pattern:**

The data access layer should sit at the boundary of your application:

```
┌─────────────────────────────────────────┐
│         Application Logic               │
│  (Business rules, API handlers, etc.)   │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         Data Access Layer               │
│  (DynamoDB operations, key generation)  │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│            DynamoDB Table               │
└─────────────────────────────────────────┘
```

**Key Principles:**

1. **Separation of Concerns**

   - Application code works with domain objects
   - Data layer handles DynamoDB-specific concerns
   - No DynamoDB code scattered throughout application

2. **Separate Indexing from Application Attributes**
   - Keep PK, SK, GSI1PK, GSI1SK separate from domain data
   - Makes migrations easier
   - Clearer separation of concerns

**File Structure:**

```
src/
├── entities/
│   ├── user.ts           # User entity interface & helpers
│   ├── order.ts          # Order entity interface & helpers
│   └── product.ts        # Product entity interface & helpers
├── data-access/
│   ├── dynamodb.ts       # DynamoDB client setup
│   ├── keys.ts           # Key generation helpers
│   ├── users.ts          # User CRUD operations
│   ├── orders.ts         # Order CRUD operations
│   └── products.ts       # Product CRUD operations
└── api/
    └── handlers.ts       # API route handlers
```

**Entity Interfaces Example:**

```typescript
// src/entities/user.ts

export interface User {
  username: string;
  email: string;
  name: string;
  createdAt: string;
  updatedAt: string;
}

// Internal DynamoDB item structure
export interface UserItem {
  PK: string; // USER#<username>
  SK: string; // USER#<username>
  Type: "User";
  Username: string;
  Email: string;
  Name: string;
  CreatedAt: string;
  UpdatedAt: string;
  GSI1PK?: string; // EMAIL#<email>
  GSI1SK?: string; // EMAIL#<email>
}
```

**Key Generation Helpers:**

```typescript
// src/data-access/keys.ts

export const keys = {
  user: {
    primary: (username: string) => ({
      PK: `USER#${username}`,
      SK: `USER#${username}`,
    }),
    byEmail: (email: string) => ({
      GSI1PK: `EMAIL#${email}`,
      GSI1SK: `EMAIL#${email}`,
    }),
  },

  order: {
    primary: (orderId: string) => ({
      PK: `ORDER#${orderId}`,
      SK: `ORDER#${orderId}`,
    }),
    byCustomer: (customerId: string, timestamp: string, orderId: string) => ({
      PK: `CUSTOMER#${customerId}`,
      SK: `ORDER#${timestamp}#${orderId}`,
    }),
    byStatus: (status: string, timestamp: string, orderId: string) => ({
      GSI1PK: `STATUS#${status}`,
      GSI1SK: `${timestamp}#${orderId}`,
    }),
  },
};
```

**CRUD Operations Example:**

```typescript
// src/data-access/users.ts

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  PutCommand,
  GetCommand,
  UpdateCommand,
  DeleteCommand,
} from "@aws-sdk/lib-dynamodb";
import { User, UserItem } from "../entities/user";
import { keys } from "./keys";

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);
const TABLE_NAME = process.env.TABLE_NAME!;

// Transform domain object to DynamoDB item
function toItem(user: User): UserItem {
  return {
    ...keys.user.primary(user.username),
    ...keys.user.byEmail(user.email),
    Type: "User",
    Username: user.username,
    Email: user.email,
    Name: user.name,
    CreatedAt: user.createdAt,
    UpdatedAt: user.updatedAt,
  };
}

// Transform DynamoDB item to domain object
function fromItem(item: UserItem): User {
  return {
    username: item.Username,
    email: item.Email,
    name: item.Name,
    createdAt: item.CreatedAt,
    updatedAt: item.UpdatedAt,
  };
}

export async function createUser(user: User): Promise<User> {
  const item = toItem(user);

  await docClient.send(
    new PutCommand({
      TableName: TABLE_NAME,
      Item: item,
      ConditionExpression: "attribute_not_exists(PK)", // Prevent overwrites
    })
  );

  return user;
}

export async function getUserByUsername(
  username: string
): Promise<User | null> {
  const result = await docClient.send(
    new GetCommand({
      TableName: TABLE_NAME,
      Key: keys.user.primary(username),
    })
  );

  return result.Item ? fromItem(result.Item as UserItem) : null;
}

export async function getUserByEmail(email: string): Promise<User | null> {
  const result = await docClient.send(
    new QueryCommand({
      TableName: TABLE_NAME,
      IndexName: "GSI1",
      KeyConditionExpression: "GSI1PK = :pk",
      ExpressionAttributeValues: {
        ":pk": `EMAIL#${email}`,
      },
      Limit: 1,
    })
  );

  if (!result.Items || result.Items.length === 0) {
    return null;
  }

  return fromItem(result.Items[0] as UserItem);
}

export async function updateUser(
  username: string,
  updates: Partial<User>
): Promise<User> {
  const now = new Date().toISOString();

  // Build update expression dynamically
  const updateExpressions: string[] = ["UpdatedAt = :updatedAt"];
  const expressionAttributeValues: Record<string, any> = {
    ":updatedAt": now,
  };

  if (updates.name) {
    updateExpressions.push("Name = :name");
    expressionAttributeValues[":name"] = updates.name;
  }

  if (updates.email) {
    updateExpressions.push("Email = :email");
    updateExpressions.push("GSI1PK = :gsi1pk");
    updateExpressions.push("GSI1SK = :gsi1sk");
    expressionAttributeValues[":email"] = updates.email;
    expressionAttributeValues[":gsi1pk"] = `EMAIL#${updates.email}`;
    expressionAttributeValues[":gsi1sk"] = `EMAIL#${updates.email}`;
  }

  const result = await docClient.send(
    new UpdateCommand({
      TableName: TABLE_NAME,
      Key: keys.user.primary(username),
      UpdateExpression: `SET ${updateExpressions.join(", ")}`,
      ExpressionAttributeValues: expressionAttributeValues,
      ReturnValues: "ALL_NEW",
    })
  );

  return fromItem(result.Attributes as UserItem);
}

export async function deleteUser(username: string): Promise<void> {
  await docClient.send(
    new DeleteCommand({
      TableName: TABLE_NAME,
      Key: keys.user.primary(username),
    })
  );
}
```

**Query Operations Example:**

```typescript
// src/data-access/orders.ts

import { QueryCommand } from "@aws-sdk/lib-dynamodb";
import { Order, OrderItem } from "../entities/order";
import { keys } from "./keys";

export async function getOrdersForCustomer(
  customerId: string,
  limit: number = 20
): Promise<Order[]> {
  const result = await docClient.send(
    new QueryCommand({
      TableName: TABLE_NAME,
      KeyConditionExpression: "PK = :pk AND begins_with(SK, :sk)",
      ExpressionAttributeValues: {
        ":pk": `CUSTOMER#${customerId}`,
        ":sk": "ORDER#",
      },
      Limit: limit,
      ScanIndexForward: false, // Most recent first
    })
  );

  return (result.Items || []).map((item) => fromItem(item as OrderItem));
}

export async function getOrdersByStatus(
  status: string,
  limit: number = 50
): Promise<Order[]> {
  const result = await docClient.send(
    new QueryCommand({
      TableName: TABLE_NAME,
      IndexName: "GSI1",
      KeyConditionExpression: "GSI1PK = :pk",
      ExpressionAttributeValues: {
        ":pk": `STATUS#${status}`,
      },
      Limit: limit,
      ScanIndexForward: false,
    })
  );

  return (result.Items || []).map((item) => fromItem(item as OrderItem));
}
```

**Error Handling:**

```typescript
import { ConditionalCheckFailedException } from "@aws-sdk/client-dynamodb";

export class UserAlreadyExistsError extends Error {
  constructor(username: string) {
    super(`User with username ${username} already exists`);
    this.name = "UserAlreadyExistsError";
  }
}

export async function createUser(user: User): Promise<User> {
  try {
    const item = toItem(user);

    await docClient.send(
      new PutCommand({
        TableName: TABLE_NAME,
        Item: item,
        ConditionExpression: "attribute_not_exists(PK)",
      })
    );

    return user;
  } catch (error) {
    if (error instanceof ConditionalCheckFailedException) {
      throw new UserAlreadyExistsError(user.username);
    }
    throw error;
  }
}
```

**Validation Checklist:**

- [ ] All entity interfaces defined
- [ ] Key generation helpers centralized
- [ ] CRUD operations for each entity
- [ ] Query operations for each access pattern
- [ ] Proper error handling
- [ ] Transform functions (toItem/fromItem) implemented
- [ ] No DynamoDB code in application logic
- [ ] Unit tests written

**Output:**

- Complete data access layer implementation
- Unit tests passing

---

### Step 3: Write Debugging Scripts

**Duration:** 1 day

**Use Guide:** [Guide 9: Implementation Tips](../guides/9-implementation-tips.md) (from DynamoDB Book)

**Activities:**

1. Create scripts to test each access pattern
2. Add data seeding scripts
3. Create table exploration tools
4. Document usage of scripts

**Why Debugging Scripts?**

From the DynamoDB Book:

- Console view is difficult with generic key names (PK, SK)
- Need to validate access patterns work correctly
- Useful for onboarding new team members
- Essential for troubleshooting production issues

**Example Script Structure:**

```
scripts/
├── seed-data.ts          # Populate table with test data
├── test-patterns.ts      # Test all access patterns
├── explore-table.ts      # Browse table contents
└── validate-model.ts     # Validate data model consistency
```

**Seed Data Script:**

```typescript
// scripts/seed-data.ts

import { createUser } from "../src/data-access/users";
import { createOrder } from "../src/data-access/orders";

async function seedData() {
  console.log("Seeding test data...");

  // Create test users
  const users = [
    {
      username: "alice",
      email: "alice@example.com",
      name: "Alice Smith",
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    },
    {
      username: "bob",
      email: "bob@example.com",
      name: "Bob Jones",
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    },
  ];

  for (const user of users) {
    await createUser(user);
    console.log(`✓ Created user: ${user.username}`);
  }

  // Create test orders
  // ... similar pattern

  console.log("✓ Seed data complete");
}

seedData().catch(console.error);
```

**Test Access Patterns Script:**

```typescript
// scripts/test-patterns.ts

import { getUserByUsername, getUserByEmail } from "../src/data-access/users";
import {
  getOrdersForCustomer,
  getOrdersByStatus,
} from "../src/data-access/orders";

async function testAccessPatterns() {
  console.log("Testing access patterns...\n");

  // Test: Get User by Username
  console.log("1. Get User by Username");
  const user1 = await getUserByUsername("alice");
  console.log(`   Result: ${user1 ? user1.name : "Not found"}`);
  console.log(`   ✓ Pattern works\n`);

  // Test: Get User by Email
  console.log("2. Get User by Email");
  const user2 = await getUserByEmail("alice@example.com");
  console.log(`   Result: ${user2 ? user2.name : "Not found"}`);
  console.log(`   ✓ Pattern works\n`);

  // Test: Get Orders for Customer
  console.log("3. Get Orders for Customer");
  const orders = await getOrdersForCustomer("C123", 10);
  console.log(`   Result: ${orders.length} orders found`);
  console.log(`   ✓ Pattern works\n`);

  // ... test remaining patterns

  console.log("✓ All access patterns validated");
}

testAccessPatterns().catch(console.error);
```

**Table Exploration Script:**

```typescript
// scripts/explore-table.ts

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, ScanCommand } from "@aws-sdk/lib-dynamodb";

const client = DynamoDBDocumentClient.from(new DynamoDBClient({}));

async function exploreTable(entityType?: string) {
  const params: any = {
    TableName: process.env.TABLE_NAME!,
    Limit: 20,
  };

  if (entityType) {
    params.FilterExpression = "#type = :type";
    params.ExpressionAttributeNames = { "#type": "Type" };
    params.ExpressionAttributeValues = { ":type": entityType };
  }

  const result = await client.send(new ScanCommand(params));

  console.log(`Found ${result.Items?.length || 0} items:\n`);
  result.Items?.forEach((item, index) => {
    console.log(`${index + 1}. ${item.Type} - PK: ${item.PK}, SK: ${item.SK}`);
    console.log(`   ${JSON.stringify(item, null, 2)}\n`);
  });
}

// Usage: npm run explore-table [entityType]
const entityType = process.argv[2];
exploreTable(entityType).catch(console.error);
```

**Validation Checklist:**

- [ ] Seed data script created
- [ ] Test script for each access pattern
- [ ] Table exploration tool created
- [ ] Scripts documented in README
- [ ] All access patterns tested successfully

**Output:**

- `scripts/` directory with debugging tools
- Updated README with script usage

---

### Step 4: Add Monitoring and Alerting

**Duration:** 1-2 days

**Activities:**

1. Set up CloudWatch alarms
2. Configure metrics collection
3. Add application-level logging
4. Create operational dashboards
5. Document alert responses

**Key Metrics to Monitor:**

1. **Throttling Metrics**

   - `UserErrors` (400-level errors)
   - `SystemErrors` (500-level errors)
   - `ConsumedReadCapacityUnits`
   - `ConsumedWriteCapacityUnits`

2. **Performance Metrics**

   - `SuccessfulRequestLatency`
   - `ReturnedItemCount`

3. **Operational Metrics**
   - `ConditionalCheckFailedRequests`
   - `TransactionConflict`

**CloudWatch Alarm Example (CDK):**

```typescript
import * as cloudwatch from "aws-cdk-lib/aws-cloudwatch";
import * as actions from "aws-cdk-lib/aws-cloudwatch-actions";
import * as sns from "aws-cdk-lib/aws-sns";

// Create SNS topic for alerts
const alertTopic = new sns.Topic(this, "DynamoDBAlerts");

// Alarm for throttled reads
new cloudwatch.Alarm(this, "ReadThrottleAlarm", {
  metric: table.metricUserErrors({
    dimensions: {
      TableName: table.tableName,
      Operation: "GetItem",
    },
  }),
  threshold: 10,
  evaluationPeriods: 2,
  alarmDescription: "Alert when read requests are throttled",
});
alertTopic.addSubscription(
  new subscriptions.EmailSubscription("team@example.com")
);
```

**Application Logging:**

```typescript
// Add structured logging to data access layer
import * as winston from "winston";

const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [new winston.transports.Console()],
});

export async function getUserByUsername(
  username: string
): Promise<User | null> {
  const startTime = Date.now();

  try {
    const result = await docClient.send(
      new GetCommand({
        TableName: TABLE_NAME,
        Key: keys.user.primary(username),
      })
    );

    const duration = Date.now() - startTime;
    logger.info("GetUser success", {
      operation: "getUserByUsername",
      username,
      found: !!result.Item,
      durationMs: duration,
    });

    return result.Item ? fromItem(result.Item as UserItem) : null;
  } catch (error) {
    const duration = Date.now() - startTime;
    logger.error("GetUser failed", {
      operation: "getUserByUsername",
      username,
      error: error.message,
      durationMs: duration,
    });
    throw error;
  }
}
```

**Validation Checklist:**

- [ ] CloudWatch alarms configured
- [ ] Metrics dashboard created
- [ ] Application logging added
- [ ] Alert response playbook documented
- [ ] Team notified of alerts

**Output:**

- CloudWatch alarms configured
- Operational dashboard
- Alert response documentation

---

### Step 5: Deploy and Test

**Duration:** 2-3 days

**Activities:**

1. Deploy to development environment
2. Run integration tests
3. Load testing (if applicable)
4. Deploy to staging
5. User acceptance testing
6. Deploy to production
7. Monitor and iterate

**Deployment Checklist:**

**Development:**

- [ ] Infrastructure deployed via IaC
- [ ] Application deployed
- [ ] Seed data loaded
- [ ] All access patterns tested
- [ ] Debugging scripts validated

**Staging:**

- [ ] Infrastructure deployed
- [ ] Application deployed
- [ ] Production-like data loaded
- [ ] Performance testing completed
- [ ] Monitoring validated
- [ ] User acceptance testing passed

**Production:**

- [ ] Infrastructure deployed
- [ ] Application deployed (blue/green or canary)
- [ ] Monitoring active
- [ ] Alerts configured
- [ ] Rollback plan ready
- [ ] Team on standby for first 24 hours

**Integration Testing:**

```typescript
// tests/integration/users.test.ts

import {
  createUser,
  getUserByUsername,
  getUserByEmail,
} from "../../src/data-access/users";

describe("User CRUD Operations", () => {
  test("Create and retrieve user", async () => {
    const user = {
      username: "testuser",
      email: "test@example.com",
      name: "Test User",
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };

    await createUser(user);

    const retrieved = await getUserByUsername("testuser");
    expect(retrieved).toMatchObject(user);
  });

  test("Get user by email", async () => {
    const retrieved = await getUserByEmail("test@example.com");
    expect(retrieved?.username).toBe("testuser");
  });

  test("Prevent duplicate users", async () => {
    const user = {
      username: "testuser",
      email: "another@example.com",
      name: "Test User",
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };

    await expect(createUser(user)).rejects.toThrow(UserAlreadyExistsError);
  });
});
```

**Load Testing (Artillery Example):**

```yaml
# load-test.yml
config:
  target: "https://api.example.com"
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 300
      arrivalRate: 50
      name: "Sustained load"
    - duration: 120
      arrivalRate: 100
      name: "Spike"

scenarios:
  - name: "Get user by username"
    flow:
      - get:
          url: "/users/{{ $randomString() }}"
```

**Validation Checklist:**

- [ ] All environments deployed successfully
- [ ] Integration tests passing
- [ ] Load tests completed (if applicable)
- [ ] No performance regressions
- [ ] Monitoring working correctly
- [ ] Team trained on new system

**Output:**

- Application deployed to production
- All tests passing
- Monitoring active

---

## Phase 3 Deliverables

Upon completion of Phase 3, you should have:

1. **Infrastructure**

   - DynamoDB table(s) defined in IaC
   - Deployed to all environments

2. **Data Access Layer**

   - Complete implementation of all CRUD operations
   - All access patterns implemented
   - Unit tests passing

3. **Debugging Tools**

   - Seed data scripts
   - Access pattern test scripts
   - Table exploration tools

4. **Monitoring**

   - CloudWatch alarms configured
   - Operational dashboard
   - Application logging

5. **Documentation**
   - Updated access patterns table with implementation notes
   - API documentation
   - Alert response playbook

## Common Pitfalls

### 1. Not Separating Indexing from Application Attributes

**Problem:** Mixing PK/SK values with domain data in application code  
**Solution:** Keep separate transform functions (toItem/fromItem)

### 2. Data Access Scattered Throughout Application

**Problem:** DynamoDB calls mixed with business logic  
**Solution:** Centralize all data access in dedicated layer

### 3. Missing Error Handling

**Problem:** Not handling DynamoDB-specific errors properly  
**Solution:** Wrap operations and translate to domain errors

### 4. No Debugging Scripts

**Problem:** Can't validate access patterns work correctly  
**Solution:** Create test scripts for each pattern upfront

### 5. Skipping Load Testing

**Problem:** Discover performance issues in production  
**Solution:** Load test in staging with production-like data

## Best Practices

### 1. Keep Data Access at Application Boundary

```
✅ Good:
API Handler → Data Access Layer → DynamoDB

❌ Bad:
API Handler → Business Logic → DynamoDB calls scattered everywhere
```

### 2. Use Consistent Naming Conventions

- Entity interfaces: `User`, `Order`, `Product`
- DynamoDB items: `UserItem`, `OrderItem`, `ProductItem`
- Keys: `keys.user.primary()`, `keys.order.byCustomer()`

### 3. Always Include Type Attribute

Makes debugging and ETL much easier:

```typescript
{
  PK: 'USER#alice',
  SK: 'USER#alice',
  Type: 'User',  // Always include!
  // ... other attributes
}
```

### 4. Write Scripts Early

Don't wait until you have problems:

- Write debugging scripts as you implement patterns
- Use them during development
- Share with team for onboarding

### 5. Start with On-Demand Pricing

Then switch to provisioned once traffic is predictable:

- Lower risk during development
- Easier to handle unexpected spikes
- Switch to provisioned to optimize costs

## Moving Forward

After Phase 3 completion:

- [ ] Monitor production metrics closely
- [ ] Gather user feedback
- [ ] Document lessons learned
- [ ] Plan for Phase 4 (optimization & scaling)

---

## Additional Resources

- [Guide 4: Entity Schema Guide](../guides/4-entity-schema.md)
- [Guide 7: Anti-Patterns Guide](../guides/7-common-anti-patterns.md)
- [Guide 8: Migration Strategies](../guides/8-migration-strategies.md)
- [Quick Reference Cards](../quick-reference-cards.md)
- DynamoDB Book Chapter 9: "From modeling to implementation"
- AWS SDK Documentation
- AWS CDK/CloudFormation Documentation
