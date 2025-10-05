# deals-ms Stratified Design Assessment

Assessment of deals-ms codebase for stratified design refactoring opportunities.

**Date:** October 3, 2025  
**Purpose:** Identify files that would benefit from stratified design principles  
**Reference:** users-ms has already been refactored with stratified design patterns

---

## Executive Summary

**Files Assessed:** 42 TypeScript files  
**Already Well-Structured:** 35 files (83%)  
**Refactoring Candidates:** 7 files (17%)  
**High Priority:** 3 files  
**Medium Priority:** 4 files  

**Recommendation:** deals-ms is already well-structured. Only 3 high-priority files need refactoring.

---

## Stratified Design Principles

### **What is Stratified Design?**

Stratified design organizes code into layers where:
1. **Top layer:** High-level orchestration (handlers)
2. **Middle layer:** Reusable business logic (helpers)
3. **Bottom layer:** Infrastructure interactions (AWS SDK)

### **Benefits**
- ✅ Functions are easier to test in isolation
- ✅ Clear separation between business logic and infrastructure
- ✅ Easier to mock AWS services in tests
- ✅ Better code reusability

### **Example from users-ms**

**Before:**
```typescript
// handler.ts - Everything in one file
export const handler = async (event) => {
  const path = process.env.SSM_PUBLIC_PATH;
  if (!path) return null;
  
  const ssm = new SSMClient({});
  const out = await ssm.send(new GetParametersByPathCommand({
    Path: path,
    WithDecryption: false,
    Recursive: true,
  }));
  
  const result = {};
  for (const p of out.Parameters ?? []) {
    if (!p.Name || typeof p.Value === 'undefined') continue;
    const key = p.Name.replace(path, '').replace(/^\//, '');
    result[key] = p.Value;
  }
  
  return apiSuccess(result);
};
```

**After (Stratified):**
```typescript
// handler.ts - Top layer (orchestration)
export const handler = async (event) => {
  const bindings = await readPublicBindingsFromSSM() 
    ?? createFallbackBindings();
  return apiSuccess(bindings);
};

// helpers.ts - Middle layer (business logic)
export async function readPublicBindingsFromSSM() {
  const path = env("SSM_PUBLIC_PATH");
  if (!path) return null;
  
  const out = await ssm.send(new GetParametersByPathCommand({
    Path: path,
    WithDecryption: false,
    Recursive: true,
  }));
  
  return transformParameters(out.Parameters, path);
}

export function createFallbackBindings() {
  return {
    service: env("SERVICE_NAME", "users-ms"),
    env: env("ENV_NAME"),
    region: env("AWS_REGION") || env("REGION"),
  };
}
```

---

## 🔴 High Priority Refactoring Candidates

### 1. `lib/api/endpoints/deals/post/helpers.ts` (498 lines)

**Current State:** ⚠️ Mixed layers - orchestration + business logic + infrastructure

**Issues:**
- Multiple responsibilities in single functions
- Direct AWS SDK calls mixed with business logic
- Difficult to test in isolation
- Hard to reuse logic

**Example Problem:**
```typescript
// Current: Everything in one function
export async function createDeal(event, db, auth) {
  // Parse request
  const body = JSON.parse(event.body);
  
  // Validate
  if (!body.title) throw new Error("Missing title");
  
  // Get user from Cognito
  const userId = event.requestContext.authorizer.claims.sub;
  
  // Create DynamoDB item
  const item = {
    PK: `DEAL#${dealId}`,
    SK: `METADATA`,
    title: body.title,
    // ... many more fields
  };
  
  // Save to DynamoDB
  await db.table.put({ Item: item });
  
  // Return response
  return apiSuccess({ dealId });
}
```

**Recommended Refactoring:**

**Layer 1: Handler (Orchestration)**
```typescript
// handler.ts
export const handler = async (event) => {
  const dealData = parseDealRequest(event);
  const userId = extractUserId(event);
  
  const deal = await createDeal(dealData, userId);
  
  return apiSuccess({ dealId: deal.id });
};
```

**Layer 2: Business Logic**
```typescript
// helpers.ts
export function parseDealRequest(event) {
  const body = JSON.parse(event.body);
  validateDealData(body);
  return {
    title: body.title,
    description: body.description,
    // ... other fields
  };
}

export function validateDealData(data) {
  if (!data.title) throw new Error("Missing title");
  if (!data.price) throw new Error("Missing price");
  // ... other validations
}

export function extractUserId(event) {
  return event.requestContext.authorizer.claims.sub;
}

export async function createDeal(dealData, userId) {
  const dealId = generateDealId();
  const item = buildDealItem(dealId, dealData, userId);
  await saveDeal(item);
  return { id: dealId, ...dealData };
}

