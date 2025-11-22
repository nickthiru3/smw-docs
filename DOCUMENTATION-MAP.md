# SMW Workspace Documentation Map

Visual map of all documentation across the entire SaveMyWaste workspace.

---

## How to Read This Map

- **Boxes**: Documentation files or directories
- **Arrows**: Navigation flow (where docs reference each other)
- **Colors**:
  - 🟦 Blue: Entry points
  - 🟩 Green: Project-level docs
  - 🟨 Yellow: Microservice docs
  - 🟧 Orange: Frontend docs

---

## Workspace Documentation Hierarchy

```plantuml
@startuml SMW Workspace Documentation Map

!define ENTRY_POINT #LightBlue
!define PROJECT_DOCS #LightGreen
!define MICROSERVICE_DOCS #LightYellow
!define FRONTEND_DOCS #LightCoral

title SaveMyWaste - Workspace Documentation Map

' ============================================
' WORKSPACE ENTRY POINT
' ============================================

package "📍 Workspace Entry" ENTRY_POINT {
  [SMW Docs README\ndocs/README.md] as WorkspaceREADME
  [Workspace Doc Map\ndocs/DOCUMENTATION-MAP.md] as WorkspaceDocMap
}

' ============================================
' PROJECT-LEVEL DOCUMENTATION
' ============================================

package "📋 Project Documentation" PROJECT_DOCS {
  [Project README\nproject/README.md] as ProjectREADME

  package "Specifications" {
    [ERD\nspecs/erd.puml] as ERD
    [Entity Key Table\nspecs/entity-key-table.md] as EntityKeyTable
    [Actors\nspecs/actors.md] as Actors
  }

  package "Entities" {
    [Entities README\nspecs/entities/README.md] as EntitiesREADME
    [Merchants Entity\nspecs/entities/merchants.md] as MerchantsEntity
    [Reviews Entity\nspecs/entities/reviews.md] as ReviewsEntity
    [... other entities] as OtherEntities
  }

  package "User Stories" {
    [Stories\nspecs/stories/] as Stories
    [Consumer Stories\nspecs/stories/consumers/] as ConsumerStories
    [Merchant Stories\nspecs/stories/merchants/] as MerchantStories
  }

  package "API Specifications" {
    [API Specs\nspecs/api/] as APISpecs
    [OpenAPI Specs] as OpenAPISpecs
  }

  package "Research" {
    [Research\nresearch/] as Research
  }
}

' ============================================
' MERCHANTS MICROSERVICE DOCUMENTATION
' ============================================

package "🔧 Merchants Microservice" MICROSERVICE_DOCS {
  [Merchants Docs README\nsvc-merchants/docs/README.md] as MerchantsREADME
  [Merchants Doc Map\nsvc-merchants/docs/DOCUMENTATION-MAP.md] as MerchantsDocMap

  package "Implementation Guides" {
    [Implementation README\nimplementation/README.md] as ImplREADME
    [Dev Guide v2\nmicroservice-development-guide-v2.md] as DevGuideV2

    package "Configuration" {
      [Config README\nconfiguration-management/README.md] as ConfigREADME
      [9 Domain Configs] as DomainConfigs
    }

    package "Endpoints" {
      [Part 1: Lambda Handlers\nadding-endpoints-part-1.md] as EndpointsPart1
      [Part 2: API Gateway\nadding-endpoints-part-2.md] as EndpointsPart2
    }

    package "Database" {
      [Database Setup\ndatabase-setup.md] as DatabaseSetup
      [Data Access Layer\ndata-access.md] as DataAccess
    }

    package "Other" {
      [Deployment\ndeployment.md] as Deployment
      [Monitoring\nmonitoring.md] as Monitoring
      [Authentication\nauthentication.md] as Authentication
      [Authorization\nauthorization.md] as Authorization
    }
  }

  package "Testing Guides" {
    [Handler Testing\ntesting/handler-testing-guide.md] as HandlerTesting
    [CDK Testing\ntesting/cdk-template-testing-guide.md] as CDKTesting
    [Schema Testing\ntesting/schema-testing-guide.md] as SchemaTesting
    [Helpers Testing\ntesting/unit-helpers-testing-guide.md] as HelpersTesting
    [E2E Testing\ntesting/e2e-testing-guide.md] as E2ETesting
  }
}

' ============================================
' WEB APP DOCUMENTATION
' ============================================

package "🌐 Web Application" FRONTEND_DOCS {
  [Web Docs README\nweb/docs/README.md] as WebREADME
  [Web Implementation\nweb/docs/implementation/] as WebImplementation
  [Web Testing\nweb/docs/testing/] as WebTesting
}

' ============================================
' NAVIGATION FLOW - WORKSPACE LEVEL
' ============================================

WorkspaceREADME -down-> ProjectREADME : "1. Understand\nProject"
WorkspaceREADME -down-> MerchantsREADME : "2. Backend\nDevelopment"
WorkspaceREADME -down-> WebREADME : "3. Frontend\nDevelopment"
WorkspaceREADME -right-> WorkspaceDocMap : "Visual\nMap"

' ============================================
' PROJECT DOCUMENTATION FLOW
' ============================================

ProjectREADME -down-> ERD : "View\nData Model"
ProjectREADME -down-> EntitiesREADME : "Entity\nSpecs"
ProjectREADME -down-> Stories : "User\nStories"
ProjectREADME -down-> APISpecs : "API\nContracts"

ERD -right-> EntityKeyTable : "Key\nReference"
ERD -down-> EntitiesREADME

EntitiesREADME -down-> MerchantsEntity
EntitiesREADME -down-> ReviewsEntity
EntitiesREADME -down-> OtherEntities

Stories -down-> ConsumerStories
Stories -down-> MerchantStories

' ============================================
' PROJECT TO MICROSERVICE FLOW
' ============================================

MerchantsEntity -down-> MerchantsREADME : "Implement\nBackend"
ConsumerStories -down-> MerchantsREADME : "Implement\nFeature"
APISpecs -down-> MerchantsREADME : "API\nImplementation"

' ============================================
' MICROSERVICE DOCUMENTATION FLOW
' ============================================

MerchantsREADME -down-> ImplREADME : "Main\nEntry"
MerchantsREADME -right-> MerchantsDocMap : "Visual\nMap"

ImplREADME -down-> DevGuideV2 : "Workflow"

DevGuideV2 -down-> ConfigREADME : "Step 1:\nConfigure"
DevGuideV2 -down-> DatabaseSetup : "Step 2:\nDatabase"
DevGuideV2 -down-> EndpointsPart1 : "Step 3:\nHandlers"
DevGuideV2 -down-> EndpointsPart2 : "Step 4:\nAPI Gateway"
DevGuideV2 -down-> Deployment : "Step 5:\nDeploy"
DevGuideV2 -down-> Monitoring : "Step 6:\nMonitor"

ConfigREADME -down-> DomainConfigs

EndpointsPart1 -right-> EndpointsPart2
EndpointsPart1 -down-> HandlerTesting
EndpointsPart2 -down-> CDKTesting

DatabaseSetup -right-> DataAccess

' ============================================
' PROJECT TO FRONTEND FLOW
' ============================================

ConsumerStories -down-> WebREADME : "Implement\nFrontend"
APISpecs -down-> WebREADME : "API\nIntegration"

' ============================================
' FRONTEND DOCUMENTATION FLOW
' ============================================

WebREADME -down-> WebImplementation
WebREADME -down-> WebTesting

' ============================================
' CROSS-REFERENCES
' ============================================

DatabaseSetup -up-> MerchantsEntity : "Entity\nReference"
EndpointsPart1 -up-> APISpecs : "API\nContract"
Authentication -right-> Authorization

' ============================================
' LEGEND
' ============================================

legend right
  |= Color |= Category |
  | <back:LightBlue>   </back> | Entry Points |
  | <back:LightGreen>   </back> | Project Docs |
  | <back:LightYellow>   </back> | Microservice Docs |
  | <back:LightCoral>   </back> | Frontend Docs |
endlegend

@enduml
```

