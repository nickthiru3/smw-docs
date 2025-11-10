# SMW API Specification

**Format**: OpenAPI 3.0.3  
**Structure**: Modular (story-level ownership)  
**Last Updated**: 2024-11-06

---

## Overview

This directory contains the complete API specification for the SMW platform using a **modular, story-driven structure**. Each story adds its own endpoints, schemas, and responses, which are referenced in the root `openapi.yaml` file.

---

## Directory Structure

```
api/
├── openapi.yaml                          # Root OpenAPI spec (references all modules)
├── README.md                            # This file
├── schemas/                             # Reusable data models
│   ├── _index.yaml                      # Schema registry
│   ├── merchant.yaml                    # Merchant entity schema
│   ├── error.yaml                       # Standard error schema
│   └── ...                              # Additional schemas
├── responses/                           # Reusable response definitions
│   ├── _index.yaml                      # Response registry
│   ├── validation-error.yaml            # 400 error response
│   ├── internal-server-error.yaml       # 500 error response
│   └── ...                              # Additional responses
├── parameters/                          # Reusable parameter definitions (future)
│   ├── _index.yaml
│   └── ...
└── resources/                           # API endpoints organized by resource
    └── merchants/
        ├── search.yaml                   # GET /merchants/search (Story 001)
        └── ...                          # Additional merchant endpoints
```

---

## Modular Structure Benefits

### ✅ Story-Level Ownership

- Each story creates its own endpoint files
- Teams work in parallel without conflicts
- Clear ownership and responsibility

### ✅ Reusability

- Shared schemas via `_index.yaml` registries
- Common error responses
- Consistent parameter definitions

### ✅ Version Control Friendly

- Small, focused files
- Easier code reviews
- Clear diffs showing what changed

### ✅ Maintainability

- Find and update specific endpoints easily
- Add new endpoints without touching existing ones
- Remove deprecated endpoints cleanly

---

## How to Add a New Endpoint (Story Workflow)

### Step 1: Create Resource File

Create a new file under `resources/[resource-name]/[operation].yaml`:

```yaml
# resources/merchants/get-by-id.yaml
get:
  operationId: getMerchantById
  tags:
    - merchants
  summary: Get merchant by ID
  parameters:
    - name: merchantId
      in: path
      required: true
      schema:
        type: string
        format: uuid
  responses:
    "200":
      description: Merchant details
      content:
        application/json:
          schema:
            $ref: "../../schemas/Merchant.yaml"
    "404":
      $ref: "../../responses/NotFound.yaml"
```

### Step 2: Add Schemas (if needed)

If your endpoint introduces new data models:

1. Create `schemas/[EntityName].yaml`
2. Add reference to `schemas/_index.yaml`:
   ```yaml
   EntityName:
     $ref: "./EntityName.yaml"
   ```

### Step 3: Add Responses (if needed)

If your endpoint has unique error responses:

1. Create `responses/[ResponseName].yaml`
2. Add reference to `responses/_index.yaml`

### Step 4: Register in Root OpenAPI

Add your endpoint to `openapi.yaml`:

```yaml
paths:
  /merchants/{merchantId}:
    $ref: "./resources/merchants/get-by-id.yaml"
```

---

## Current API Coverage

### Story 001: Browse Providers by Waste Category

**Endpoints**:

- `GET /merchants/search` - Search merchants by category

**Schemas**:

- `Merchant` - Complete merchant entity
- `Error` - Standard error response

**Responses**:

- `ValidationError` - 400 Bad Request
- `InternalServerError` - 500 Internal Server Error

---

## Validation & Testing

### Validate OpenAPI Spec

```bash
# Using Swagger CLI
npx @apidevtools/swagger-cli validate openapi.yaml

# Using Redocly CLI
npx @redocly/cli lint openapi.yaml
```

### Generate Documentation

```bash
# Generate HTML docs with Redoc
npx @redocly/cli build-docs openapi.yaml -o api-docs.html

# Or use Swagger UI
npx swagger-ui-watcher openapi.yaml
```

### Mock Server (for development)

```bash
# Using Prism
npx @stoplight/prism-cli mock openapi.yaml

# Server will run at http://localhost:4010
```

---

## Conventions

### File Naming

- **All files use kebab-case**: `get-by-id.yaml`, `create-merchant.yaml`, `merchant.yaml`, `validation-error.yaml`
- Match HTTP method + action for resources: `search.yaml` (GET), `create.yaml` (POST)
- Singular form for schemas: `merchant.yaml`, `review.yaml`

### Operation IDs

- Use camelCase: `searchMerchantsByCategory`, `getMerchantById`
- Format: `[verb][Resource][ByCondition]`

### Schema Names

- Use PascalCase: `Merchant`, `Review`, `Error`
- Singular form for entities

### Response Names (in \_index.yaml)

- Use PascalCase for keys: `ValidationError`, `NotFound`
- File names use kebab-case: `validation-error.yaml`, `not-found.yaml`
- Descriptive of the error type

### Tags

- Use lowercase with underscores: `merchants`, `reviews`
- Match resource names

---

## Custom Extensions

We use custom `x-` extensions to document additional metadata:

### `x-cache-strategy`

Documents caching configuration:

```yaml
x-cache-strategy:
  bff:
    enabled: true
    ttl: 300
    key: "merchants:search:${category}"
  cdn:
    enabled: true
    ttl: 300
```

### `x-performance-targets`

Documents performance expectations (in milliseconds):

```yaml
x-performance-targets:
  backend: 100
  bff: 150
  total: 200
```

### `x-database-query`

Documents the underlying database query:

```yaml
x-database-query:
  table: Merchants
  operation: Query
  index: GSI1
  keyCondition: "GSI1PK = :category"
```

---

## Related Documentation

- **Actions & Queries**: `docs/project/specs/stories/[actor]/[story-name]/actions-queries.md`
- **Sequence Diagrams**: `docs/project/specs/stories/[actor]/[story-name]/sequence-diagram.puml`
- **Entity Schemas**: `docs/project/specs/entities/[entity].md`
- **Data Modeling Guide**: `docs/guides/data-modeling/faux-sql-design.md`

---

## Future Enhancements

- **Phase 1b**: Add pagination parameters and responses
- **Phase 2**: Add authentication/authorization schemas
- **Phase 2**: Add webhook event schemas
- **Phase 3**: Add admin endpoints and schemas
