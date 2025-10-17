# Data Access Patterns Table Template

## Overview

This template provides a structured format for documenting all access patterns in your DynamoDB data model. Complete this table during Phase 1 of the data modeling process.

**Key Principle:** "You can't design your table until you know how you'll use your data".

## Why This Template?

Documenting access patterns upfront is critical for DynamoDB success:

1. **Required for modeling** - DynamoDB data models are driven entirely by access patterns
2. **Prevents costly mistakes** - Missing patterns discovered later may require expensive migrations
3. **Team alignment** - Creates shared understanding across stakeholders
4. **Implementation guide** - Serves as specification for developers

## Template Structure

### Access Patterns Table

| Entity   | Access Pattern | Index          | Key Condition | Parameters | Frequency    | Notes                  |
| -------- | -------------- | -------------- | ------------- | ---------- | ------------ | ---------------------- |
| [Entity] | [Description]  | Main/GSI1/GSI2 | [Expression]  | [Params]   | High/Med/Low | [Special requirements] |

### Column Definitions

**Entity:** The primary entity being accessed (e.g., User, Order, Product)

**Access Pattern:** Clear description of what data is being retrieved

- Use action verbs: "Get", "List", "Query", "Update"
- Be specific about what's being fetched
- Examples: "Get User by Username", "List Orders for Customer"

**Index:** Which index satisfies this pattern

- `Main` - Base table
- `GSI1`, `GSI2`, etc. - Global secondary indexes
- `LSI1` - Local secondary index (rarely used)
- Leave blank during initial gathering phase

**Key Condition:** The DynamoDB key condition expression

- Format: `PK = <value> AND SK begins_with <value>`
- Examples: `PK = USER#<username>`, `PK = CUSTOMER#<id> AND SK begins_with ORDER#`
- Fill in during Phase 2 after primary key design

**Parameters:** What information client must provide

- List all required parameters
- Examples: `username`, `customerId, startDate, endDate`
- Helps validate if pattern is feasible (client must know these values)

**Frequency:** How often this pattern is used

- `High` - Multiple times per second, critical path
- `Medium` - Regular but not constant usage
- `Low` - Occasional or admin-only operations
- Helps prioritize optimization efforts

**Notes:** Additional context or requirements

- Pagination needs
- Sorting requirements
- Filter conditions
- Consistency requirements (strong vs eventual)
- Special business logic

---

## Example: E-Commerce Application

### Customer Access Patterns

| Entity   | Access Pattern          | Index | Key Condition            | Parameters          | Frequency | Notes                               |
| -------- | ----------------------- | ----- | ------------------------ | ------------------- | --------- | ----------------------------------- |
| Customer | Get Customer by ID      | Main  | PK = CUSTOMER#\<id\>     | customerId          | High      | Used on every authenticated request |
| Customer | Get Customer by Email   | GSI1  | GSI1PK = EMAIL#\<email\> | email               | Medium    | Used during login                   |
| Customer | Update Customer Profile | Main  | PK = CUSTOMER#\<id\>     | customerId, updates | Medium    | Must verify user owns this customer |

### Order Access Patterns

| Entity | Access Pattern           | Index | Key Condition                                          | Parameters         | Frequency | Notes                               |
| ------ | ------------------------ | ----- | ------------------------------------------------------ | ------------------ | --------- | ----------------------------------- |
| Order  | Get Order by ID          | Main  | PK = ORDER#\<orderId\>                                 | orderId            | High      | Direct order lookup                 |
| Order  | List Orders for Customer | Main  | PK = CUSTOMER#\<customerId\> AND SK begins_with ORDER# | customerId, limit  | High      | Customer order history, paginated   |
| Order  | List Recent Orders       | GSI1  | GSI1PK = STATUS#\<status\> AND GSI1SK > \<timestamp\>  | status, startDate  | Medium    | Admin dashboard                     |
| Order  | Update Order Status      | Main  | PK = ORDER#\<orderId\>                                 | orderId, newStatus | Medium    | Requires order fulfillment workflow |

### Product Access Patterns

| Entity  | Access Pattern            | Index | Key Condition                  | Parameters      | Frequency | Notes                                 |
| ------- | ------------------------- | ----- | ------------------------------ | --------------- | --------- | ------------------------------------- |
| Product | Get Product by ID         | Main  | PK = PRODUCT#\<productId\>     | productId       | High      | Product detail page                   |
| Product | List Products by Category | GSI1  | GSI1PK = CATEGORY#\<category\> | category, limit | High      | Category browsing, needs pagination   |
| Product | Search Products by Name   | GSI2  | GSI2PK = SEARCH                | searchTerm      | Medium    | Uses sparse index + filter expression |
| Product | Get Featured Products     | GSI1  | GSI1PK = FEATURED              | none            | High      | Homepage, cacheable                   |

