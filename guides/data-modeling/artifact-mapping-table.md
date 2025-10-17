You're absolutely right to question this! The artifact mapping table should be **entirely in Markdown format**. Let me provide the corrected version:

---

# Data Modeling Artifact Mapping Table

This table maps each artifact type to the relevant phases and guides in the data modeling workflow.

## Artifact Categories

### Phase 1: Experience & Domain Design

| Artifact                              | Description                                               | Created By                                | References      |
| ------------------------------------- | --------------------------------------------------------- | ----------------------------------------- | --------------- |
| **ERD (Entity-Relationship Diagram)** | Visual representation of entities and their relationships | Guide 1: ERD Creation Guide               | Phase 1, Step 3 |
| **Access Patterns Table**             | Comprehensive list of all data access requirements        | Guide 2: Access Patterns Definition Guide | Phase 1, Step 4 |
| **Entity Chart (Draft)**              | Initial entity types identified from ERD                  | Guide 1: ERD Creation Guide               | Phase 1, Step 3 |

### Phase 2: Data Modeling

| Artifact                            | Description                                          | Created By                                       | References        |
| ----------------------------------- | ---------------------------------------------------- | ------------------------------------------------ | ----------------- |
| **Primary Key Design**              | PK/SK patterns for each entity in main table         | Guide 3: Primary Key Design Guide                | Phase 2, Step 1   |
| **Entity Chart (Main Table)**       | Complete entity chart with PK/SK patterns            | Guide 3: Primary Key Design Guide                | Phase 2, Step 1   |
| **Entity Schema (TypeScript)**      | TypeScript interfaces for all application attributes | Guide 4: Entity Schema Guide                     | Phase 2, Step 2   |
| **GSI Entity Charts**               | Entity charts for each Global Secondary Index        | Guide 6: Secondary Index Strategies              | Phase 2, Step 3   |
| **Access Patterns Table (Updated)** | Updated with Index and Query details                 | Guide 6: Secondary Index Strategies              | Phase 2, Step 3   |
| **Relationship Documentation**      | Strategy decisions for each relationship             | Guide 5: Relationship Strategies Quick Reference | Phase 2, Step 1-2 |

### Phase 3: Implementation

| Artifact                 | Description                             | Created By                          | References |
| ------------------------ | --------------------------------------- | ----------------------------------- | ---------- |
| **CDK Table Definition** | AWS CDK code defining DynamoDB table    | Implementation                      | Phase 3    |
| **Data Access Layer**    | TypeScript code for DynamoDB operations | Implementation                      | Phase 3    |
| **Migration Scripts**    | ETL scripts for data model changes      | Guide 8: Migration Strategies (TBD) | Phase 3    |

## Artifact Locations

All artifacts for a specific job should be organized as follows:

```
docs/specs/jobs/<job-name>/
├── erd.puml                        # ERD diagram (PlantUML)
├── data-access-patterns.md         # Access patterns table
├── entity-chart-main.md            # Main table entity chart
├── entity-chart-gsi1.md            # GSI1 entity chart
├── entity-chart-gsi2.md            # GSI2 entity chart (if needed)
├── entity-schemas.ts               # TypeScript entity interfaces
├── relationship-decisions.md       # Relationship strategy documentation
└── migration-plan.md               # Migration strategy (if applicable)
```

## Guide Reference Map

| Guide                                   | Primary Artifacts Created                          | Phase      |
| --------------------------------------- | -------------------------------------------------- | ---------- |
| **Guide 1: ERD Creation**               | ERD (erd.puml), Draft Entity Chart                 | Phase 1    |
| **Guide 2: Access Patterns Definition** | Access Patterns Table (data-access-patterns.md)    | Phase 1    |
| **Guide 3: Primary Key Design**         | Entity Chart Main Table (entity-chart-main.md)     | Phase 2    |
| **Guide 4: Entity Schema**              | Entity Schemas (entity-schemas.ts)                 | Phase 2    |
| **Guide 5: Relationship Strategies**    | Relationship Decisions (relationship-decisions.md) | Phase 2    |
| **Guide 6: Secondary Index Strategies** | GSI Entity Charts (entity-chart-gsiX.md)           | Phase 2    |
| **Guide 7: Anti-Patterns**              | Review checklist (used throughout)                 | All Phases |
| **Guide 8: Migration Strategies**       | Migration Plan (migration-plan.md)                 | Phase 3    |

## Workflow Integration

### Phase 1 Flow

```
ERD Creation (Guide 1)
    → Entity Chart (Draft)
    → Access Patterns Definition (Guide 2)
    → Access Patterns Table
```

### Phase 2 Flow

```
Primary Key Design (Guide 3)
    + Relationship Strategies (Guide 5)
    → Entity Chart (Main)

Entity Schema (Guide 4)
    → Entity Schemas (TypeScript)

Secondary Index Strategies (Guide 6)
    → GSI Entity Charts
    → Updated Access Patterns Table

Anti-Patterns Review (Guide 7)
    → Validation of all above artifacts
```

### Phase 3 Flow

```
Implementation
    → CDK Table Definition
    → Data Access Layer

Migration Strategies (Guide 8)
    → Migration Scripts (if needed)
```

## Validation Checkpoints

After creating each artifact, validate against:

1. **ERD** - All entities represented, relationships clear
2. **Access Patterns** - Complete, specific, with parameters
3. **Primary Keys** - Satisfy uniqueness, enable access patterns
4. **Entity Schemas** - All application attributes documented
5. **GSI Designs** - Enable unsatisfied patterns, properly overloaded
6. **Anti-Patterns** - None present in design

---

## Quick Reference: Which Guide When?

| Situation                                 | Use This Guide                    |
| ----------------------------------------- | --------------------------------- |
| Starting a new data model from scratch    | Start with Guide 1 (ERD)          |
| Have entities, need to define queries     | Guide 2 (Access Patterns)         |
| Have access patterns, need to design keys | Guide 3 (Primary Key Design)      |
| Have keys, need to document attributes    | Guide 4 (Entity Schema)           |
| Modeling relationships between entities   | Guide 5 (Relationship Strategies) |
| Primary key doesn't satisfy all patterns  | Guide 6 (Secondary Indexes)       |
| Reviewing existing design                 | Guide 7 (Anti-Patterns)           |
| Changing existing data model              | Guide 8 (Migrations)              |
