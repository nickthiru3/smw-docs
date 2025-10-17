# DynamoDB Data Modeling Guide

## Overview

This comprehensive guide provides a structured approach to designing efficient DynamoDB data models. Whether you're starting a new project or migrating an existing application, these guides will help you build scalable, performant DynamoDB tables.

**Key Principle:** "You can't design your table until you know how you'll use your data".

## Quick Start

**New to DynamoDB?** Start here:

1. Read [Phase 1: Experience & Domain Design](phases/1-experience-and-domain-design.md)
2. Review the [Quick Reference Cards](quick-reference-cards.md)
3. Study a complete [Data Modeling Example](../specs/)

**Experienced with DynamoDB?** Jump to:

- [Migration Strategies Guide](guides/8-migration-strategies.md)
- [Anti-Patterns Guide](guides/7-common-anti-patterns.md)
- [Quick Reference Cards](quick-reference-cards.md)

---

## Documentation Structure

### 📋 Process Guides (Phases)

Follow these phases sequentially when building a new data model:

| Phase                        | Description                | Duration | Key Deliverables                    |
| ---------------------------- | -------------------------- | -------- | ----------------------------------- |
| [Phase 1](phases/1-experience-and-domain-design.md) | Experience & Domain Design | 3-5 days | ERD, Access Patterns Table          |
| [Phase 2](phases/2-data-modeling.md) | Data Modeling              | 5-7 days | Entity Charts, Schemas, GSI Designs |
| [Phase 3](phases/3-implementation.md) | Implementation             | Ongoing  | CDK Code, Data Access Layer         |

### 📖 Strategy Guides

Reference these guides for specific modeling challenges:

| Guide                                          | Topic                   | Use When                |
| ---------------------------------------------- | ----------------------- | ----------------------- |
| [Guide 1](guides/1-erd-creation.md)            | ERD Creation            | Starting new model      |
| [Guide 2](guides/2-access-patterns.md)         | Access Patterns         | Defining queries        |
| [Guide 3](guides/3-primary-key-design.md)      | Primary Key Design      | Designing table keys    |
| [Guide 4](guides/4-entity-schema.md)           | Entity Schema           | Documenting attributes  |
| [Guide 5](guides/5-relationship-strategies.md) | Relationship Strategies | Modeling 1:1, 1:N, M:N  |
| [Guide 6](guides/6-secondary-indexes.md)       | Secondary Indexes       | Adding GSIs             |
| [Guide 7](guides/7-common-anti-patterns.md)    | Anti-Patterns           | Reviewing design        |
| [Guide 8](guides/8-migration-strategies.md)    | Migration Strategies    | Changing existing model |

### 🎯 Quick References

- [Quick Reference Cards](quick-reference-cards.md) - Cheat sheets for common scenarios
- [Artifact Mapping Table](artifact-mapping-table.md) - Track all modeling artifacts

---

## Core Concepts

### The DynamoDB Modeling Mindset

DynamoDB modeling is fundamentally different from relational database modeling:

**Relational Databases:**

- Design entities and relationships first
- Write queries later
- Normalize to avoid duplication
- Use JOINs to combine data

**DynamoDB:**

- Define access patterns first
- Design table to satisfy those patterns
- Denormalize for performance
- Pre-join data in item collections

### Key Principles

1. **Access Patterns First**

   - You must know your queries before modeling
   - Document every access pattern with specific parameters
   - No "flexible querying" like SQL SELECT \*

2. **Pre-Join Your Data**

   - Fetch related items in single request
   - Group related entities in item collections
   - Avoid multiple serial requests

3. **Use Generic Key Names**

   - PK, SK instead of CustomerId, OrderDate
   - Enables key overloading
   - Easier to evolve over time

4. **Overload Your Indexes**

   - Don't create one GSI per pattern
   - Handle multiple entity types per index
   - Use generic GSI attribute names (GSI1PK, GSI1SK)

5. **Storage is Cheap, Compute is Expensive**
   - Denormalize aggressively
   - Duplicate data to avoid additional reads
   - Don't optimize for storage at expense of performance

---

## Common Patterns

### One-to-Many Relationships

| Scenario            | Strategy                     | Guide Reference                                   |
| ------------------- | ---------------------------- | ------------------------------------------------- |
| Bounded (<20 items) | Complex attribute (List/Map) | [Guide 5](guides/5-relationship-strategies.md)    |
| Unbounded           | Composite primary key        | [Guide 3](guides/3-primary-key-design.md)         |
| Need reverse lookup | Secondary index              | [Guide 6](guides/6-secondary-index-strategies.md) |
| Hierarchical data   | Composite sort key           | [Guide 5](guides/5-relationship-strategies.md)    |