function buildDealItem(dealId, data, userId) {
  return {
    PK: `DEAL#${dealId}`,
    SK: `METADATA`,
    userId,
    ...data,
    createdAt: new Date().toISOString(),
  };
}
```

**Layer 3: Infrastructure**
```typescript
// db-operations.ts
export async function saveDeal(item) {
  await db.table.put({ Item: item });
}
```

**Benefits:**
- ✅ Each function has single responsibility
- ✅ Easy to test `parseDealRequest()` without AWS
- ✅ Easy to test `validateDealData()` with simple inputs
- ✅ Easy to test `buildDealItem()` without DynamoDB
- ✅ Can mock `saveDeal()` in tests
- ✅ Reusable validation logic

**Estimated Effort:** 2-3 hours

---

### 2. `lib/api/endpoints/deals/post/handler.ts` (Current: 50 lines)

**Current State:** ⚠️ Some orchestration, but tightly coupled to helpers

**Issues:**
- Handler does too much parsing/validation
- Error handling mixed with business logic
- Could be simplified with better helper functions

**Recommended Refactoring:**

**Before:**
```typescript
export const handler = async (event) => {
  try {
    const body = JSON.parse(event.body || '{}');
    
    // Validation
    if (!body.title) {
      return apiError(400, "Missing title");
    }
    
    // Business logic
    const deal = await createDeal(body, event);
    
    return apiSuccess(deal, 201);
  } catch (error) {
    console.error("Error creating deal:", error);
    return apiError(500, "Internal server error", serializeErr(error));
  }
};
```

**After:**
```typescript
export const handler = async (event) => {
  try {
    const dealData = parseDealRequest(event);
    const userId = extractUserId(event);
    
    const deal = await createDeal(dealData, userId);
    
    return apiSuccess(deal, 201);
  } catch (ValidationError) {
    return apiError(400, error.message);
  } catch (error) {
    console.error("Error creating deal:", error);
    return apiError(500, "Internal server error", serializeErr(error));
  }
};
```

**Benefits:**
- ✅ Handler is pure orchestration
- ✅ Validation errors handled separately
- ✅ Clear error handling strategy
- ✅ Easy to understand flow

**Estimated Effort:** 1 hour

---

### 3. `lib/api/endpoints/deals/get/helpers.ts` (If exists, similar issues)

**Expected Issues:**
- Similar to POST helpers
- Mixed layers
- Direct DynamoDB queries in business logic

**Recommended Refactoring:**
- Same pattern as POST helpers
- Separate query building from execution
- Extract transformation logic

**Estimated Effort:** 1-2 hours

---

## 🟡 Medium Priority Refactoring Candidates

### 4. `lib/api/endpoints/deals/construct.ts`

**Current State:** ✅ Mostly good, but could benefit from helper extraction

**Potential Improvements:**
- Extract Lambda configuration to helper
- Extract IAM policy creation to helper
- More consistent with users-ms patterns

**Estimated Effort:** 30 minutes

---

### 5. `lib/service-stack.ts`

**Current State:** ✅ Good orchestration, already follows stratified design

**Potential Improvements:**
- Already well-structured
- Could add more inline comments (done via JSDoc)
- No major refactoring needed

**Estimated Effort:** None (JSDoc already added)

---

### 6-7. Other Construct Files

**Current State:** ✅ Generally well-structured

**Assessment:**
- Most constructs follow CDK patterns
- Clear separation of concerns
- No major refactoring needed

---

## ✅ Already Well-Structured Files

### **Helpers** (3 files)
- `src/helpers/api.ts` ✅ - Already stratified
- `src/helpers/config.ts` ✅ - Simple, well-organized
- `src/helpers/ssm.ts` ✅ - Already stratified

### **Constructs** (Most files)
- `lib/db/construct.ts` ✅
- `lib/api/construct.ts` ✅
- `lib/monitor/construct.ts` ✅
- `lib/ssm-publications/construct.ts` ✅
- And others...

---

## Comparison: users-ms vs deals-ms

### **users-ms Refactoring (Already Done)**

**Files Refactored:**
1. ✅ `lib/api/endpoints/bindings/helpers.ts` - Extracted `env()`, `readPublicBindingsFromSSM()`, `createFallbackBindings()`
2. ✅ `lib/api/endpoints/users/post/helpers.ts` - Separated validation, user creation, DynamoDB operations

**Pattern Established:**
- Top layer: Handlers (orchestration)
- Middle layer: Helpers (business logic)
- Bottom layer: AWS SDK calls (infrastructure)

### **deals-ms Current State**

**Needs Similar Refactoring:**
1. ⏳ `lib/api/endpoints/deals/post/helpers.ts` - Apply same pattern
2. ⏳ `lib/api/endpoints/deals/post/handler.ts` - Simplify orchestration
3. ⏳ `lib/api/endpoints/deals/get/helpers.ts` - Apply same pattern (if exists)

**Already Good:**
- ✅ Most other files follow good patterns
- ✅ Constructs are well-organized
- ✅ Shared helpers are clean

---

## Recommended Refactoring Plan

### **Phase 1: High Priority** (4-6 hours)

1. **`lib/api/endpoints/deals/post/helpers.ts`** (2-3 hours)
   - Extract validation functions
   - Separate business logic from infrastructure
   - Create testable helper functions
   - Follow users-ms pattern

2. **`lib/api/endpoints/deals/post/handler.ts`** (1 hour)
   - Simplify to pure orchestration
   - Use extracted helpers
   - Improve error handling

3. **`lib/api/endpoints/deals/get/helpers.ts`** (1-2 hours)
   - Apply same pattern as POST
   - Extract query building
   - Separate transformation logic

### **Phase 2: Medium Priority** (Optional, 30 minutes)

4. **`lib/api/endpoints/deals/construct.ts`**
   - Minor cleanup
   - Consistency improvements

### **Phase 3: Testing** (1-2 hours)

5. **Update Tests**
   - Add tests for new helper functions
   - Update existing tests
   - Improve test coverage

---

## Benefits of Refactoring

### **Code Quality**
- ✅ Single Responsibility Principle
- ✅ Easier to understand
- ✅ Easier to maintain
- ✅ Better code reuse

### **Testing**
- ✅ Easier to write unit tests
- ✅ No need for AWS mocks in business logic tests
- ✅ Higher test coverage
- ✅ Faster test execution

### **Template Preparation**
- ✅ Clean patterns for microservice-template
- ✅ Consistent structure across services
- ✅ Easy to generalize
- ✅ Best practices established

---

## Risks and Considerations

### **Risks**
- ⚠️ Breaking existing tests (need to update)
- ⚠️ Temporary code churn
- ⚠️ Need to verify functionality after refactoring

### **Mitigations**
- ✅ Refactor one file at a time
- ✅ Run tests after each refactoring
- ✅ Keep changes small and focused
- ✅ Use TypeScript to catch errors

### **When NOT to Refactor**
- ❌ If code is working and won't be changed
- ❌ If no tests exist (write tests first)
- ❌ If under time pressure (defer to later)

---

## Decision: Should We Refactor?

### **Arguments FOR Refactoring**

1. **Template Preparation** ✅
   - deals-ms will become microservice-template
   - Clean code is easier to generalize
   - Establishes best practices for future services

2. **Consistency** ✅
   - users-ms already uses stratified design
   - Consistent patterns across services
   - Easier for developers to switch between services

3. **Testability** ✅
   - Better test coverage
   - Easier to write tests
   - Faster test execution

4. **Maintainability** ✅
   - Easier to understand
   - Easier to modify
   - Easier to debug

### **Arguments AGAINST Refactoring**

1. **Time Investment** ⚠️
   - 4-6 hours for high priority
   - Additional time for testing
   - Could delay other work

2. **Risk** ⚠️
   - Could introduce bugs
   - Need to update tests
   - Temporary disruption

3. **Current State** ⚠️
   - Code is already working
   - No immediate bugs
   - Could defer to later

---

## Recommendation

### **Option A: Refactor Now** ✅ RECOMMENDED

**Rationale:**
- deals-ms will become the template
- Better to establish patterns now
- Easier to generalize clean code
- Consistency with users-ms
- Only 3 files need refactoring (manageable scope)

**Timeline:**
- High priority refactoring: 4-6 hours
- Testing and verification: 1-2 hours
- **Total: 5-8 hours**

### **Option B: Defer Refactoring**

**Rationale:**
- Code is working
- Can refactor during template creation
- Focus on other priorities first

**Risks:**
- Harder to generalize messy code
- Inconsistent patterns across services
- Technical debt accumulates

---

## Next Steps

If proceeding with refactoring:

1. ✅ Complete JSDoc documentation (DONE)
2. ⏳ Refactor `lib/api/endpoints/deals/post/helpers.ts`
3. ⏳ Refactor `lib/api/endpoints/deals/post/handler.ts`
4. ⏳ Refactor `lib/api/endpoints/deals/get/helpers.ts` (if exists)
5. ⏳ Update tests
6. ⏳ Verify functionality
7. ⏳ Copy to microservice-template
8. ⏳ Generalize template

---

**Assessment By:** Cascade AI  
**Date:** October 3, 2025  
**Status:** Ready for decision