---

## Documentation Layers

### Layer 1: Workspace Entry (Blue)

**Purpose**: Entry point for all team members

**Files**:

- `docs/README.md` - Workspace documentation entry
- `docs/DOCUMENTATION-MAP.md` - This file

**Who uses**: Everyone (new team members start here)

---

### Layer 2: Project Documentation (Green)

**Purpose**: Project-level specifications and design

**Location**: `docs/project/`

**Contains**:

- Entity-Relationship Diagrams (ERD)
- Entity specifications
- User stories
- API specifications
- Research and analysis

**Who uses**:

- Product managers
- Data modelers
- All developers (for context)

**Flow**:

```
Project README → ERD → Entities → Stories → API Specs
```

---

### Layer 3: Microservice Documentation (Yellow)

**Purpose**: Backend implementation guides

**Location**: `svc-merchants/docs/`

**Contains**:

- Development workflow (8 steps)
- Configuration management (10 guides)
- Endpoint development (2 parts)
- Database setup
- Testing guides (5 guides)
- Deployment and monitoring

**Who uses**: Backend developers, DevOps engineers

**Flow**:

```
Merchants README → Implementation README → Dev Guide v2 →
Configure → Database → Handlers → API Gateway → Deploy → Monitor
```

---

### Layer 4: Frontend Documentation (Orange)

**Purpose**: Frontend implementation guides

**Location**: `web/docs/`

**Contains**:

- SvelteKit setup
- Component development
- State management
- API integration
- Testing guides

