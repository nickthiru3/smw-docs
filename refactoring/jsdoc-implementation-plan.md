# JSDoc Implementation Plan: users-ms & deals-ms

Strategy for documenting both services simultaneously, leveraging shared files.

---

## File Overlap Analysis

### ✅ **Identical Files** (Can document once, apply to both)

| File | users-ms | deals-ms | Status |
|------|----------|----------|--------|
| `src/helpers/api.ts` | ✅ Exists | ✅ Exists | **IDENTICAL** |
| `src/helpers/ssm.ts` | ✅ Exists | ✅ Exists | **IDENTICAL** |

**Strategy:** Document once in users-ms, then copy to deals-ms

### ⚠️ **Similar Files** (Need separate documentation)

| File | users-ms | deals-ms | Difference |
|------|----------|----------|------------|
| `src/helpers/config.ts` | ✅ Exists | ✅ Exists | **DIFFERENT** (service-specific) |

**Strategy:** Document separately (service-specific config)

### 📝 **Unique to users-ms**

| File | Reason |
|------|--------|
| `src/helpers/email.ts` | Email functionality only in users-ms |
| `lib/auth/**/*` | Authentication only in users-ms |
| `lib/api/endpoints/users/**/*` | User endpoints only in users-ms |

**Strategy:** Document only in users-ms

### 📝 **Unique to deals-ms**

| File | Reason |
|------|--------|
| `lib/api/endpoints/deals/**/*` | Deal endpoints only in deals-ms |

**Strategy:** Will document when we work on deals-ms

---

## Implementation Strategy

### **Approach: Simultaneous Documentation**

Document shared files once, apply to both services immediately. This:
- ✅ Saves time (no duplication of effort)
- ✅ Ensures consistency across services
- ✅ Makes future template creation easier
- ✅ Reduces cognitive load (think once, apply twice)

---

## Phase-by-Phase Plan

### **Phase 1: Shared Helpers** (2 hours)

Document these files once, apply to both services:

1. **`src/helpers/api.ts`** (30 min)
   - Document in users-ms
   - Copy to deals-ms
   - Verify identical

2. **`src/helpers/ssm.ts`** (1 hour)
   - Document in users-ms
   - Copy to deals-ms
   - Verify identical

3. **`src/helpers/config.ts`** (30 min)
   - Document in users-ms (users-specific)
   - Document in deals-ms (deals-specific)
   - Different content, same structure

**Deliverable:** All shared helpers documented in both services

---

### **Phase 2: users-ms Unique Helpers** (1 hour)

Document users-ms specific files:

1. **`src/helpers/email.ts`** (1 hour)
   - Only in users-ms
   - Email sending logic

**Deliverable:** All users-ms helpers documented

---

### **Phase 3: users-ms Endpoint Helpers** (1.5 hours)

Document users-ms endpoint helpers:

1. **`lib/api/endpoints/bindings/helpers.ts`** (45 min)
   - Bindings logic
   - Could be useful for deals-ms template

2. **`lib/auth/user-pool/welcome-email/handler.ts`** (45 min)
   - Email handler
   - users-ms specific

**Deliverable:** All users-ms endpoint helpers documented

---

### **Phase 4: users-ms Key Constructs** (4 hours)

Document users-ms infrastructure:

1. **`lib/service-stack.ts`** (1 hour)
   - Main stack orchestration
   - Similar structure in deals-ms

2. **`lib/auth/construct.ts`** (30 min)
   - users-ms specific

3. **`lib/auth/user-pool/construct.ts`** (1 hour)
   - users-ms specific

4. **`lib/auth/user-pool/welcome-email/construct.ts`** (45 min)
   - users-ms specific

5. **`lib/db/construct.ts`** (45 min)
   - **Similar in deals-ms** - can reuse structure

6. **`lib/api/construct.ts`** (45 min)
   - **Similar in deals-ms** - can reuse structure

7. **`lib/monitor/construct.ts`** (30 min)
   - **Similar in deals-ms** - can reuse structure