### Many-to-Many Relationships

| Scenario               | Strategy                          | Guide Reference                                |
| ---------------------- | --------------------------------- | ---------------------------------------------- |
| Limited related data   | Shallow duplication               | [Guide 5](guides/5-relationship-strategies.md) |
| Bidirectional queries  | Adjacency list                    | [Guide 5](guides/5-relationship-strategies.md) |
| Highly connected graph | Materialized graph                | [Guide 5](guides/5-relationship-strategies.md) |
| Highly mutable data    | Normalization (multiple requests) | [Guide 5](guides/5-relationship-strategies.md) |

### Migration Patterns

| Change Type            | Strategy            | Additive? | Guide Reference                             |
| ---------------------- | ------------------- | --------- | ------------------------------------------- |
| Add optional attribute | Lazy loading        | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md) |
| Add independent entity | New item collection | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md) |
| Add related entity     | Share partition key | ✅ Yes    | [Guide 8](guides/8-migration-strategies.md) |
| Add entity needing GSI | Create GSI + ETL    | ❌ No     | [Guide 8](guides/8-migration-strategies.md) |
| Refactor pattern       | New indexes + ETL   | ❌ No     | [Guide 8](guides/8-migration-strategies.md) |

---

## Anti-Patterns to Avoid

### ❌ Common Mistakes

**Access Patterns:**

- Using Scan in application code
- Filter expressions as primary access mechanism
- Multiple serial requests for related data

**Key Design:**

- Timestamps in keys unless client knows them
- Descriptive key names (CustomerId, OrderDate)
- Reusing attributes across indexes
- Missing Type attribute on items

**Relationships:**

- Not pre-joining data in item collections
- Unbounded lists in single items
- Normalizing data to save storage

**Architecture:**

- Using an ORM
- Data access not at application boundary
- Not separating indexing from application attributes
- Multiple tables for related entities

→ See [Guide 7: Anti-Patterns](guides/7-common-anti-patterns.md) for details

---

## Typical Workflow

### For New Applications

```
1. Understand Application
   ↓
2. Create ERD (Guide 1)
   ↓
3. Define Access Patterns (Guide 2)
   ↓
4. Design Primary Keys (Guide 3)
   ↓
5. Model Relationships (Guide 5)
   ↓
6. Define Entity Schemas (Guide 4)
   ↓
7. Design Secondary Indexes (Guide 6)
   ↓
8. Validate Against Anti-Patterns (Guide 7)
   ↓
9. Implement (Phase 3)
```

### For Migrations

```
1. Identify Change Type
   ↓
2. Choose Migration Strategy (Guide 8)
   ↓
3. Design New/Updated Schema
   ↓
4. Plan ETL (if needed)
   ↓
5. Test on Copy of Data
   ↓
6. Execute Migration
   ↓
7. Validate & Monitor
```

---

## Repository Structure

Your data modeling artifacts should be organized as follows:

```
docs/specs/jobs/<job-name>/
├── erd.puml                        # ERD diagram
├── data-access-patterns.md         # Access patterns table
├── entity-chart-main.md            # Main table entity chart
├── entity-chart-gsi1.md            # GSI1 entity chart
├── entity-chart-gsi2.md            # GSI2 entity chart (if needed)
├── entity-schemas.ts               # TypeScript entity interfaces
├── relationship-decisions.md       # Relationship strategy docs
└── migration-plan.md               # Migration strategy (if applicable)
```

See [Artifact Mapping Table](artifact-mapping-table.md) for complete details.

---

## Learning Resources

### Internal Documentation

- [Phase Documentation](phases/) - Step-by-step process guides
- [Strategy Guides](guides/) - Deep dives on specific patterns
- [Quick Reference Cards](quick-reference-cards.md) - At-a-glance help
- [Example Models](../specs/) - Complete working examples

### External Resources

- **The DynamoDB Book** by Alex DeBrie - Primary reference for this guide
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/) - Official AWS documentation
- [NoSQL Workbench](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html) - Visual data modeling tool
- [Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html) - AWS best practices

---

## Getting Help

### Decision Trees

**"I need to..."**

- Start a new data model → [Phase 1](phases/1-experience-and-domain-design.md)
- Define my queries → [Guide 2](guides/2-access-patterns-definition.md)
- Design my keys → [Guide 3](guides/3-primary-key-design.md)
- Model relationships → [Guide 5](guides/5-relationship-strategies.md)
- Add a GSI → [Guide 6](guides/6-secondary-index-strategies.md)
- Migrate existing model → [Guide 8](guides/8-migration-strategies.md)
- Check if my design is correct → [Guide 7](guides/7-common-anti-patterns.md)