---

## Example: SaaS Application

### Organization Access Patterns

| Entity       | Access Pattern           | Index | Key Condition          | Parameters | Frequency | Notes                          |
| ------------ | ------------------------ | ----- | ---------------------- | ---------- | --------- | ------------------------------ |
| Organization | Get Organization by ID   | Main  | PK = ORG#\<orgId\>     | orgId      | High      | Used in auth middleware        |
| Organization | Get Organization by Slug | GSI1  | GSI1PK = SLUG#\<slug\> | slug       | High      | Tenant identification from URL |
| Organization | List All Organizations   | Main  | Scan                   | none       | Low       | Admin only, rare operation     |

### User Access Patterns

| Entity | Access Pattern             | Index | Key Condition                               | Parameters               | Frequency | Notes                                    |
| ------ | -------------------------- | ----- | ------------------------------------------- | ------------------------ | --------- | ---------------------------------------- |
| User   | Get User by Username       | Main  | PK = USER#\<username\>                      | username                 | High      | Authentication                           |
| User   | Get User by Email          | GSI1  | GSI1PK = EMAIL#\<email\>                    | email                    | Medium    | Password reset flow                      |
| User   | List Users in Organization | Main  | PK = ORG#\<orgId\> AND SK begins_with USER# | orgId, limit             | Medium    | Team management page                     |
| User   | Update User Role           | Main  | PK = USER#\<username\>                      | username, orgId, newRole | Low       | Admin operation, must verify permissions |

### Project Access Patterns

| Entity  | Access Pattern                 | Index | Key Condition                                              | Parameters       | Frequency | Notes                                 |
| ------- | ------------------------------ | ----- | ---------------------------------------------------------- | ---------------- | --------- | ------------------------------------- |
| Project | Get Project by ID              | GSI1  | GSI1PK = PROJECT#\<projectId\>                             | projectId        | High      | Direct access to project              |
| Project | List Projects for Organization | Main  | PK = ORG#\<orgId\> AND SK begins_with PROJECT#             | orgId, limit     | High      | Organization dashboard                |
| Project | List Projects for User         | GSI2  | GSI2PK = USER#\<username\> AND GSI2SK begins_with PROJECT# | username, limit  | Medium    | User's project list                   |
| Project | Archive Project                | Main  | PK = ORG#\<orgId\>, SK = PROJECT#\<projectId\>             | orgId, projectId | Low       | Remove from active list, don't delete |

---

## Example: GitHub-like Application

### Repository Access Patterns

| Entity | Access Pattern             | Index | Key Condition                                   | Parameters      | Frequency | Notes                  |
| ------ | -------------------------- | ----- | ----------------------------------------------- | --------------- | --------- | ---------------------- |
| Repo   | Get Repo by Owner and Name | Main  | PK = ACCOUNT#\<owner\> AND SK = REPO#\<name\>   | owner, repoName | High      | Main repo access       |
| Repo   | List Repos for Account     | Main  | PK = ACCOUNT#\<owner\> AND SK begins_with REPO# | owner, limit    | High      | User/org profile page  |
| Repo   | Get Repo by ID             | GSI1  | GSI1PK = REPO#\<repoId\>                        | repoId          | Medium    | Webhook callbacks, API |

### Issue Access Patterns

| Entity | Access Pattern              | Index | Key Condition                                             | Parameters          | Frequency | Notes                                 |
| ------ | --------------------------- | ----- | --------------------------------------------------------- | ------------------- | --------- | ------------------------------------- |
| Issue  | Get Issue by Number         | Main  | PK = REPO#\<repoId\> AND SK = ISSUE#\<status\>#\<number\> | repoId, issueNumber | High      | Need to know status or query multiple |
| Issue  | Get Issue by ID             | GSI1  | GSI1PK = ISSUE#\<issueId\>                                | issueId             | Medium    | Direct lookup                         |
| Issue  | List Open Issues for Repo   | Main  | PK = REPO#\<repoId\> AND SK begins_with ISSUE#OPEN#       | repoId, limit       | High      | Issue list page, sorted by number     |
| Issue  | List Closed Issues for Repo | Main  | PK = REPO#\<repoId\> AND SK begins_with ISSUE#CLOSED#     | repoId, limit       | Medium    | Closed issues tab                     |
| Issue  | List Issues Created by User | GSI2  | GSI2PK = USER#\<username\> AND GSI2SK begins_with ISSUE#  | username, limit     | Low       | User activity page                    |

### Comment Access Patterns