8. **`lib/ssm-publications/construct.ts`** (30 min)
   - **Similar in deals-ms** - can reuse structure

**Deliverable:** All users-ms key constructs documented

---

### **Phase 5: deals-ms Unique Files** (2 hours)

After completing users-ms, document deals-ms specific files:

1. **`lib/api/endpoints/deals/post/helpers.ts`** (1 hour)
   - Deal creation logic
   - Similar structure to users-ms helpers

2. **`lib/api/endpoints/deals/post/construct.ts`** (30 min)
   - Already has some JSDoc
   - Enhance to match users-ms quality

3. **`lib/service-stack.ts`** (30 min)
   - Similar to users-ms
   - Adapt documentation

**Deliverable:** All deals-ms files documented

---

## Execution Order

### **Day 1: Shared Foundation** (2 hours)

```
✅ Phase 1: Shared Helpers
   - src/helpers/api.ts (both services)
   - src/helpers/ssm.ts (both services)
   - src/helpers/config.ts (both services, different content)
```

**Why first:** These are used everywhere, foundational

---

### **Day 2: users-ms Helpers** (2.5 hours)

```
✅ Phase 2: users-ms Unique Helpers
   - src/helpers/email.ts

✅ Phase 3: users-ms Endpoint Helpers
   - lib/api/endpoints/bindings/helpers.ts
   - lib/auth/user-pool/welcome-email/handler.ts
```

**Why second:** Complete all helper documentation before constructs

---

### **Day 3-4: users-ms Constructs** (4 hours)

```
✅ Phase 4: users-ms Key Constructs
   - lib/service-stack.ts
   - lib/auth/construct.ts
   - lib/auth/user-pool/construct.ts
   - lib/auth/user-pool/welcome-email/construct.ts
   - lib/db/construct.ts
   - lib/api/construct.ts
   - lib/monitor/construct.ts
   - lib/ssm-publications/construct.ts
```

**Why third:** Infrastructure documentation after helpers

---

### **Day 5: deals-ms Unique** (2 hours)

```
✅ Phase 5: deals-ms Unique Files
   - lib/api/endpoints/deals/post/helpers.ts
   - lib/api/endpoints/deals/post/construct.ts
   - lib/service-stack.ts (adapt from users-ms)
```

**Why last:** Leverage patterns from users-ms

---

## File-by-File Breakdown

### **Shared Files (Apply to Both)**

#### 1. `src/helpers/api.ts` ✅ IDENTICAL

**Functions to document:**
- `addCorsHeader()` - CORS header creation
- `apiSuccess()` - Success response formatter
- `apiError()` - Error response formatter
- `serializeErr()` - Error serialization

**Approach:**
1. Add JSDoc to users-ms version
2. Copy entire file to deals-ms
3. Verify with `diff`

---

#### 2. `src/helpers/ssm.ts` ✅ IDENTICAL

**Functions to document:**
- `buildSsmPublicPath()` - Public path construction
- `buildSsmPrivatePath()` - Private path construction
- `readParam()` - Single parameter read
- `readBindings()` - Multiple parameter read
- `publishStringParameters()` - String parameter publishing
- `publishSecureStringParameters()` - Secure parameter publishing
- `readSecureParam()` - Secure parameter read
- `readSecureBindings()` - Multiple secure parameter read

**Approach:**
1. Add JSDoc to users-ms version
2. Copy entire file to deals-ms
3. Verify with `diff`

---

#### 3. `src/helpers/config.ts` ⚠️ DIFFERENT

**Functions to document:**
- `makeEnv()` - Environment object creation
- `makeTags()` - Tag creation
- `makeDescription()` - Description formatting

**Approach:**
1. Document users-ms version
2. Document deals-ms version separately
3. Keep same JSDoc structure, different examples

---

### **users-ms Unique Files**

#### 4. `src/helpers/email.ts`

**Functions to document:**
- Email sending functions
- Template data preparation
- SES integration

---

