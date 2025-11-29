# Template Conversion Progress Log

**Purpose:** Track progress on converting svc-merchants to a reusable AI agentic backend template.

**Reference:** [02-template-conversion-plan.md](./02-template-conversion-plan.md)

**Status:** In Progress  
**Started:** November 29, 2025  
**Last Updated:** November 29, 2025

---

## Progress Summary

| Phase                                  | Status         | Started      | Completed |
| -------------------------------------- | -------------- | ------------ | --------- |
| Phase 1: Create Template Copy          | 🔄 In Progress | Nov 29, 2025 | -         |
| Phase 2: Identify Template Handles     | ⬜ Not Started | -            | -         |
| Phase 3: Remove Merchant-Specific Code | ⬜ Not Started | -            | -         |
| Phase 4: Add AI-Specific Constructs    | ⬜ Not Started | -            | -         |
| Phase 5: Update Documentation          | ⬜ Not Started | -            | -         |
| Phase 6: Create Configuration Guide    | ⬜ Not Started | -            | -         |

---

## Detailed Progress

### Phase 1: Create Template Copy

**Goal:** Create a separate copy of svc-merchants for template conversion.

#### Task 1.1: Copy Directory

**Status:** ✅ Completed  
**Date:** November 29, 2025

**Actions taken:**

- Copied `svc-merchants/` to `svc-users-template/`
- Location: `/home/nickt/projects/smw/svc-users-template/`
- Command: `cp -r svc-merchants svc-users-template`

**Verification:**

- Directory created with all files intact
- Includes: bin/, lib/, src/, config/, docs/, test/, scripts/
- Includes: package.json, tsconfig.json, cdk.json, etc.

**Notes:**

- Named `svc-users-template` because this will become the users/auth management service template
- The generic microservice template (without Cognito) will be created separately after reviewing the user's existing generic template project

#### Task 1.2: Update Git Configuration

**Status:** ⬜ Pending User Action  
**Date:** November 29, 2025

**Instructions for user:**

1. Create a new GitHub repository:

   - Repository name: `svc-users-template` (or preferred name)
   - Description: "AWS CDK template for user management microservice with Cognito"
   - Visibility: Private (can make public later)
   - Do NOT initialize with README (we have one)

2. Initialize git in the copied directory:

   ```bash
   cd /home/nickt/projects/smw/svc-users-template
   rm -rf .git
   git init
   git add .
   git commit -m "Initial commit: Users service template from svc-merchants"
   ```

3. Add remote and push:
   ```bash
   git remote add origin git@github.com:nickthiru/svc-users-template.git
   git branch -M main
   git push -u origin main
   ```

#### Task 1.3: Initial Commit

**Status:** ⬜ Pending (after Task 1.2)

---

### Phase 2: Identify Template Handles

**Status:** ⬜ Not Started

(Will be updated as we progress)

---

### Phase 3: Remove Merchant-Specific Code

**Status:** ⬜ Not Started

(Will be updated as we progress)

---

### Phase 4: Add AI-Specific Constructs

**Status:** ⬜ Not Started

(Will be updated as we progress)

---

### Phase 5: Update Documentation

**Status:** ⬜ Not Started

(Will be updated as we progress)

---

### Phase 6: Create Configuration Guide

**Status:** ⬜ Not Started

(Will be updated as we progress)

---

## Decisions Made

| Date         | Decision                                        | Rationale                                                                       |
| ------------ | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| Nov 29, 2025 | Name template copy `svc-users-template`         | This becomes the users/auth service template; generic template will be separate |
| Nov 29, 2025 | Create separate GitHub repo                     | Enables GitHub template feature, clean history                                  |
| Nov 29, 2025 | Include both Pinecone and OpenSearch constructs | Flexibility for dev (free tier) vs production (AWS-native)                      |

---

## Blockers & Issues

| Date | Issue | Status | Resolution |
| ---- | ----- | ------ | ---------- |
| -    | -     | -      | -          |

---

## Notes & Observations

### November 29, 2025

**Template Strategy Clarification:**

The user has two template types in mind:

1. **Users Service Template** (`svc-users-template`)

   - Based on current svc-merchants
   - Includes Cognito (User Pool, Identity Pool, Groups)
   - One instance per application (shared auth)
   - Will be used for AI agentic apps

2. **Generic Microservice Template** (to be reviewed)
   - User has an existing template project without Cognito
   - For domain-specific microservices
   - Will be reviewed and merged with learnings from svc-merchants

**Next Steps:**

1. Complete Phase 1 (copy + git setup)
2. User will add generic template to workspace
3. Review generic template and identify differences
4. Merge improvements from svc-merchants into generic template
5. Continue with Phase 2+ on svc-users-template

---

## Related Documents

- [Backend Infrastructure Adoption](./01-backend-infrastructure-adoption.md)
- [Template Conversion Plan](./02-template-conversion-plan.md)
- [svc-merchants Documentation](../../svc-merchants/docs/)
