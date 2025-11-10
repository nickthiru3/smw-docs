# OpenAPI Modularization Guide

**Purpose**: Story-driven, modular API specification using OpenAPI 3.0.3  
**Last Updated**: 2024-11-07

---

## Table of Contents

1. [Why Modularization?](#why-modularization)
2. [Architecture Overview](#architecture-overview)
3. [Directory Structure](#directory-structure)
4. [Modularization Patterns](#modularization-patterns)
5. [Story Workflow](#story-workflow)
6. [Best Practices](#best-practices)
7. [Validation & Tooling](#validation--tooling)
8. [AWS Integration](#aws-integration)

---

## Why Modularization?

### The Problem with Monolithic OpenAPI Specs

**Traditional Approach** (Single File):
```
api/
└── openapi.yaml  (5000+ lines, all endpoints, schemas, responses)
```

**Problems:**
- ❌ **Merge Conflicts** - Multiple teams editing same file
- ❌ **Difficult Navigation** - Finding specific endpoints in huge file
- ❌ **No Ownership** - Unclear which team owns which endpoints
- ❌ **Slow Reviews** - Large diffs, hard to review changes
- ❌ **Tight Coupling** - Changes to one endpoint affect entire file

### The Modular Approach

**Our Approach** (Modular Structure):
```
api/
├── openapi.yaml           # Root (100 lines)
├── schemas/               # Reusable data models
├── responses/             # Reusable responses
├── parameters/            # Reusable parameters
└── resources/             # Endpoints by resource
    └── merchants/
        ├── search.yaml    # Story 001
        └── get-by-id.yaml # Story 002
```

**Benefits:**
- ✅ **Story-Level Ownership** - Each story creates its own files
- ✅ **Parallel Development** - Teams work without conflicts
- ✅ **Easy Navigation** - Find endpoints by resource/operation
- ✅ **Clear Ownership** - File-level ownership
- ✅ **Small Diffs** - Easy code reviews
- ✅ **Loose Coupling** - Changes isolated to specific files

---

## Architecture Overview

### Modularization Strategy

```
┌─────────────────────────────────────────────────────────┐
│ Root: openapi.yaml                                      │
│ - API metadata (title, version, servers)                │
│ - Component registries ($ref to _index.yaml files)      │
│ - Path registries ($ref to resource files)              │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┬─────────────┐
        │                 │                 │             │
        ▼                 ▼                 ▼             ▼
┌───────────────┐ ┌───────────────┐ ┌──────────────┐ ┌──────────────┐
│ schemas/      │ │ responses/    │ │ parameters/  │ │ resources/   │
│ _index.yaml   │ │ _index.yaml   │ │ _index.yaml  │ │ (endpoints)  │
│               │ │               │ │              │ │              │
│ Merchant.yaml │ │ ValidationErr │ │ category.yaml│ │ search.yaml  │
│ Error.yaml    │ │ ServerError   │ │ merchantId   │ │ create.yaml  │
│ Review.yaml   │ │ NotFound      │ │ ...          │ │ ...          │
└───────────────┘ └───────────────┘ └──────────────┘ └──────────────┘
```

### Reference Chain

```yaml
# Root: openapi.yaml
paths:
  /merchants/search:
    $ref: "./resources/merchants/search.yaml"

# Resource: resources/merchants/search.yaml
get:
  parameters:
    - $ref: "../../parameters/query/category.yaml"
  responses:
    "200":
      content:
        application/json:
          schema:
            $ref: "../../schemas/Merchant.yaml"
    "400":
      $ref: "../../responses/ValidationError.yaml"
```

---

## Directory Structure

### Complete Structure

```
docs/project/specs/api/
├── openapi.yaml                          # Root specification
├── README.md                             # API documentation
│
├── schemas/                              # Data models (entities)
│   ├── _index.yaml                       # Schema registry
│   ├── Merchant.yaml                     # Merchant entity
│   ├── Review.yaml                       # Review entity
│   ├── Error.yaml                        # Standard error
│   └── ...
│
├── responses/                            # Reusable responses
│   ├── _index.yaml                       # Response registry
│   ├── ValidationError.yaml              # 400 Bad Request
│   ├── InternalServerError.yaml          # 500 Server Error
│   ├── NotFound.yaml                     # 404 Not Found
│   ├── Unauthorized.yaml                 # 401 Unauthorized
│   └── ...
│
├── parameters/                           # Reusable parameters
│   ├── _index.yaml                       # Parameter registry
│   ├── path/                             # Path parameters
│   │   ├── merchantId.yaml
│   │   └── ...
│   ├── query/                            # Query parameters
│   │   ├── category.yaml
│   │   ├── limit.yaml
│   │   ├── offset.yaml
│   │   └── ...
│   ├── header/                           # Header parameters
│   │   ├── cache-control.yaml
│   │   ├── authorization.yaml
│   │   └── ...
│   └── cookie/                           # Cookie parameters
│       └── ...
│
└── resources/                            # API endpoints
    ├── merchants/                        # Merchant resource
    │   ├── search.yaml                   # GET /merchants/search
    │   ├── get-by-id.yaml                # GET /merchants/{id}
    │   ├── create.yaml                   # POST /merchants
    │   └── ...
    ├── reviews/                          # Review resource
    │   ├── list-by-merchant.yaml         # GET /reviews?merchantId=X
    │   ├── create.yaml                   # POST /reviews
    │   └── ...
    └── ...
```

### File Naming Conventions

**All files use `kebab-case.yaml`** for consistency:

**Schemas**: 
- `merchant.yaml`, `review.yaml`, `error.yaml`
- Singular form for entities

**Responses**: 
- `validation-error.yaml`, `not-found.yaml`, `unauthorized.yaml`
- Descriptive of error type

**Parameters**: 
- `merchant-id.yaml`, `category.yaml`, `cache-control.yaml`
- Match parameter name

**Resources**: 
- `search.yaml`, `get-by-id.yaml`, `create.yaml`
- Match HTTP method + action

---

## Modularization Patterns

### Pattern 1: Schema Modularization

**Problem**: Large, complex schemas duplicated across endpoints

**Solution**: Extract to reusable schema files

```yaml
# schemas/merchant.yaml
type: object
description: Merchant entity
required:
  - merchantId
  - legalName
  - primaryCategory
properties:
  merchantId:
    type: string
    format: uuid
  legalName:
    type: string
    minLength: 1
    maxLength: 200
  # ... more properties

# schemas/_index.yaml
Merchant:
  $ref: "./merchant.yaml"
Error:
  $ref: "./error.yaml"

# Root: openapi.yaml
components:
  schemas:
    $ref: "./schemas/_index.yaml"

# Usage in endpoint
responses:
  "200":
    content:
      application/json:
        schema:
          $ref: "../../schemas/merchant.yaml"
```

### Pattern 2: Response Modularization

**Problem**: Same error responses repeated across endpoints

**Solution**: Extract to reusable response files

```yaml
# responses/validation-error.yaml
description: Invalid request parameters or body
content:
  application/json:
    schema:
      $ref: "../schemas/error.yaml"
    examples:
      invalidCategory:
        summary: Invalid category parameter
        value:
          error: "Invalid category"
          message: "Category must be one of: Repair, Refill, Recycling, Donate"
          code: "INVALID_CATEGORY"

# responses/_index.yaml
ValidationError:
  $ref: "./validation-error.yaml"
InternalServerError:
  $ref: "./internal-server-error.yaml"

# Root: openapi.yaml
components:
  responses:
    $ref: "./responses/_index.yaml"

# Usage in endpoint
responses:
  "400":
    $ref: "../../responses/validation-error.yaml"
  "500":
    $ref: "../../responses/internal-server-error.yaml"
```

### Pattern 3: Parameter Modularization

**Problem**: Same parameters used across multiple endpoints

**Solution**: Extract to reusable parameter files organized by type

```yaml
# parameters/query/category.yaml
name: category
in: query
required: true
description: Primary service category to filter by
schema:
  type: string
  enum: [Repair, Refill, Recycling, Donate]
example: Repair

# parameters/path/merchant-id.yaml
name: merchantId
in: path
required: true
description: Unique identifier for the merchant
schema:
  type: string
  format: uuid
example: "550e8400-e29b-41d4-a716-446655440000"

# parameters/_index.yaml
# Query parameters
category:
  $ref: "./query/category.yaml"
limit:
  $ref: "./query/limit.yaml"

# Path parameters
merchantId:
  $ref: "./path/merchant-id.yaml"

# Root: openapi.yaml
components:
  parameters:
    $ref: "./parameters/_index.yaml"

# Usage in endpoint
parameters:
  - $ref: "../../parameters/query/category.yaml"
  - $ref: "../../parameters/path/merchant-id.yaml"
```

### Pattern 4: Resource/Endpoint Modularization

**Problem**: Hundreds of endpoints in single file

**Solution**: One file per endpoint, organized by resource

```yaml
# resources/merchants/search.yaml
get:
  operationId: searchMerchantsByCategory
  tags:
    - merchants
  summary: Search merchants by category
  parameters:
    - $ref: "../../parameters/query/category.yaml"
  responses:
    "200":
      description: Successful response
      content:
        application/json:
          schema:
            type: object
            properties:
              merchants:
                type: array
                items:
                  $ref: "../../schemas/merchant.yaml"
    "400":
      $ref: "../../responses/validation-error.yaml"

# Root: openapi.yaml
paths:
  /merchants/search:
    $ref: "./resources/merchants/search.yaml"
  /merchants/{merchantId}:
    $ref: "./resources/merchants/get-by-id.yaml"
```

---

## Story Workflow

### Step-by-Step: Adding a New Endpoint

#### Step 1: Identify Requirements

From your story card and actions-queries.md:
- Endpoint path: `/merchants/{merchantId}`
- HTTP method: `GET`
- Request: Path parameter `merchantId`
- Response: `Merchant` object
- Errors: `404 Not Found`, `500 Internal Server Error`

#### Step 2: Check for Reusable Components

**Schemas**: Does `Merchant.yaml` exist?
- ✅ Yes → Reuse it
- ❌ No → Create it

**Parameters**: Does `merchantId.yaml` exist?
- ✅ Yes → Reuse it
- ❌ No → Create it

**Responses**: Do `NotFound.yaml` and `InternalServerError.yaml` exist?
- ✅ Yes → Reuse them
- ❌ No → Create them

#### Step 3: Create New Components (if needed)

```bash
# Create parameter (if doesn't exist)
touch parameters/path/merchantId.yaml

# Create response (if doesn't exist)
touch responses/NotFound.yaml

# Update registries
# Edit parameters/_index.yaml
# Edit responses/_index.yaml
```

#### Step 4: Create Endpoint File

```bash
# Create resource file
touch resources/merchants/get-by-id.yaml
```

```yaml
# resources/merchants/get-by-id.yaml
get:
  operationId: getMerchantById
  tags:
    - merchants
  summary: Get merchant details by ID
  description: |
    Retrieves complete merchant information including location, contact, ratings, and services.
    
    **Story**: 002 - View Detailed Business Information
  
  parameters:
    - $ref: "../../parameters/path/merchantId.yaml"
  
  responses:
    "200":
      description: Merchant details
      content:
        application/json:
          schema:
            $ref: "../../schemas/Merchant.yaml"
    
    "404":
      $ref: "../../responses/NotFound.yaml"
    
    "500":
      $ref: "../../responses/InternalServerError.yaml"
  
  x-cache-strategy:
    bff:
      enabled: true
      ttl: 300
      key: "merchant:${merchantId}"
  
  x-performance-targets:
    backend: 50
    bff: 100
    total: 150
```

#### Step 5: Register in Root OpenAPI

```yaml
# openapi.yaml
paths:
  /merchants/search:
    $ref: "./resources/merchants/search.yaml"
  
  # Add new endpoint
  /merchants/{merchantId}:
    $ref: "./resources/merchants/get-by-id.yaml"
```

#### Step 6: Validate

```bash
# Validate OpenAPI spec
npx @apidevtools/swagger-cli validate openapi.yaml

# Or use Redocly
npx @redocly/cli lint openapi.yaml
```

#### Step 7: Generate Documentation

```bash
# Generate HTML docs
npx @redocly/cli build-docs openapi.yaml -o api-docs.html

# Or run mock server for testing
npx @stoplight/prism-cli mock openapi.yaml
```

---

## Best Practices

### 1. Registry Pattern

**Always use `_index.yaml` files as registries:**

```yaml
# schemas/_index.yaml
Merchant:
  $ref: "./Merchant.yaml"
Review:
  $ref: "./Review.yaml"
Error:
  $ref: "./Error.yaml"
```

**Why?**
- ✅ Single source of truth for component names
- ✅ Easy to see all available components
- ✅ Prevents naming conflicts
- ✅ Simplifies refactoring

### 2. Relative References

**Use relative paths from endpoint files:**

```yaml
# resources/merchants/search.yaml
parameters:
  - $ref: "../../parameters/query/category.yaml"  # ✅ Relative
  # NOT: $ref: "#/components/parameters/category"  # ❌ Absolute
```

**Why?**
- ✅ Files are self-contained
- ✅ Easy to move/reorganize
- ✅ Works with validation tools

### 3. Consistent Naming

**Follow naming conventions (all kebab-case):**

| Type        | Convention  | Example                    |
| ----------- | ----------- | -------------------------- |
| Schemas     | kebab-case  | `merchant.yaml`            |
| Responses   | kebab-case  | `validation-error.yaml`    |
| Parameters  | kebab-case  | `merchant-id.yaml`         |
| Resources   | kebab-case  | `get-by-id.yaml`           |
| Directories | kebab-case  | `merchants/`               |

### 4. Story Metadata

**Document which story created each endpoint:**

```yaml
# resources/merchants/search.yaml
get:
  summary: Search merchants by category
  description: |
    Returns all merchants in the specified category.
    
    **Story**: 001 - Browse Providers by Waste Category
    **Created**: 2024-11-06
```

### 5. Custom Extensions

**Use `x-` extensions for implementation details:**

```yaml
x-cache-strategy:
  bff:
    enabled: true
    ttl: 300
    key: "merchants:search:${category}"

x-performance-targets:
  backend: 100
  bff: 150
  total: 200

x-database-query:
  table: Merchants
  operation: Query
  index: GSI1
  keyCondition: "GSI1PK = :category"
```

### 6. Examples in Responses

**Provide realistic examples:**

```yaml
responses:
  "200":
    content:
      application/json:
        schema:
          $ref: "../../schemas/merchant.yaml"
        examples:
          repairShop:
            summary: A repair shop merchant
            value:
              merchantId: "550e8400-e29b-41d4-a716-446655440000"
              legalName: "Green Repair Shop Inc."
              # ... complete example
```

### 7. Error Response Consistency

**Use standard error schema across all endpoints:**

```yaml
# schemas/error.yaml
type: object
required:
  - error
  - message
  - code
properties:
  error:
    type: string
    description: Error type
  message:
    type: string
    description: Human-readable message
  code:
    type: string
    description: Machine-readable code
```

---

## Validation & Tooling

### Validation Tools

#### 1. Swagger CLI

```bash
# Install
npm install -g @apidevtools/swagger-cli

# Validate
swagger-cli validate openapi.yaml

# Bundle (combine all refs into single file)
swagger-cli bundle openapi.yaml -o openapi-bundled.yaml
```

#### 2. Redocly CLI

```bash
# Install
npm install -g @redocly/cli

# Lint
redocly lint openapi.yaml

# Bundle
redocly bundle openapi.yaml -o openapi-bundled.yaml

# Generate docs
redocly build-docs openapi.yaml -o api-docs.html
```

#### 3. Spectral (Advanced Linting)

```bash
# Install
npm install -g @stoplight/spectral-cli

# Create .spectral.yaml
extends: [[spectral:oas, all]]
rules:
  operation-description: error
  operation-tags: error

# Lint
spectral lint openapi.yaml
```

### Documentation Generation

#### 1. Redoc (Static HTML)

```bash
npx @redocly/cli build-docs openapi.yaml -o api-docs.html
```

#### 2. Swagger UI (Interactive)

```bash
npx swagger-ui-watcher openapi.yaml
# Opens at http://localhost:3000
```

#### 3. RapiDoc (Modern UI)

```html
<!DOCTYPE html>
<html>
  <head>
    <script
      type="module"
      src="https://unpkg.com/rapidoc/dist/rapidoc-min.js"
    ></script>
  </head>
  <body>
    <rapi-doc spec-url="openapi.yaml" theme="dark"> </rapi-doc>
  </body>
</html>
```

### Mock Server

#### Prism (Stoplight)

```bash
# Install
npm install -g @stoplight/prism-cli

# Run mock server
prism mock openapi.yaml

# Server runs at http://localhost:4010

# Test endpoint
curl http://localhost:4010/merchants/search?category=Repair
```

### Code Generation

#### OpenAPI Generator

```bash
# Install
npm install -g @openapitools/openapi-generator-cli

# Generate TypeScript client
openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-fetch \
  -o ./generated/client

# Generate server stubs
openapi-generator-cli generate \
  -i openapi.yaml \
  -g nodejs-express-server \
  -o ./generated/server
```

---

## AWS Integration

### AWS API Gateway Extensions

#### Request Validation

```yaml
# Root: openapi.yaml
x-amazon-apigateway-request-validators:
  all:
    validateRequestBody: true
    validateRequestParameters: true
  params:
    validateRequestParameters: true
    validateRequestBody: false
  body:
    validateRequestBody: true
    validateRequestParameters: false

x-amazon-apigateway-request-validator: all  # Default validator

# Per-endpoint override
# resources/merchants/search.yaml
get:
  x-amazon-apigateway-request-validator: params
```

**Benefits:**
- ✅ Validation at API Gateway (before Lambda invocation)
- ✅ Saves Lambda costs for invalid requests
- ✅ Faster error responses

#### Lambda Integration

```yaml
# resources/merchants/search.yaml
get:
  x-amazon-apigateway-integration:
    type: aws_proxy
    httpMethod: POST
    uri: arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${SearchMerchantsFunction}/invocations
    passthroughBehavior: when_no_match
```

#### CORS Configuration

```yaml
# Root: openapi.yaml
x-amazon-apigateway-cors:
  allowOrigin: "*"  # Or specific domain: "https://smw.example.com"
  allowHeaders:
    - Content-Type
    - Authorization
    - X-Api-Key
  allowMethods:
    - GET
    - POST
    - PUT
    - DELETE
    - OPTIONS
  maxAge: 3600
```

#### Binary Media Types

```yaml
# Root: openapi.yaml
x-amazon-apigateway-binary-media-types:
  - image/jpeg
  - image/png
  - application/pdf
```

### CDK Integration

#### Generate API Gateway from OpenAPI

```typescript
// lib/constructs/api-from-openapi.ts
import * as cdk from "aws-cdk-lib";
import * as apigateway from "aws-cdk-lib/aws-apigateway";
import * as lambda from "aws-cdk-lib/aws-lambda";
import { Construct } from "constructs";
import * as fs from "fs";
import * as yaml from "js-yaml";

export interface ApiFromOpenApiProps {
  openApiPath: string;
  lambdaFunctions: Record<string, lambda.Function>;
}

export class ApiFromOpenApi extends Construct {
  public readonly api: apigateway.SpecRestApi;

  constructor(scope: Construct, id: string, props: ApiFromOpenApiProps) {
    super(scope, id);

    // Load OpenAPI spec
    const openApiSpec = yaml.load(
      fs.readFileSync(props.openApiPath, "utf8")
    ) as any;

    // Replace Lambda ARN placeholders
    const specString = JSON.stringify(openApiSpec);
    let processedSpec = specString;

    for (const [name, func] of Object.entries(props.lambdaFunctions)) {
      processedSpec = processedSpec.replace(
        new RegExp(`\\$\\{${name}\\}`, "g"),
        func.functionArn
      );
    }

    // Create API Gateway from OpenAPI spec
    this.api = new apigateway.SpecRestApi(this, "Api", {
      apiDefinition: apigateway.ApiDefinition.fromInline(
        JSON.parse(processedSpec)
      ),
      deployOptions: {
        stageName: "prod",
        tracingEnabled: true,
        metricsEnabled: true,
      },
    });

    // Grant API Gateway permission to invoke Lambdas
    for (const func of Object.values(props.lambdaFunctions)) {
      func.grantInvoke(
        new cdk.aws_iam.ServicePrincipal("apigateway.amazonaws.com")
      );
    }
  }
}

// Usage in stack
const searchFunction = new lambda.Function(this, "SearchFunction", {
  // ... Lambda config
});

const api = new ApiFromOpenApi(this, "Api", {
  openApiPath: "docs/project/specs/api/openapi.yaml",
  lambdaFunctions: {
    SearchMerchantsFunction: searchFunction,
  },
});
```

---

## Troubleshooting

### Common Issues

#### 1. Reference Resolution Errors

**Error**: `Can't resolve reference: ../../schemas/Merchant.yaml`

**Solution**: Check relative path from endpoint file to schema file

```yaml
# From: resources/merchants/search.yaml
# To: schemas/Merchant.yaml
# Path: ../../schemas/Merchant.yaml

# Count directory levels:
# resources/merchants/ → resources/ → root/ → schemas/
# = 2 levels up (../..) + schemas/
```

#### 2. Circular References

**Error**: `Circular reference detected`

**Solution**: Avoid schemas referencing each other in loops

```yaml
# ❌ Bad: Circular reference
# Merchant.yaml
properties:
  reviews:
    type: array
    items:
      $ref: "./Review.yaml"

# Review.yaml
properties:
  merchant:
    $ref: "./Merchant.yaml"

# ✅ Good: Use ID reference
# Review.yaml
properties:
  merchantId:
    type: string
    format: uuid
```

#### 3. Registry Not Found

**Error**: `Can't resolve $ref: "./schemas/_index.yaml"`

**Solution**: Ensure registry file exists and is properly formatted

```yaml
# schemas/_index.yaml must exist with proper structure
Merchant:
  $ref: "./Merchant.yaml"
Error:
  $ref: "./Error.yaml"
```

---

## Migration Strategy

### From Monolithic to Modular

#### Phase 1: Audit Current Spec

```bash
# Count endpoints
grep -c "paths:" openapi-old.yaml

# Count schemas
grep -c "schemas:" openapi-old.yaml

# Identify duplicated responses
grep -A 5 "responses:" openapi-old.yaml | sort | uniq -d
```

#### Phase 2: Extract Schemas

```bash
# Create schemas directory
mkdir -p schemas

# Extract each schema to separate file
# (Manual or scripted)

# Create registry
cat > schemas/_index.yaml << EOF
Merchant:
  \$ref: "./Merchant.yaml"
Error:
  \$ref: "./Error.yaml"
EOF
```

#### Phase 3: Extract Responses

```bash
# Create responses directory
mkdir -p responses

# Extract common responses
# ValidationError, InternalServerError, NotFound, etc.

# Create registry
cat > responses/_index.yaml << EOF
ValidationError:
  \$ref: "./ValidationError.yaml"
InternalServerError:
  \$ref: "./InternalServerError.yaml"
EOF
```

#### Phase 4: Extract Parameters

```bash
# Create parameters directory structure
mkdir -p parameters/{path,query,header,cookie}

# Extract reusable parameters

# Create registry
cat > parameters/_index.yaml << EOF
category:
  \$ref: "./query/category.yaml"
merchantId:
  \$ref: "./path/merchantId.yaml"
EOF
```

#### Phase 5: Extract Endpoints

```bash
# Create resources directory
mkdir -p resources/merchants

# Extract each endpoint to separate file
# One file per path + method combination

# Update root openapi.yaml with $ref to each endpoint
```

#### Phase 6: Validate

```bash
# Validate modular spec
swagger-cli validate openapi.yaml

# Compare with original (should be functionally identical)
swagger-cli bundle openapi.yaml -o openapi-bundled.yaml
diff openapi-old.yaml openapi-bundled.yaml
```

---

## Summary

### Key Takeaways

1. **Modularization enables story-driven development**
   - Each story creates its own endpoint files
   - Teams work in parallel without conflicts

2. **Reusability reduces duplication**
   - Shared schemas, responses, parameters via registries
   - DRY principle applied to API specs

3. **Tooling makes it practical**
   - Validation tools ensure correctness
   - Documentation generators create beautiful docs
   - Mock servers enable frontend development

4. **AWS integration is seamless**
   - `x-amazon-apigateway-*` extensions for AWS features
   - CDK can generate API Gateway from OpenAPI spec

5. **Maintenance is easier**
   - Small files, clear ownership
   - Easy to find and update endpoints
   - Simple to add/remove features

### Next Steps

1. ✅ Create modular structure for your API
2. ✅ Set up validation in CI/CD pipeline
3. ✅ Generate documentation automatically
4. ✅ Use mock server for frontend development
5. ✅ Integrate with AWS CDK for deployment

---

## References

- [OpenAPI 3.0.3 Specification](https://spec.openapis.org/oas/v3.0.3)
- [AWS API Gateway OpenAPI Extensions](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-swagger-extensions.html)
- [Redocly CLI Documentation](https://redocly.com/docs/cli/)
- [Swagger CLI Documentation](https://github.com/APIDevTools/swagger-cli)
- [Prism Mock Server](https://stoplight.io/open-source/prism)