#### 5. `lib/api/endpoints/bindings/helpers.ts`

**Functions to document:**
- `env()` - Environment variable helper
- `readPublicBindingsFromSSM()` - SSM reading
- `createFallbackBindings()` - Fallback creation

---

#### 6-13. **Constructs** (8 files)

Each construct needs:
- Class-level JSDoc
- Constructor JSDoc
- Method JSDoc (if applicable)

---

### **deals-ms Unique Files**

#### 14. `lib/api/endpoints/deals/post/helpers.ts`

**Functions to document:**
- Deal creation helpers
- Validation logic
- DynamoDB operations

---

#### 15. `lib/api/endpoints/deals/post/construct.ts`

**Already has some JSDoc** - enhance to match users-ms quality

---

## Quality Checklist

For each file, ensure:

- [ ] Class/module-level JSDoc explaining purpose
- [ ] Function/method JSDoc with:
  - [ ] Brief description
  - [ ] Detailed explanation
  - [ ] @param tags for all parameters
  - [ ] @returns tag
  - [ ] @throws tags for exceptions
  - [ ] @example tags where helpful
  - [ ] @see tags for cross-references
- [ ] Consistent formatting
- [ ] No functionality changes (JSDoc only)
- [ ] Verified with `cdk synth` (constructs)
- [ ] Verified with `npm test` (helpers)

---

## Verification Strategy

### **For Shared Files**

```bash
# After documenting in users-ms, verify identical to deals-ms
diff users-ms/src/helpers/api.ts deals-ms/src/helpers/api.ts

# If identical, copy
cp users-ms/src/helpers/api.ts deals-ms/src/helpers/api.ts

# Verify copy successful
diff users-ms/src/helpers/api.ts deals-ms/src/helpers/api.ts
```

### **For Constructs**

```bash
# Verify no functionality changes
cd users-ms
cdk synth > /tmp/before-jsdoc.yaml

# Add JSDoc
# ...

cdk synth > /tmp/after-jsdoc.yaml
diff /tmp/before-jsdoc.yaml /tmp/after-jsdoc.yaml
# Should be empty (no changes)
```

### **For Helpers**

```bash
# Run tests to verify no breakage
npm test
# Should pass with same coverage
```

---

## Time Estimates

| Phase | Files | Time | Cumulative |
|-------|-------|------|------------|
| Phase 1: Shared Helpers | 3 | 2h | 2h |
| Phase 2: users-ms Helpers | 1 | 1h | 3h |
| Phase 3: users-ms Endpoint Helpers | 2 | 1.5h | 4.5h |
| Phase 4: users-ms Constructs | 8 | 4h | 8.5h |
| Phase 5: deals-ms Unique | 3 | 2h | 10.5h |
| **Total** | **17** | **10.5h** | |

**Breakdown:**
- Shared work (both services): 2 hours
- users-ms unique: 6.5 hours
- deals-ms unique: 2 hours

**Efficiency gain:** ~2 hours saved by documenting shared files once

---

## Success Metrics

After completion:

✅ **15 users-ms files** fully documented  
✅ **3 shared files** documented in both services  
✅ **3 deals-ms files** fully documented  
✅ **All tests passing** (no functionality changes)  
✅ **CDK synth identical** (constructs unchanged)  
✅ **Consistent JSDoc style** across all files  
✅ **Ready for template creation** (microservice-template)  

---

## Next Steps After JSDoc

1. ✅ Complete JSDoc for both services
2. ⏳ Assess deals-ms for stratified design opportunities
3. ⏳ Apply stratified design to deals-ms (if needed)
4. ⏳ Update tests if necessary
5. ⏳ Copy deals-ms to microservice-template
6. ⏳ Generalize template for bootstrapping

---

**Last Updated:** October 2025  
**Maintainer:** Development Team  
**Related Docs:**
- [users-ms JSDoc Assessment](./users-ms-jsdoc-assessment.md)
- [Construct Refactoring Assessment](./construct-refactoring-assessment.md)