**"I'm working with..."**

- One-to-one relationship → Denormalize (embed as attribute)
- One-to-many relationship → See [Guide 5](guides/5-relationship-strategies.md)
- Many-to-many relationship → See [Guide 5](guides/5-relationship-strategies.md)
- Time-series data → Composite sort key with timestamp
- Unique constraints → DynamoDB Transactions + tracking item
- Sequential IDs → Counter pattern (see [Guide 3](guides/3-primary-key-design.md))

### Common Questions

**Q: How do I know if I should use a simple or composite primary key?**

A: Use composite (PK + SK) if you need to:

- Fetch multiple related items in one request
- Model one-to-many or many-to-many relationships
- Sort items by a specific attribute

Use simple (PK only) if:

- All queries fetch single items by ID
- No relationships between entities
- Very simple access patterns

**Q: How many GSIs do I need?**

A: Start with 0. Add GSIs only when:

- Primary key can't satisfy an access pattern
- You need to query by different attributes
- You need reverse lookups

Then overload your GSIs—don't create one per pattern.

**Q: Can I change my data model later?**

A: Yes! Migrations are manageable. See [Guide 8](guides/8-migration-strategies.md) for strategies. Many changes are additive and don't require ETL.

**Q: Should I use single-table or multi-table design?**

A: Single-table design is recommended for most applications. Exceptions:

- Very new applications prioritizing flexibility
- GraphQL applications
- ETL/analytics-heavy workloads

See DynamoDB Book Chapter 8 for detailed discussion.

**Q: How do I model [specific relationship]?**

A: See [Guide 5: Relationship Strategies](guides/5-relationship-strategies.md) for comprehensive patterns.

---

## Validation Checklists

### Before Starting Implementation

- [ ] All access patterns documented with specific parameters
- [ ] ERD created and reviewed
- [ ] Entity charts created for main table and GSIs
- [ ] Relationship strategies decided and documented
- [ ] Anti-pattern review completed
- [ ] Peer review obtained
- [ ] Team consensus on design

### After Implementation

- [ ] All access patterns tested
- [ ] Performance testing completed
- [ ] Debugging scripts written
- [ ] Documentation updated
- [ ] Monitoring and alerting configured
- [ ] Backup and disaster recovery planned

---

## Success Metrics

A well-designed DynamoDB table should achieve:

✅ **Performance:**

- Sub-10ms response times at scale
- All queries use GetItem or Query (not Scan)
- Minimal filter expressions

✅ **Efficiency:**

- Fetch all data for access pattern in single request
- Proper use of item collections
- Overloaded GSIs

✅ **Maintainability:**

- Clear documentation of all access patterns
- Entity charts for all indexes
- Type attribute on all items
- Separation of indexing and application attributes

✅ **Cost:**

- Optimal read/write capacity usage
- Minimal wasted throughput
- Efficient projection strategies on GSIs

---

## Contributing

This guide is a living document. To contribute:

1. Propose changes via PR
2. Include DynamoDB Book citations where applicable
3. Add examples for clarity
4. Update quick reference cards if adding new patterns
5. Ensure consistency with existing guides

---

## Version History

- **v1.0** (2024) - Initial comprehensive guide
  - Phase-based workflow
  - 8 detailed strategy guides
  - Quick reference cards
  - Migration strategies
  - Example implementations

---

## Golden Rules

Remember these principles as you model:

1. **Know your access patterns first**
2. **Pre-join data in item collections**
3. **Use generic key names (PK, SK)**
4. **Overload GSIs**
5. **Separate indexing from application attributes**
6. **Always include Type attribute**
7. **Don't use Scan in application code**
8. **Storage is cheap, compute is expensive**
9. **Plan migrations before making changes**
10. **Test on copy of production data**

---

## Need Help?

- Review [Quick Reference Cards](quick-reference-cards.md) for common scenarios
- Check [Anti-Patterns Guide](guides/7-common-anti-patterns.md) if something feels wrong
- Study [complete examples](../specs/) to see patterns in action
- Consult The DynamoDB Book for deep theoretical background

**Remember:** 90% of DynamoDB work happens in planning. Take time to model correctly upfront, and implementation will be straightforward.

---

## References

All citations in this guide reference **The DynamoDB Book** by Alex DeBrie. Chapter numbers are noted in brackets throughout the documentation.