**Who uses**: Frontend developers

**Flow**:

```
Web README → Implementation → Testing
```

---

## Common Navigation Paths

### Path 1: New Team Member Onboarding

```
Workspace README (docs/README.md)
  ↓
Project README (docs/project/README.md)
  ↓
ERD (docs/project/specs/erd.puml)
  ↓
Entity Specs (docs/project/specs/entities/)
  ↓
User Stories (docs/project/specs/stories/)
  ↓
Choose: Backend or Frontend
  ↓
Merchants README or Web README
```

**Time**: 2-4 hours

---

### Path 2: Backend Feature Implementation

```
User Story (docs/project/specs/stories/)
  ↓
Entity Spec (docs/project/specs/entities/)
  ↓
API Spec (docs/project/specs/api/)
  ↓
Merchants README (svc-merchants/docs/README.md)
  ↓
Dev Guide v2 (microservice-development-guide-v2.md)
  ↓
Follow 8-step workflow
```

**Time**: 1-2 hours (reading), 4-8 hours (implementation)

---

### Path 3: Frontend Feature Implementation

```
User Story (docs/project/specs/stories/)
  ↓
API Spec (docs/project/specs/api/)
  ↓
Design Mockups (if available)
  ↓
Web README (web/docs/README.md)
  ↓
Follow implementation workflow
```

**Time**: 1-2 hours (reading), 4-8 hours (implementation)

---

### Path 4: Data Modeling

```
Project README (docs/project/README.md)
  ↓
ERD (docs/project/specs/erd.puml)
  ↓
Entity Key Table (docs/project/specs/entity-key-table.md)
  ↓
Entity Specs (docs/project/specs/entities/)
  ↓
Database Setup (svc-merchants/docs/implementation/database-setup.md)
```

**Time**: 1-2 hours

---

## Quick Reference

### By Role

| Role                   | Entry Point                                                      | Next Steps                       |
| ---------------------- | ---------------------------------------------------------------- | -------------------------------- |
| **New Team Member**    | [Workspace README](../README.md)                                 | → Project README → ERD → Stories |
| **Product Manager**    | [Project README](./project/README.md)                            | → Stories → API Specs            |
| **Backend Developer**  | [Merchants README](../svc-merchants/docs/README.md)              | → Dev Guide v2 → Implementation  |
| **Frontend Developer** | [Web README](../web/docs/README.md)                              | → Implementation → Testing       |
| **Data Modeler**       | [ERD](./project/specs/erd.puml)                                  | → Entities → Database Setup      |
| **DevOps Engineer**    | [Deployment](../svc-merchants/docs/implementation/deployment.md) | → Monitoring                     |

### By Task

| Task                   | Start Here                                                                               | Related Docs               |
| ---------------------- | ---------------------------------------------------------------------------------------- | -------------------------- |
| **Understand project** | [Project README](./project/README.md)                                                    | ERD, Entities, Stories     |
| **View data model**    | [ERD](./project/specs/erd.puml)                                                          | Entity Key Table, Entities |
| **Read user stories**  | [Stories](./project/specs/stories/)                                                      | API Specs                  |
| **Implement backend**  | [Merchants README](../svc-merchants/docs/README.md)                                      | Dev Guide, Implementation  |
| **Implement frontend** | [Web README](../web/docs/README.md)                                                      | Implementation, Testing    |
| **Deploy services**    | [Deployment](../svc-merchants/docs/implementation/deployment.md)                         | Monitoring                 |
| **Configure service**  | [Config README](../svc-merchants/docs/implementation/configuration-management/README.md) | Domain Configs             |

---

## Documentation Statistics

### Total Documentation Files

- **Workspace-level**: 2 files
- **Project-level**: 10+ files (ERD, entities, stories, API specs, research)
- **Merchants microservice**: 29+ files (implementation + testing)
- **Web app**: TBD

**Total**: 40+ documentation files

### Documentation Coverage

- ✅ Workspace entry point
- ✅ Project specifications
- ✅ Entity documentation
- ✅ User stories
- ✅ API specifications
- ✅ Backend implementation guides
- ✅ Backend testing guides
- ⏳ Frontend documentation (TBD)

---

## Diagram Source

The PlantUML diagram source is embedded in this file. To regenerate:

1. Copy the PlantUML code block
2. Use PlantUML online editor or local tool
3. Export as PNG/SVG

**PlantUML Online Editor**: http://www.plantuml.com/plantuml/uml/

---

## Need Help?

- **Can't find documentation?** Check the [Quick Reference](#quick-reference) above
- **Not sure where to start?** Follow a [Common Navigation Path](#common-navigation-paths)
- **Documentation unclear?** Open an issue or submit a PR
- **New to the project?** Start with [Workspace README](../README.md)

---

**This map covers the entire SMW workspace documentation!** 🗺️
