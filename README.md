# SaveMyWaste (SMW) Documentation

Welcome to the SaveMyWaste project documentation! This is the entry point for all team members.

**Project**: SaveMyWaste (SMW) App  
**Purpose**: Connect consumers with circular-economy service providers and knowledge about the circular economy  
**Target Market**: Malaysia

---

## 🎯 Start Here

### New Team Members

1. **[Project Documentation](./project/README.md)** - Start here to understand the project

   - Technical specifications
   - Entity-Relationship Diagrams
   - User stories and features
   - Data models and access patterns

2. **[Documentation Map](./DOCUMENTATION-MAP.md)** - Visual guide to all documentation across the workspace

### Developers

- **Backend**: [Merchants Microservice Docs](../svc-merchants/docs/README.md)
- **Frontend**: [Web App Docs](../web/docs/README.md) (if exists)

---

## 📚 Documentation Structure

```
smw/
├── docs/                                    # ← You are here
│   ├── README.md                            # This file (workspace entry point)
│   ├── DOCUMENTATION-MAP.md                 # Visual map of all docs
│   │
│   ├── project/                             # Project-level documentation
│   │   ├── README.md                        # Project overview
│   │   ├── specs/                           # Technical specifications
│   │   │   ├── erd.puml                     # Entity-Relationship Diagram
│   │   │   ├── entities/                    # Entity documentation
│   │   │   ├── stories/                     # User stories
│   │   │   └── api/                         # API specifications
│   │   └── research/                        # Research and analysis
│   │
│   └── guides/                              # Cross-cutting guides (if any)
│
├── svc-merchants/                           # Merchants microservice
│   ├── docs/
│   │   ├── README.md                        # Microservice docs entry
│   │   ├── DOCUMENTATION-MAP.md             # Microservice doc map
│   │   ├── implementation/                  # Implementation guides
│   │   └── testing/                         # Testing guides
│   └── (source code)
│
├── web/                                     # Frontend web app
│   ├── docs/                                # Frontend docs
│   └── (source code)
│
└── (other services...)
```

---

## 🗺️ Quick Navigation

### By Role

| Role                   | Start Here                                                                         |
| ---------------------- | ---------------------------------------------------------------------------------- |
| **New Team Member**    | [Project README](./project/README.md)                                              |
| **Product Manager**    | [User Stories](./project/specs/stories/)                                           |
| **Backend Developer**  | [Merchants Service Docs](../svc-merchants/docs/README.md)                          |
| **Frontend Developer** | [Web App Docs](../web/docs/README.md)                                              |
| **Data Modeler**       | [Entity Documentation](./project/specs/entities/)                                  |
| **DevOps Engineer**    | [Merchants Service Deployment](../svc-merchants/docs/implementation/deployment.md) |

### By Task

| Task                          | Documentation                                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| **Understand the project**    | [Project README](./project/README.md)                                              |
| **View data model**           | [ERD](./project/specs/erd.puml) & [Entities](./project/specs/entities/)            |
| **Read user stories**         | [Stories](./project/specs/stories/)                                                |
| **Implement backend feature** | [Merchants Service Implementation](../svc-merchants/docs/implementation/README.md) |
| **Build frontend feature**    | [Web App Docs](../web/docs/README.md)                                              |
| **Deploy services**           | [Deployment Guides](../svc-merchants/docs/implementation/deployment.md)            |

---

## 📖 Documentation Areas

### 1. Project Documentation

**Location**: `docs/project/`

**Purpose**: Project-level specifications, entities, and user stories

**Contains**:

- Entity-Relationship Diagrams (ERD)
- Entity specifications (Merchants, Reviews, etc.)
- User stories and feature specs
- API specifications (OpenAPI)
- Research and analysis

**Start**: [Project README](./project/README.md)

---

### 2. Microservice Documentation

#### Merchants Microservice

**Location**: `svc-merchants/docs/`

**Purpose**: Backend implementation guides for the Merchants microservice

**Contains**:

- Development workflow
- Configuration management
- Endpoint development (Lambda + API Gateway)
- Database setup (DynamoDB)
- Testing guides
- Deployment guides

**Start**: [Merchants Service README](../svc-merchants/docs/README.md)

---

### 3. Web App Documentation

**Location**: `web/docs/`

**Purpose**: Frontend implementation guides for the web application

**Contains**:

- SvelteKit setup
- Component development
- State management
- API integration
- Testing guides

**Start**: [Web App README](../web/docs/README.md) (if exists)

---

## 🚀 Getting Started Workflows

### Workflow 1: Understanding the Project

**For**: New team members, product managers, stakeholders

```
1. Read Project README
   ↓
2. Review Entity-Relationship Diagram (ERD)
   ↓
3. Explore Entity Specifications
   ↓
4. Read User Stories
   ↓
5. Review API Specifications
```

**Start**: [Project README](./project/README.md)

---

### Workflow 2: Implementing a Backend Feature

**For**: Backend developers

```
1. Read User Story
   ↓
2. Review Entity Specification
   ↓
3. Check API Specification
   ↓
4. Go to Merchants Service Docs
   ↓
5. Follow Implementation Workflow
```

**Start**: [User Stories](./project/specs/stories/) → [Merchants Service](../svc-merchants/docs/README.md)

---

### Workflow 3: Implementing a Frontend Feature

**For**: Frontend developers

```
1. Read User Story
   ↓
2. Review API Specification
   ↓
3. Check Design Mockups (if available)
   ↓
4. Go to Web App Docs
   ↓
5. Follow Implementation Workflow
```

**Start**: [User Stories](./project/specs/stories/) → [Web App Docs](../web/docs/README.md)

---

## 🔍 Finding Documentation

### Can't Find What You're Looking For?

1. **Check the [Documentation Map](./DOCUMENTATION-MAP.md)** - Visual guide to all docs
2. **Search by role** - See "Quick Navigation" above
3. **Search by task** - See "Quick Navigation" above
4. **Browse by area** - See "Documentation Areas" above

### Documentation Not Clear?

- Open an issue in the relevant repository
- Ask in team chat
- Submit a PR to improve documentation

---

## 📊 Documentation Overview

### Total Documentation Areas

- **Project-level**: 1 area (specs, entities, stories, research)
- **Microservices**: 1+ services (svc-merchants, ...)
- **Frontend**: 1+ apps (web, ...)

### Documentation Principles

1. **Hierarchical**: Start broad (project) → narrow (service)
2. **Role-based**: Different entry points for different roles
3. **Task-oriented**: Find docs by what you need to do
4. **Cross-referenced**: Docs link to related docs
5. **Visual**: Diagrams and maps for navigation

---

## 🤝 Contributing to Documentation

### When to Update

- **New feature**: Update user stories and implementation guides
- **New entity**: Update ERD and entity specs
- **New service**: Add service documentation
- **Process change**: Update workflow guides

### How to Update

1. Update the relevant documentation
2. Update cross-references
3. Update documentation map if structure changed
4. Test all links
5. Submit PR for review

---

## 📞 Need Help?

- **Project questions**: Check [Project README](./project/README.md)
- **Backend questions**: Check [Merchants Service Docs](../svc-merchants/docs/README.md)
- **Frontend questions**: Check [Web App Docs](../web/docs/README.md)
- **Can't find docs**: Check [Documentation Map](./DOCUMENTATION-MAP.md)

---

**Welcome to the SaveMyWaste team!** 🌱♻️