| Entity  | Access Pattern          | Index | Key Condition                                              | Parameters      | Frequency | Notes               |
| ------- | ----------------------- | ----- | ---------------------------------------------------------- | --------------- | --------- | ------------------- |
| Comment | Get Comment by ID       | GSI1  | GSI1PK = COMMENT#\<commentId\>                             | commentId       | Medium    | Direct comment link |
| Comment | List Comments for Issue | Main  | PK = ISSUE#\<issueId\> AND SK begins_with COMMENT#         | issueId, limit  | High      | Issue detail page   |
| Comment | List Comments by User   | GSI2  | GSI2PK = USER#\<username\> AND GSI2SK begins_with COMMENT# | username, limit | Low       | User activity feed  |

---

## Filling Out This Template

### Phase 1: Access Pattern Gathering

**Step 1: Identify all entities**

- List every noun in your application
- Include main entities and supporting entities
- Don't worry about DynamoDB structure yet

**Step 2: List all read operations**

- For each entity, list every way it will be accessed
- Think about: detail pages, list pages, search, filters
- Consider both user-facing and internal operations

**Step 3: List all write operations**

- Create, update, delete operations
- Batch operations
- Background jobs

**Step 4: Specify parameters**

- What information does client have?
- What will be in the URL or request body?
- Validate client will know these values at request time

**Step 5: Estimate frequency**

- Which patterns are on the critical path?
- Which are used most often?
- Which are administrative or rare?

**At this stage, leave Index and Key Condition blank!**

### Phase 2: Data Model Design

Once you have all access patterns documented:

**Step 6: Design primary key**

- Create entity chart
- Design PK and SK patterns for each entity
- See [Guide 3: Primary Key Design](../guides/3-primary-key-design.md)

**Step 7: Map patterns to indexes**

- Fill in "Index" column for each pattern
- Start with Main table
- Add GSIs only when needed
- See [Guide 6: Secondary Index Strategies](../guides/6-secondary-index-strategies.md)

**Step 8: Write key conditions**

- Specify exact DynamoDB expressions
- Validate patterns are achievable
- Document any filter expressions needed

---

## Validation Checklist

Before finalizing your access patterns table:

- [ ] All entities from ERD are covered
- [ ] Every screen/API endpoint has corresponding patterns
- [ ] Parameters are specified for each pattern
- [ ] Client will know all parameters at request time
- [ ] Frequency estimates provided
- [ ] Write operations include permission requirements
- [ ] Special requirements noted (sorting, filtering, pagination)
- [ ] Stakeholders have reviewed and approved
- [ ] No patterns use Scan in application code
- [ ] No patterns require multiple serial requests

---

## Anti-Patterns to Avoid

### ❌ Vague Descriptions

**Bad:** "Get user stuff"  
**Good:** "Get User by Username"

### ❌ Missing Parameters

**Bad:** "List orders"  
**Good:** "List Orders for Customer (customerId, limit)"

### ❌ Assuming Flexibility

**Bad:** "Query products by anything"  
**Good:** Separate patterns: "Get Product by ID", "List Products by Category", "Search Products by Name"

### ❌ Including Implementation Too Early

**Bad:** Filling in Index/Key Condition before all patterns gathered  
**Good:** Complete all patterns first, then design table

### ❌ Forgetting Write Patterns

**Bad:** Only documenting reads  
**Good:** Include creates, updates, deletes, and their requirements

---

## Next Steps

After completing this template:

1. **Review with stakeholders** - Ensure nothing is missing
2. **Proceed to Phase 2** - Design primary keys and indexes
3. **Fill in technical columns** - Add Index and Key Condition details
4. **Validate against anti-patterns** - Review [Guide 7: Common Anti-Patterns](./guides/7-common-anti-patterns.md)
5. **Begin implementation** - Use as specification for development

---

## Additional Resources

- [Phase 1: Experience & Domain Design](../phases/1-experience-and-domain-design.md)
- [Guide 2: Access Patterns Definition](../guides/2-access-patterns-definition.md)
- [Guide 3: Primary Key Design](../guides/3-primary-key-design.md)
- [Quick Reference Cards](../quick-reference-cards.md)

---

## Blank Template

Copy this for your project:

| Entity | Access Pattern | Index | Key Condition | Parameters | Frequency | Notes |
| ------ | -------------- | ----- | ------------- | ---------- | --------- | ----- |
|        |                |       |               |            |           |       |
|        |                |       |               |            |           |       |
|        |                |       |               |            |           |       |
|        |                |       |               |            |           |       |
|        |                |       |               |            |           |       |

**Remember:** This document will evolve as you design your data model. Start with all patterns documented, even if you don't know how to implement them yet!
