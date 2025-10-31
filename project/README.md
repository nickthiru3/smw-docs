# SMW Project Documentation

**Project**: SaveMyWaste (SMW) App  
**Purpose**: Connect consumers with circular-economy service providers and knowledge about the circular economy  
**Last Updated**: 2024-10-28

---

## Overview

SMW is a platform that helps Malaysian consumers discover and engage with sustainable businesses offering repair, refill, recycling, and donation services. This directory contains all project-level documentation including specifications, data models, research, and delivery artifacts.

---

## Directory Structure

```
docs/project/
├── README.md                    # This file
│
├── specs/                       # Technical specifications
│   ├── erd.puml                 # Entity-Relationship Diagram (all entities)
│   ├── entity-key-table.md      # Master entity-key reference
│   ├── actors.md                # User roles and personas
│   │
│   ├── entities/                # Detailed entity documentation
│   │   ├── README.md            # Guide for entity files
│   │   ├── merchants.md         # Merchant entity (table + GSIs + access patterns)
│   │   └── reviews.md           # Review entity (table + GSIs + access patterns)
│   │
│   └── jobs/                    # Job stories (feature specifications)
│       └── consumers/
│           └── search-by-category/
│               ├── job-card.md
│               ├── mockups/
│               └── sequence-diagram.puml
│
├── research/                    # Research and discovery artifacts
│   └── circular-economy-merchant-attributes.md
│
└── status-board.md              # Project status and progress tracking
```

---

## Quick Navigation

### Data Modeling

- **ERD**: [`specs/erd.puml`](specs/erd.puml) - Visual entity relationships
- **Entity-Key Table**: [`specs/entity-key-table.md`](specs/entity-key-table.md) - Quick reference for all entities
- **Entities Directory**: [`specs/entities/`](specs/entities/) - Detailed entity documentation

### Job Stories

- **All Jobs**: [`specs/jobs/`](specs/jobs/) - Feature specifications by actor
- **Search by Category**: [`specs/jobs/consumers/search-by-category/`](specs/jobs/consumers/search-by-category/)

### Research

- **Merchant Attributes**: [`research/circular-economy-merchant-attributes.md`](research/circular-economy-merchant-attributes.md)

### Project Management

- **Status Board**: [`status-board.md`](status-board.md) - Current sprint status
- **Actors**: [`specs/actors.md`](specs/actors.md) - User personas

---

## Key Concepts

### SEED(S) Methodology

SMW follows the SEED(S) product discovery and delivery methodology:

- **S**tory - Job stories with acceptance criteria
- **E**xplore - Extended discovery (mockups, sequence diagrams)
- **E**laborate - Technical design (ERD, entity schemas, access patterns)
- **D**eliver - Implementation and deployment
- **(S)** - Sustain - Monitoring and iteration

### Data Modeling Approach

**Faux-SQL DynamoDB Design**:

- Multiple DynamoDB tables (one per entity type)
- Descriptive key names (MerchantId, ReviewId, not PK/SK)
- Normalized data with foreign keys
- Simple GSIs for alternate lookups
- Entity-centric documentation

See: [`../guides/data-modeling/guides/faux-sql-dynamodb-modeling.md`](../guides/data-modeling/guides/faux-sql-dynamodb-modeling.md)

### Architecture

**Microservices**:

- `merchants-search-service` - Merchant discovery and search
- `web` - SvelteKit 5 frontend application

**Infrastructure**:

- AWS CDK for infrastructure as code
- DynamoDB for data persistence
- Lambda for serverless compute

---

## Workflow: From Job to Implementation

### Phase 1: Job Story Definition

1. **Create job card**: `specs/jobs/<actor>/<job-name>/job-card.md`
2. **Define acceptance criteria**: What must be true for job to be complete
3. **Identify actors**: Who is this job for?

### Phase 2: Extended Discovery

