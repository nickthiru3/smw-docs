# DynamoDB Data Modeling Guide

## Overview

This guide helps you choose and implement the right DynamoDB modeling approach for your application. We provide two comprehensive guides—**Faux-SQL** and **Single-Table Design**—each tailored to specific use cases and team needs.

**Key Principle:** "You can't design your table until you know how you'll use your data"

---

## 🎯 Choose Your Approach

### Quick Decision Tree

```
Are you building a new application or MVP?
│
├─ YES → Is development velocity your top priority?
│   │
│   ├─ YES → Use Faux-SQL Approach ✅
│   │         (Fast development, familiar patterns)
│   │
│   └─ NO → Do you need sub-10ms response times?
│       │
│       ├─ YES → Use Single-Table Design ✅
│       │         (Maximum performance)
│       │
│       └─ NO → Use Faux-SQL Approach ✅
│                 (Simpler, easier to change)
│
└─ NO (Existing app) → Are you experiencing performance issues?
    │
    ├─ YES → Migrate to Single-Table Design ✅
    │         (See migration guides in both approaches)
    │
    └─ NO → Keep current approach ✅
              (Don't optimize prematurely)
```

---

## 📚 Complete Guides

### [Faux-SQL DynamoDB Modeling](faux-sql-dynamodb-modeling.md)

**Best for:** New applications, MVPs, teams learning DynamoDB

**Approach:** Multiple tables with descriptive key names (like SQL)

**Key Features:**

- ✅ Fast development velocity
- ✅ Familiar SQL-like patterns
- ✅ Easy to change and evolve
- ✅ Simple to understand and debug
- ✅ Great for analytics and reporting
- ⚠️ Accepts 50-100ms response times
- ⚠️ Multiple requests for related data

**When to use:**

- Building MVP or early-stage product
- Access patterns still evolving
- Team prefers relational modeling
- Analytics requirements are high
- Using GraphQL
- Traffic < 10K requests/sec

**Guide includes:** Complete process from ERD to implementation, relationship patterns, migration strategies

---

### [Single-Table Design](single-table-design.md)

**Best for:** High-scale applications, performance-critical systems

**Approach:** One table with generic keys (PK/SK) and overloaded GSIs

**Key Features:**

- ⚡ Maximum performance (sub-10ms)
- 💰 Cost-efficient at scale
- 🔄 Atomic transactions
- 📈 Handles millions of items
- ⚠️ Steeper learning curve
- ⚠️ Requires upfront planning
- ⚠️ Harder to change

**When to use:**

- Need sub-10ms response times
- High throughput (>10K requests/sec)
- Complex relationships requiring pre-joining
- Cost optimization critical
- Access patterns are stable

**Guide includes:** 3 phases (11 steps), advanced topics, migration strategies, complete examples

---

## 📊 Approach Comparison

| Factor                | Faux-SQL                          | Single-Table                  |
| --------------------- | --------------------------------- | ----------------------------- |
| **Tables**            | Multiple (one per entity)         | Single table for all entities |
| **Key Names**         | Descriptive (CustomerId, OrderId) | Generic (PK, SK, GSI1PK)      |
| **Learning Curve**    | Easy (familiar SQL patterns)      | Steep (DynamoDB-specific)     |
| **Development Speed** | Fast (iterate quickly)            | Slower (requires planning)    |
| **Performance**       | 50-100ms typical                  | Sub-10ms possible             |
| **Flexibility**       | High (easy to change)             | Low (migrations complex)      |
| **Analytics**         | Easy (normalized data)            | Hard (denormalized)           |
| **Cost at Scale**     | Higher (more requests)            | Lower (fewer requests)        |
| **Best For**          | MVPs, evolving apps               | Production, high-scale        |

---

## 🚀 Migration Path

**Recommended Strategy:** Start with Faux-SQL, migrate to Single-Table when needed

```
Phase 1: Faux-SQL (Months 1-6)
├─ Fast development
├─ Learn access patterns
└─ Validate product-market fit

Phase 2: Evaluate (Month 6)
├─ Measure performance
├─ Assess scale
└─ Decide if migration needed

Phase 3: Migrate (If needed)
├─ Design single-table schema
├─ Dual-write strategy
├─ Backfill historical data
├─ Switch reads gradually
└─ Deprecate old tables
```

Both guides include detailed migration strategies.

## 🎓 Getting Started

### For New Projects

1. **Choose your approach** using the decision tree above
2. **Read the complete guide** for your chosen approach
3. **Follow the process** outlined in the guide
4. **Implement and test** your data model

### For Existing Projects

1. **Assess current performance** and scale
2. **Identify pain points** (slow queries, high costs, etc.)
3. **Consider migration** if needed (see migration sections in guides)
4. **Plan carefully** before making changes

---

## 📖 Learning Resources

### Primary Resources

- **[Faux-SQL Guide](faux-sql-dynamodb-modeling.md)** - Complete guide for Faux-SQL approach
- **[Single-Table Guide](single-table-design.md)** - Complete guide for single-table design

### External Resources

- _The DynamoDB Book_ by Alex DeBrie - Essential reading (primary reference for these guides, particularly the single-table design approach)
- [AWS DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/) - Official documentation
- [NoSQL Workbench](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.html) - Visual modeling tool
- [Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html) - AWS recommendations

---

## ❓ Common Questions

**Q: Which approach should I use?**

A: Use the decision tree at the top of this guide. Generally: Faux-SQL for new/evolving apps, Single-Table for high-scale/performance-critical apps.

**Q: Can I switch approaches later?**

A: Yes! Both guides include migration strategies. It's common to start with Faux-SQL and migrate to Single-Table when scale demands it.

**Q: Do I need to know all my access patterns upfront?**

A: **Faux-SQL**: No, you can add patterns incrementally by adding GSIs.
**Single-Table**: Yes, you should know most patterns upfront for optimal design.

**Q: What if I'm not sure which approach to use?**

A: When in doubt, start with **Faux-SQL**. It's easier to learn, faster to develop, and you can always migrate to Single-Table later if needed.

**Q: Are there any tools to help?**

A: Yes! AWS NoSQL Workbench is great for visualizing and testing your data model. Both guides include CDK/CloudFormation templates.

---

## 🎯 Key Takeaways

1. **There's no one-size-fits-all** - Choose the approach that fits your needs
2. **Start simple** - Faux-SQL is great for MVPs and learning
3. **Optimize when needed** - Migrate to Single-Table when scale demands it
4. **Plan before coding** - 90% of DynamoDB work happens in design
5. **Both approaches are valid** - Endorsed by The DynamoDB Book for different use cases

---

## 📞 Need Help?

- **Choosing an approach?** Use the decision tree at the top
- **Learning DynamoDB?** Start with the [Faux-SQL Guide](faux-sql-dynamodb-modeling.md)
- **Optimizing for scale?** Read the [Single-Table Guide](single-table-design.md)
- **Specific question?** Check the [Quick Reference Cards](quick-reference-cards.md)

---

**Remember:** The best approach is the one that helps your team ship quality software efficiently. Don't over-optimize prematurely!
