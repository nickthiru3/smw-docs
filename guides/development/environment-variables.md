# Environment Variables Guide

A comprehensive guide to understanding and using environment variables across different contexts in the microservices architecture.

---

## Table of Contents

1. [Overview: Two Separate Worlds](#overview-two-separate-worlds)
2. [World 1: CDK Synthesis Time](#world-1-cdk-synthesis-time)
3. [World 2: Lambda Runtime](#world-2-lambda-runtime)
4. [Testing Contexts](#testing-contexts)
5. [Common Questions & Answers](#common-questions--answers)
6. [Best Practices](#best-practices)
7. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
8. [Troubleshooting](#troubleshooting)

---

## Overview: Two Separate Worlds

Environment variables serve different purposes in different execution contexts. Understanding these contexts is critical to avoiding confusion and bugs.

### The Two Worlds

| Context | When | Where | Purpose |
|---------|------|-------|---------|
| **CDK Synthesis** | Running `cdk synth/deploy` | Local machine / CI/CD | Configure infrastructure |
| **Lambda Runtime** | Lambda executes in AWS | AWS Lambda environment | Configure application behavior |

**Key Insight:** These are completely separate execution environments with different environment variables!

---

## World 1: CDK Synthesis Time

### What is CDK Synthesis?

CDK synthesis is when your TypeScript code generates CloudFormation templates. This happens on your local machine or in CI/CD when you run:
```bash
cdk synth
cdk deploy
```

### Environment Variable Flow

```
.env file → dotenv.config() → process.env → config object → CDK constructs
```

### Step-by-Step Example

#### 1. `.env` File (Local Development)
```bash
# .env
ENV_NAME=dev
AWS_ACCOUNT_ID=123456789012
AWS_REGION=us-east-1
SERVICE_NAME=users-ms
GITHUB_REPO=owner/repo
```

#### 2. `bin/app.ts` - Load Environment Variables
```typescript
// Load environment variables FIRST before any other imports
import * as dotenv from "dotenv";
dotenv.config(); // Loads .env into process.env

import { App } from "aws-cdk-lib";
import { ServiceStack } from "#lib/service-stack";
import appConfig from "#config/default";

const app = new App();
const config = appConfig; // Config object reads from process.env

new ServiceStack(app, 'Stack', {
  env: { account: config.accountId, region: config.region },
  config, // Pass config to constructs
});
```

#### 3. `config/default.ts` - Single Source of Truth
```typescript
/**
 * Configuration object that reads from process.env at CDK synthesis time.
 * This is the ONLY place where CDK code should read process.env.
 */
const defaultConfig: IConfig = {
  envName: process.env.ENV_NAME || "local",
  accountId: process.env.AWS_ACCOUNT_ID || throwError("AWS_ACCOUNT_ID required"),
  region: process.env.AWS_REGION || throwError("AWS_REGION required"),
  service: {
    name: process.env.SERVICE_NAME || "users-ms",
    displayName: process.env.SERVICE_DISPLAY_NAME || "Microservice",
  },
  // ... more config
};

export default loadConfig(); // Validates and returns config
```

#### 4. CDK Constructs - Use Config Object
```typescript
import type { IConfig } from "#config/default";

interface IMyConstructProps {
  readonly config: IConfig; // Receive config as prop
}

class MyConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IMyConstructProps) {
    super(scope, id);
    
    const { config } = props;
    
    // ✅ CORRECT: Read from config object
    const region = config.region;
    const serviceName = config.service.name;
    
    // ❌ WRONG: Don't read process.env directly in constructs
    // const region = process.env.AWS_REGION;
    
    // Use config values to create resources
    new Bucket(this, 'Bucket', {
      bucketName: `${serviceName}-${region}-bucket`,
    });
  }
}
```

### Why This Pattern?

**Benefits:**
- **Single source of truth** - Config object is the only place reading `process.env`
- **Type safety** - Config object has TypeScript types
- **Validation** - Config validates required values at startup
- **Testability** - Easy to mock config object in tests
- **Clarity** - Clear what configuration is available

---

## World 2: Lambda Runtime

### What is Lambda Runtime?

Lambda runtime is when your handler code executes in AWS Lambda. This happens when:
- API Gateway receives a request
- EventBridge triggers a scheduled function
- SNS/SQS delivers a message

### Environment Variable Flow

```
CDK sets env vars → Lambda process.env → Handler reads directly
```

### Step-by-Step Example

#### 1. CDK Construct - Set Lambda Environment Variables
```typescript
import { NodejsFunction } from "aws-cdk-lib/aws-lambda-nodejs";

class ApiConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IApiConstructProps) {
    super(scope, id);
    
    const { userPool, table, config } = props;
    
    // ✅ CORRECT: CDK sets environment variables for Lambda
    const handler = new NodejsFunction(this, 'Handler', {
      entry: 'lib/api/endpoints/users/post/handler.ts',
      environment: {
        // These are available in Lambda as process.env
        USER_POOL_ID: userPool.userPoolId,
        USER_POOL_CLIENT_ID: userPool.userPoolClientId,
        TABLE_NAME: table.tableName,
        REGION: config.region,
      },
    });
  }
}
```

#### 2. Lambda Handler - Read from process.env
```typescript
import { APIGatewayProxyEvent, APIGatewayProxyResult } from "aws-lambda";

/**
 * Lambda handler for POST /users
 * 
 * Environment variables (set by CDK):
 * - USER_POOL_ID: Cognito User Pool ID
 * - USER_POOL_CLIENT_ID: Cognito User Pool Client ID
 * - TABLE_NAME: DynamoDB table name
 */
export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  
  // ✅ CORRECT: Read from process.env at runtime
  const userPoolId = process.env.USER_POOL_ID;
  const userPoolClientId = process.env.USER_POOL_CLIENT_ID;
  const tableName = process.env.TABLE_NAME;
  
  // ❌ WRONG: Can't import config object (doesn't exist in Lambda bundle)
  // import config from '#config/default';
  // const tableName = config.resources.tablePrefix;
  
  // Validate required env vars
  if (!userPoolId || !userPoolClientId || !tableName) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: "Missing required environment variables" }),
    };
  }
  
  // Use env vars in application logic
  const cognito = new CognitoClient({ region: process.env.AWS_REGION });
  // ... rest of handler
};
```

### Why Lambda Reads process.env Directly?

**Reasons:**
- **Config object doesn't exist** - It's not bundled with Lambda code
- **Runtime values** - Many values (like ARNs) are only known after CDK deployment
- **AWS Lambda standard** - This is the standard AWS Lambda pattern
- **Performance** - No overhead of loading/parsing config files

---

## Testing Contexts

### 1. Unit Tests (Testing Lambda Code)

**Context:** Simulating Lambda runtime environment

**Pattern:** Manually set `process.env` to test environment-dependent behavior

```typescript
describe("getRequiredEnv", () => {
  const originalEnv = process.env;
  
  beforeEach(() => {
    // Reset to clean state
    process.env = { ...originalEnv };
  });
  
  afterEach(() => {
    // Restore original
    process.env = originalEnv;
  });
  
  test("returns ok=true when all env vars are set", () => {
    // ✅ CORRECT: Set env vars to simulate Lambda environment
    process.env.USER_POOL_ID = "test-pool-id";
    process.env.USER_POOL_CLIENT_ID = "test-client-id";
    process.env.TABLE_NAME = "test-table";
    
    const result = getRequiredEnv();
    
    expect(result.ok).toBe(true);
    expect(result.data.userPoolId).toBe("test-pool-id");
  });
  
  test("returns error when USER_POOL_ID is missing", () => {
    // ✅ CORRECT: Omit env var to test error handling
    process.env.USER_POOL_CLIENT_ID = "test-client-id";
    process.env.TABLE_NAME = "test-table";
    delete process.env.USER_POOL_ID;
    
    const result = getRequiredEnv();
    
    expect(result.ok).toBe(false);
    expect(result.response.statusCode).toBe(500);
  });
});
```

### 2. Integration Tests (Testing Lambda Handlers)

**Context:** Testing handler with mocked AWS services

**Pattern:** Set `process.env` and mock AWS SDK

```typescript
// Mock AWS SDK before importing handler
const mockSend = jest.fn();

jest.mock("@aws-sdk/client-cognito-identity-provider", () => ({
  CognitoIdentityProviderClient: jest.fn().mockImplementation(() => ({
    send: mockSend,
  })),
  SignUpCommand: jest.fn((params) => params),
}));

import { handler } from "#lib/api/endpoints/users/post/handler";

describe("POST /users handler", () => {
  beforeEach(() => {
    jest.resetAllMocks();
    // ✅ CORRECT: Set env vars that CDK would set
    process.env.USER_POOL_ID = "test-pool-id";
    process.env.USER_POOL_CLIENT_ID = "test-client-id";
    process.env.TABLE_NAME = "test-table";
  });
  
  test("returns 201 on successful sign-up", async () => {
    mockSend.mockResolvedValueOnce({
      UserSub: "test-user-123",
      UserConfirmed: false,
    });
    
    const event = makeEvent({ /* valid payload */ });
    const res = await handler(event, {} as any);
    
    expect(res.statusCode).toBe(201);
  });
});
```

### 3. Infrastructure Tests (Testing CDK Constructs)

**Context:** Testing CloudFormation template generation

**Pattern:** Pass mock config object

```typescript
import { Template } from "aws-cdk-lib/assertions";
import { ServiceStack } from "#lib/service-stack";
import type { IConfig } from "#config/default";

describe("ServiceStack", () => {
  test("creates Cognito User Pool with correct groups", () => {
    const app = new cdk.App();
    
    // ✅ CORRECT: Create mock config object
    const mockConfig: IConfig = {
      envName: "test",
      accountId: "123456789012",
      region: "us-east-1",
      service: { name: "users-ms", displayName: "Users MS" },
      // ... other required fields
    };
    
    const stack = new ServiceStack(app, "TestStack", {
      env: { account: mockConfig.accountId, region: mockConfig.region },
      config: mockConfig, // Pass mock config
    });
    
    const template = Template.fromStack(stack);
    
    // Assert on generated CloudFormation
    template.hasResourceProperties("AWS::Cognito::UserPoolGroup", {
      GroupName: "merchant",
    });
  });
});
```

### 4. E2E Tests (Testing Deployed Infrastructure)

**Context:** Testing against real AWS resources

**Pattern:** Read from `outputs.json` (CDK deployment outputs)

```typescript
import request from "supertest";
import { getApiBaseUrlFromOutputs } from "#test/support/get-api-url";

// ✅ CORRECT: Read API URL from CDK outputs
const resolvedApiUrl = getApiBaseUrlFromOutputs() || process.env.TEST_API_URL;

const preconditionsOk = Boolean(resolvedApiUrl);
const maybeDescribe = preconditionsOk ? describe : describe.skip;

maybeDescribe("E2E: POST /users", () => {
  const client = request(resolvedApiUrl!);
  
  test("returns 201 with userId on valid payload", async () => {
    const body = {
      userType: "merchant",
      email: `e2e-test-${Date.now()}@example.com`,
      password: "Test123!@#",
      // ... rest of payload
    };
    
    const res = await client.post("/users").send(body);
    
    expect([200, 201]).toContain(res.status);
    expect(res.body).toHaveProperty("userId");
  });
});
```

**Why read from outputs.json?**
- Contains actual deployed resource values (API URLs, ARNs, etc.)
- Generated by `cdk deploy --outputs-file outputs.json`
- Ensures tests run against real infrastructure

---

## Common Questions & Answers

### Q1: Why can't Lambda handlers import the config object?

**A:** The config object is designed for CDK synthesis time, not Lambda runtime:

1. **Different execution context** - Config runs on your machine during `cdk deploy`, Lambda runs in AWS
2. **Not bundled** - The config files aren't included in the Lambda bundle
3. **Unnecessary overhead** - Lambda gets values directly from CDK via environment variables
4. **Runtime values** - Many values (ARNs, IDs) don't exist until after deployment

**Example of the problem:**
```typescript
// ❌ WRONG: This won't work in Lambda
import config from '#config/default';

export const handler = async () => {
  const tableName = config.resources.tablePrefix; // Config doesn't exist!
};

// ✅ CORRECT: Read from process.env
export const handler = async () => {
  const tableName = process.env.TABLE_NAME; // Set by CDK
};
```

### Q2: Is the config object redundant since we have dotenv?

**A:** No! They serve different purposes:

| Tool | Purpose | When Used | Where Used |
|------|---------|-----------|------------|
| **dotenv** | Loads `.env` into `process.env` | CDK synthesis | Local development |
| **config object** | Validates, types, and organizes env vars | CDK synthesis | All environments |
| **process.env in Lambda** | Runtime configuration | Lambda execution | AWS Lambda |

**Flow:**
```
.env file → dotenv.config() → process.env → config object → CDK constructs
                                                ↓
                                          Lambda env vars → Lambda process.env
```

### Q3: Should CDK constructs ever read process.env directly?

**A:** No! Always use the config object:

```typescript
// ❌ WRONG: CDK construct reading process.env
class MyConstruct extends Construct {
  constructor(scope: Construct, id: string) {
    super(scope, id);
    const region = process.env.AWS_REGION; // Bad!
  }
}

// ✅ CORRECT: CDK construct using config object
interface IMyConstructProps {
  readonly config: IConfig;
}

class MyConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IMyConstructProps) {
    super(scope, id);
    const region = props.config.region; // Good!
  }
}
```

**Why?**
- Single source of truth
- Type safety
- Validation
- Testability

### Q4: How do I test Lambda handlers that read process.env?

**A:** Set `process.env` in your tests to simulate the Lambda environment:

```typescript
describe("handler", () => {
  const originalEnv = process.env;
  
  beforeEach(() => {
    process.env = { ...originalEnv };
    // Set env vars that CDK would set
    process.env.TABLE_NAME = "test-table";
  });
  
  afterEach(() => {
    process.env = originalEnv; // Restore
  });
  
  test("uses TABLE_NAME from env", async () => {
    const res = await handler(makeEvent());
    // Assert handler used the env var correctly
  });
});
```

### Q5: What if I need the same value in both CDK and Lambda?

**A:** CDK reads from config, then passes to Lambda via environment variables:

```typescript
// CDK construct
class ApiConstruct extends Construct {
  constructor(scope: Construct, id: string, props: { config: IConfig }) {
    super(scope, id);
    
    const { config } = props;
    
    // CDK reads from config
    const region = config.region;
    
    // CDK passes to Lambda
    new NodejsFunction(this, 'Handler', {
      environment: {
        REGION: region, // Lambda will read from process.env.REGION
      },
    });
  }
}

// Lambda handler
export const handler = async () => {
  const region = process.env.REGION; // Reads value CDK set
};
```

### Q6: Why do tests manipulate process.env? Isn't that bad practice?

**A:** It's necessary and correct for testing environment-dependent behavior:

**Unit tests** must simulate the Lambda environment:
```typescript
test("handles missing env var", () => {
  delete process.env.TABLE_NAME; // Simulate missing env var
  const result = getRequiredEnv();
  expect(result.ok).toBe(false); // Verify error handling
});
```

**Best practices:**
- Save and restore original `process.env`
- Use `beforeEach`/`afterEach` for cleanup
- Isolate tests so they don't affect each other

### Q7: When should I use outputs.json vs process.env in tests?

**A:** Depends on the test type:

| Test Type | Use | Why |
|-----------|-----|-----|
| **Unit tests** | `process.env` | Simulating Lambda environment |
| **Integration tests** | `process.env` | Testing handler with mocked AWS |
| **Infrastructure tests** | Mock config object | Testing CDK template generation |
| **E2E tests** | `outputs.json` | Testing real deployed resources |

```typescript
// E2E test - use outputs.json
const apiUrl = getApiBaseUrlFromOutputs(); // ✅ Real deployed API

// Unit test - use process.env
process.env.TABLE_NAME = "test-table"; // ✅ Simulate Lambda env
```

### Q8: What's the difference between .env and Lambda environment variables?

**A:** Completely different contexts:

| `.env` File | Lambda Environment Variables |
|-------------|------------------------------|
| Used during `cdk deploy` | Used during Lambda execution |
| On your local machine | In AWS Lambda environment |
| Configures infrastructure | Configures application |
| Read by config object | Read by handler code |
| Example: `AWS_ACCOUNT_ID` | Example: `TABLE_NAME` |

**They never interact directly!** CDK reads `.env` → creates infrastructure → sets Lambda env vars.

---

## Best Practices

### 1. CDK Synthesis Time

✅ **DO:**
- Load `.env` with `dotenv.config()` in `bin/app.ts`
- Read `process.env` only in `config/default.ts`
- Pass config object to all constructs
- Validate required values in config
- Use TypeScript types for config

❌ **DON'T:**
- Read `process.env` directly in constructs
- Skip validation of required values
- Mix config and process.env access

### 2. Lambda Runtime

✅ **DO:**
- Read `process.env` directly in handlers
- Validate required env vars at handler start
- Use descriptive env var names
- Document required env vars in JSDoc
- Handle missing env vars gracefully

❌ **DON'T:**
- Try to import config object in Lambda
- Assume env vars are always set
- Use cryptic env var names
- Skip validation

### 3. Testing

✅ **DO:**
- Save and restore `process.env` in tests
- Use `beforeEach`/`afterEach` for cleanup
- Test both success and error cases
- Use `outputs.json` for E2E tests
- Mock AWS services in integration tests

❌ **DON'T:**
- Leave `process.env` modified after tests
- Skip testing missing env var scenarios
- Use real AWS resources in unit tests
- Forget to clean up test state

---

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: CDK Construct Reading process.env

```typescript
// ❌ WRONG
class MyConstruct extends Construct {
  constructor(scope: Construct, id: string) {
    super(scope, id);
    const region = process.env.AWS_REGION; // Bad!
  }
}

// ✅ CORRECT
interface IMyConstructProps {
  readonly config: IConfig;
}

class MyConstruct extends Construct {
  constructor(scope: Construct, id: string, props: IMyConstructProps) {
    super(scope, id);
    const region = props.config.region; // Good!
  }
}
```

### ❌ Anti-Pattern 2: Lambda Importing Config

```typescript
// ❌ WRONG
import config from '#config/default';

export const handler = async () => {
  const tableName = config.resources.tablePrefix; // Won't work!
};

// ✅ CORRECT
export const handler = async () => {
  const tableName = process.env.TABLE_NAME; // Works!
};
```

### ❌ Anti-Pattern 3: Hardcoding Values

```typescript
// ❌ WRONG
const handler = new NodejsFunction(this, 'Handler', {
  environment: {
    REGION: 'us-east-1', // Hardcoded!
  },
});

// ✅ CORRECT
const handler = new NodejsFunction(this, 'Handler', {
  environment: {
    REGION: config.region, // From config!
  },
});
```

### ❌ Anti-Pattern 4: Not Validating Env Vars

```typescript
// ❌ WRONG
export const handler = async () => {
  const tableName = process.env.TABLE_NAME; // Might be undefined!
  await dynamodb.putItem({ TableName: tableName }); // Crash!
};

// ✅ CORRECT
export const handler = async () => {
  const tableName = process.env.TABLE_NAME;
  if (!tableName) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: "TABLE_NAME not configured" }),
    };
  }
  await dynamodb.putItem({ TableName: tableName });
};
```

---

## Troubleshooting

### Problem: "Config value is undefined during cdk deploy"

**Cause:** Environment variable not set in `.env` or shell

**Solution:**
1. Check `.env` file exists and has the value
2. Verify `dotenv.config()` is called in `bin/app.ts`
3. Check config validation in `config/default.ts`

```bash
# Verify .env is loaded
cat .env | grep AWS_REGION

# Check CDK can see it
cdk synth --debug
```

### Problem: "Lambda handler can't find environment variable"

**Cause:** CDK construct didn't set the environment variable

**Solution:**
1. Check construct sets the env var:
```typescript
new NodejsFunction(this, 'Handler', {
  environment: {
    TABLE_NAME: table.tableName, // Make sure this is set!
  },
});
```

2. Verify in AWS Console:
   - Lambda → Configuration → Environment variables
   - Should see `TABLE_NAME` with actual value

### Problem: "Tests fail with 'process.env.X is not defined'"

**Cause:** Test didn't set required environment variables

**Solution:**
```typescript
beforeEach(() => {
  // Set all required env vars
  process.env.TABLE_NAME = "test-table";
  process.env.USER_POOL_ID = "test-pool";
});
```

### Problem: "E2E tests can't find API URL"

**Cause:** `outputs.json` not generated or in wrong location

**Solution:**
```bash
# Deploy with outputs file
cdk deploy --outputs-file outputs.json

# Verify file exists
cat outputs.json

# Check test support file
cat test/support/get-api-url.ts
```

---

## Summary

### The Golden Rules

1. **CDK Synthesis:** `.env` → `config object` → constructs
2. **Lambda Runtime:** CDK sets env vars → Lambda reads `process.env`
3. **Tests:** Simulate appropriate context (unit: `process.env`, E2E: `outputs.json`)
4. **Never:** Import config in Lambda or read `process.env` in constructs

### Quick Reference

| Context | Read From | Set By | Example |
|---------|-----------|--------|---------|
| CDK Construct | `props.config.region` | `.env` + `config/default.ts` | Infrastructure config |
| Lambda Handler | `process.env.TABLE_NAME` | CDK construct | Runtime config |
| Unit Test | `process.env.TABLE_NAME = "test"` | Test setup | Simulate Lambda |
| E2E Test | `getApiBaseUrlFromOutputs()` | `cdk deploy --outputs-file` | Real resources |

---

## Additional Resources

- [AWS Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
- [AWS CDK Best Practices](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html)
- [dotenv Documentation](https://github.com/motdotla/dotenv)
- [Jest Environment Variables](https://jestjs.io/docs/environment-variables)

---

**Last Updated:** October 2025  
**Maintainer:** Development Team  
**Related Docs:** 
- [Testing Strategy](../testing/mutation-testing.md)
- [CDK Patterns](../patterns/cdk-constructs.md)
- [Lambda Best Practices](../patterns/lambda-handlers.md)