1. **Create mockups**: UX Pilot generates UI designs
2. **Document mockups**: `specs/jobs/<job>/mockups/README.md`
3. **Create sequence diagram**: API interactions

### Phase 3: Data Modeling

1. **Update ERD**: Add new entities to `specs/erd.puml`
2. **Create/update entity files**: `specs/entities/<entity>.md`
3. **Update master table**: `specs/entity-key-table.md`
4. **Define access patterns**: Document in entity files

### Phase 4: Implementation

1. **Create entity schemas**: TypeScript types with DynamoDB Toolbox
2. **Implement repositories**: Data access layer
3. **Implement services**: Business logic
4. **Create API endpoints**: OpenAPI contract
5. **Build UI**: SvelteKit pages and components

### Phase 5: Deployment

1. **Deploy infrastructure**: AWS CDK
2. **Deploy backend**: Lambda functions
3. **Deploy frontend**: Netlify/Vercel
4. **Update status board**: Mark job as complete

---

## Current Status

### Completed Jobs

- ✅ **Search by Category** (Phase 1a) - Consumer search for merchants by category

### In Progress

- 🚧 Data modeling refinements
- 🚧 Entity schema implementation

### Planned

- 📋 User Authentication (Phase 2)
- 📋 Add to Favorites (Phase 2)
- 📋 Merchant Portal (Phase 2)

See [`status-board.md`](status-board.md) for detailed status.

---

## Documentation Standards

### Entity Documentation

Each entity has a comprehensive file in `specs/entities/`:

- Main table entity-key structure
- GSI definitions with rationale
- Access patterns with query examples
- Design decisions log
- Evolution history

**Example**: [`specs/entities/merchants.md`](specs/entities/merchants.md)

### Job Documentation

Each job has a folder in `specs/jobs/<actor>/<job-name>/`:

- `job-card.md` - Job story, acceptance criteria, requirements
- `mockups/` - UI designs and design decisions
- `sequence-diagram.puml` - API interaction flows

**Example**: [`specs/jobs/consumers/search-by-category/`](specs/jobs/consumers/search-by-category/)

### Diagram Standards

- **ERD**: PlantUML, no attributes (only entities and relationships)
- **Sequence Diagrams**: PlantUML, show API calls and data flow
- **Architecture Diagrams**: PlantUML or Mermaid

---

## Related Documentation

### Guides

- **Data Modeling**: [`../guides/data-modeling/`](../guides/data-modeling/)
- **Faux-SQL Guide**: [`../guides/data-modeling/guides/faux-sql-dynamodb-modeling.md`](../guides/data-modeling/guides/faux-sql-dynamodb-modeling.md)

### Code Repositories

- **Web**: `/home/nickt/projects/smw/web`
- **Merchants Search Service**: `/home/nickt/projects/smw/merchants-search-service`

---

## Contributing

### Adding a New Job

1. Create job folder: `specs/jobs/<actor>/<job-name>/`
2. Create job card with acceptance criteria
3. Generate mockups with UX Pilot
4. Update ERD if new entities introduced
5. Create/update entity files
6. Create sequence diagram
7. Update status board

### Adding a New Entity

1. Update `specs/erd.puml` with new entity and relationships
2. Create `specs/entities/<entity-name>.md` with full documentation
3. Update `specs/entity-key-table.md` master reference
4. Document which job introduced the entity

### Modifying an Entity

1. Update entity file: `specs/entities/<entity-name>.md`
2. Add entry to "Evolution History" section
3. Update "Design Decisions Log" with rationale
4. Update master `entity-key-table.md` if keys changed
5. Update ERD if relationships changed

---

## Questions?

- **Data Modeling**: See [`../guides/data-modeling/README.md`](../guides/data-modeling/README.md)
- **Entity Files**: See [`specs/entities/README.md`](specs/entities/README.md)
- **SEED(S) Methodology**: See job card templates in `specs/jobs/`

---

**Maintainers**: SMW Development Team  
**Last Review**: 2024-10-28
