# Linear Integration Guide

**Last Updated:** 2025-11-29  
**Status:** Active Development  
**Purpose:** Document how to integrate Linear project management with our development methodology

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Linear Concepts & Hierarchy](#2-linear-concepts--hierarchy)
3. [Mapping Our Methodology to Linear](#3-mapping-our-methodology-to-linear)
4. [Multi-Business Configuration](#4-multi-business-configuration)
5. [MCP Server Setup](#5-mcp-server-setup)
6. [Practical Implementation](#6-practical-implementation)
7. [Workflow Guidelines](#7-workflow-guidelines)
8. [Artifact Management Strategy](#8-artifact-management-strategy)

---

## 1. Introduction

### 1.1. Purpose

This guide documents how to integrate Linear (project management tool) with our existing design-and-development methodology. The goal is to:

- Use Linear for project tracking, visibility, and collaboration
- Maintain our detailed technical artifacts in the workspace (Windsurf/IDE)
- Enable AI assistants (Cascade) to manage Linear on our behalf via MCP

### 1.2. Background

Our development methodology (see `docs/guides/design-and-development/design-and-development-methodology-v3.md`) produces rich artifacts:

- **Story cards** with acceptance criteria, scenarios, and requirements
- **Sequence diagrams** (PlantUML)
- **Actions & Queries** documents
- **UI mockups**
- **API specifications** (OpenAPI)
- **Data models** (ERD, entity files)

Linear's model treats:

- **Projects** as feature implementations
- **Issues** as requirements/stories within projects

The challenge is reconciling these two systems effectively.

### 1.3. Key Insight: Complementary Systems

Linear and workspace artifacts serve **different purposes**:

| Aspect        | Linear                              | Workspace (Windsurf)                       |
| ------------- | ----------------------------------- | ------------------------------------------ |
| **Purpose**   | Tracking, visibility, collaboration | Technical design, implementation reference |
| **Audience**  | Team coordination, stakeholders     | Developers during implementation           |
| **Content**   | Status, assignments, discussions    | Detailed specs, diagrams, code             |
| **Lifecycle** | Active during development           | Permanent reference                        |

---

## 2. Linear Concepts & Hierarchy

### 2.1. Linear's Organizational Structure

```
Linear Workspace (Organization Level)
└── Teams (App/Product Level)
    └── Projects (Feature/Epic Level)
        └── Issues (Story/Task Level)
            └── Sub-issues (Sub-tasks)
```

### 2.2. Concept Definitions

| Linear Concept | Definition                       | Our Equivalent                 |
| -------------- | -------------------------------- | ------------------------------ |
| **Workspace**  | Top-level organization account   | Organization/Business entity   |
| **Team**       | A group working on a product/app | An application (e.g., SMW)     |
| **Project**    | A feature or initiative          | A Story Card (e.g., Story 001) |
| **Issue**      | A unit of work                   | Bundled stories / sub-tasks    |
| **Cycle**      | Time-boxed iteration             | Sprint                         |
| **Label**      | Categorization tag               | Phase, priority, type markers  |

### 2.3. Linear's Philosophy

According to Linear's team:

- **Projects** should be thought of as feature implementations
- **Issues** within a project are the requirements/stories of each feature
- Projects can contain links to external resources (PRDs, specs, etc.)
- Projects have a defined lifecycle (start → completion)

---

## 3. Mapping Our Methodology to Linear

### 3.1. Recommended Mapping

| Our Methodology       | Linear Entity                    | Notes                  |
| --------------------- | -------------------------------- | ---------------------- |
| Application (SMW)     | Team                             | One team per app       |
| Story Card (001, 002) | Project                          | Feature-level tracking |
| Bundled Stories       | Issues                           | Granular task tracking |
| Acceptance Criteria   | Issue descriptions / Checklist   | Validation items       |
| Technical Artifacts   | External links + workspace files | Stay in repo           |

### 3.2. What Goes Where

#### In Linear (Project Description)

The **story card's high-level content** goes into Linear's project description:

- Story description (When/I want/So I can)
- Acceptance criteria (as checklist)
- Scenarios (BDD) - summary or key scenarios
- Extended requirements summary
- Phase scope
- **Links to workspace artifacts**

#### In Workspace (Technical Artifacts)

These **remain in the workspace** but are **linked from Linear**:

| Artifact          | Workspace Location                   | Why Keep in Workspace            |
| ----------------- | ------------------------------------ | -------------------------------- |
| Sequence diagrams | `story-folder/sequence-diagram.puml` | Version-controlled, IDE-viewable |
| Actions & Queries | `story-folder/actions-queries.md`    | Detailed technical reference     |
| Mockups           | `story-folder/mockups/`              | Binary files, design tools       |
| API specs         | `story-folder/api.yml`               | Code generation, validation      |
| Data models       | `docs/project/specs/entities/`       | Shared across stories            |

### 3.3. Linking Strategy

Linear projects support **external links**. Reference workspace artifacts using:

1. **GitHub/GitLab URLs** (if docs are pushed to remote)
2. **Relative paths** in description (for developers working in IDE)

Example project description structure:

```markdown
## Story

**When** I need to find circular-economy services nearby...
**I want to** search by combining category, waste type/product, and distance filters
**So I can** quickly locate relevant providers that meet my specific needs

## Acceptance Criteria

- [ ] Home page allows users to select ONE activity type
- [ ] Users must enter an address or postcode before searching
- [ ] "Near me" option uses browser geolocation API
      ...

## Workspace Artifacts

- **Sequence Diagram:** `docs/project/specs/stories/consumers/browse-providers-by-waste-category/sequence-diagram.puml`
- **Actions & Queries:** `docs/project/specs/stories/consumers/browse-providers-by-waste-category/actions-queries.md`
- **Mockups:** `docs/project/specs/stories/consumers/browse-providers-by-waste-category/mockups/`
- **Full Story Card:** `docs/project/specs/stories/consumers/browse-providers-by-waste-category/story-card-001.md`
```

---

## 4. Multi-Business Configuration

### 4.1. The Challenge

As a solo developer with multiple business ventures:

- **Business A:** AI Agentic Products (LinkedIn Ghostwriter, Competitor Intelligence, Research Generator)
- **Business B:** Non-AI Apps (SMW - Smart Waste Management, others)

How should Linear be configured?

### 4.2. Option A: Single Workspace, Multiple Teams (Recommended for Solo Dev)

```
Your Linear Workspace ("Nick Thiru" or "NT Ventures")
├── Team: SMW (Smart Waste Management)
│   ├── Project: Browse Providers by Category (Story 001)
│   ├── Project: View Business Details (Story 002)
│   └── ...
├── Team: LinkedIn Ghostwriter
│   ├── Project: Voice Input MVP
│   ├── Project: Style Learning
│   └── ...
├── Team: Competitor Intelligence
│   └── ...
└── Team: Research Generator
    └── ...
```

**Pros:**

- Single Linear account/subscription
- Cross-team visibility
- Shared labels and workflows
- One MCP connection handles everything

**Cons:**

- All projects visible in one workspace (can be noisy)
- Less separation between business entities

### 4.3. Option B: Multiple Workspaces (For Distinct Businesses)

```
Workspace 1: "AI Agentic Products"
├── Team: LinkedIn Ghostwriter
├── Team: Competitor Intelligence
└── Team: Research Generator

Workspace 2: "SMW Ventures" (or similar)
├── Team: SMW
└── Team: Other Non-AI Apps
```

**Pros:**

- Complete separation between business entities
- Different billing/plans per workspace
- Cleaner mental model for distinct businesses

**Cons:**

- Multiple Linear subscriptions (if on paid plans)
- Need to switch workspaces in Linear UI
- **MCP limitation:** See section 4.4

### 4.4. MCP Server and Multiple Workspaces

**Current Limitation:** The Linear MCP server authenticates via OAuth to a **single Linear account**. Your Linear account can access multiple workspaces, but:

- The MCP server connects to your **default workspace** or the one authorized during OAuth
- To work with a different workspace, you would need to:
  1. Re-authorize the MCP connection to the other workspace, OR
  2. Use Linear's workspace switching (the API may support workspace selection)

**Practical Recommendation for Solo Dev:**

Use **Option A (Single Workspace, Multiple Teams)** because:

1. Simpler MCP setup - one connection works for all
2. Lower cost (one subscription)
3. You can still filter by team in Linear UI
4. Cross-project insights are valuable

**If you truly need separate workspaces:**

- Create them manually in Linear
- The MCP server will work with whichever workspace your OAuth token is scoped to
- You may need to re-authorize when switching business contexts

### 4.5. Recommended Setup for Your Situation

Given your two business areas:

```
Linear Workspace: "Nick Thiru" (or rename to something broader)
├── Team: SMW
│   └── (Smart Waste Management projects)
├── Team: AI-Products (or separate teams per product)
│   ├── LinkedIn Ghostwriter projects
│   ├── Competitor Intelligence projects
│   └── Research Generator projects
└── Team: [Other Apps]
    └── (Future non-AI apps)
```

This keeps everything in one workspace while maintaining clear separation via teams.

---

## 5. MCP Server Setup

### 5.1. Current Configuration

The Linear MCP server is configured in Windsurf's `mcp_config.json`:

```json
{
  "mcpServers": {
    "linear-mcp-server": {
      "args": ["-y", "mcp-remote", "https://mcp.linear.app/sse"],
      "command": "npx",
      "env": {}
    }
  }
}
```

### 5.2. How Authentication Works

- The MCP server uses **OAuth authentication** (not file paths or API keys in config)
- When first connecting, Linear prompts for authorization in browser
- The OAuth token grants access to your Linear workspace(s)
- **No workspace path is specified in args** - workspace access is determined by OAuth scope

### 5.3. Available MCP Tools

The Linear MCP server provides these tools for Cascade to use:

**Teams:**

- `list_teams` - List all teams in workspace
- `get_team` - Get team details

**Projects:**

- `list_projects` - List projects (filterable by team)
- `get_project` - Get project details
- `create_project` - Create new project
- `update_project` - Update project

**Issues:**

- `list_issues` - List issues (filterable)
- `get_issue` - Get issue details
- `create_issue` - Create new issue
- `update_issue` - Update issue

**Other:**

- `list_cycles` - Sprint/cycle management
- `list_issue_labels` - Available labels
- `create_issue_label` - Create new labels
- `list_users` - Team members
- `create_comment` - Add comments to issues

### 5.4. What MCP Cannot Do

- **Create workspaces** - Must be done manually in Linear UI
- **Create teams** - Must be done manually in Linear UI (as of current MCP version)
- **Manage workspace settings** - Billing, permissions, etc.

---

## 6. Practical Implementation

### 6.1. Initial Setup Steps

1. **Create Linear Workspace** (if not exists) - Manual in Linear UI
2. **Create Teams** for each app/product - Manual in Linear UI
3. **Create Labels** for workflow phases - Can be done via MCP
4. **Create Projects** for stories - Can be done via MCP
5. **Create Issues** for bundled stories - Can be done via MCP

### 6.2. Example: Setting Up SMW in Linear

**Step 1: Create Team (Manual)**

- Go to Linear UI → Settings → Teams → Create Team
- Name: "SMW"
- Key: "SMW" (for issue prefixes like SMW-1, SMW-2)

**Step 2: Create Labels (via MCP or Manual)**

```
- Phase: 1a-MVP
- Phase: 1b-Enhanced
- Phase: 2-Personalization
- Type: Frontend
- Type: Backend
- Type: Full-Stack
- Priority: High
- Priority: Medium
- Priority: Low
```

**Step 3: Create Project for Story 001 (via MCP)**

Project Name: `Browse Providers by Waste Category`
Project Description: (Story card content - see section 3.3)

**Step 4: Create Issues for Bundled Stories (via MCP)**

From Story 001's bundled stories:

- Issue: "Display category taxonomy on home page"
- Issue: "Select primary waste category"
- Issue: "Filter by waste type within category"
- Issue: "Filter by specific product"
- Issue: "Set distance radius for search"
- Issue: "View filtered results list"

### 6.3. Ongoing Workflow

1. **New Story Card Created** → Create Linear Project
2. **Implementation Starts** → Create Issues for sub-tasks
3. **Progress Updates** → Update issue status via MCP
4. **Artifacts Updated** → Reference in Linear comments
5. **Story Complete** → Close project in Linear

---

## 7. Workflow Guidelines

### 7.1. When to Use Linear vs Workspace

| Activity              | Use Linear | Use Workspace |
| --------------------- | ---------- | ------------- |
| Track progress        | ✓          |               |
| Assign work           | ✓          |               |
| Discuss requirements  | ✓          |               |
| Sprint planning       | ✓          |               |
| Write technical specs |            | ✓             |
| Create diagrams       |            | ✓             |
| Design mockups        |            | ✓             |
| Write API contracts   |            | ✓             |
| Code implementation   |            | ✓             |

### 7.2. Cascade's Role in Linear Management

Cascade (AI assistant) can:

- Create and update projects/issues based on story cards
- Update issue status as work progresses
- Add comments with progress updates
- Query project/issue status for context
- Create labels for organization

Cascade should:

- Always reference workspace artifacts by path
- Keep Linear descriptions concise (link to full docs)
- Update Linear when significant milestones are reached
- Use consistent naming conventions

### 7.3. Naming Conventions

**Projects:**

- Format: `[Story Title]`
- Example: `Browse Providers by Waste Category`

**Issues:**

- Format: `[Brief description]`
- Example: `Display category taxonomy on home page`

**Labels:**

- Phase labels: `phase:1a-mvp`, `phase:1b-enhanced`, `phase:2`
- Type labels: `type:frontend`, `type:backend`, `type:full-stack`
- Priority: Use Linear's built-in priority field

---

## 8. Artifact Management Strategy

### 8.1. Source of Truth

| Artifact Type      | Source of Truth                      | Synced To                    |
| ------------------ | ------------------------------------ | ---------------------------- |
| Story requirements | Workspace (story-card.md)            | Linear (project description) |
| Task status        | Linear (issues)                      | -                            |
| Technical specs    | Workspace (various .md, .puml, .yml) | Linear (links only)          |
| Code               | Workspace (source files)             | -                            |
| Discussions        | Linear (comments)                    | -                            |

### 8.2. Sync Strategy

**Workspace → Linear:**

- When story card is created/updated, update Linear project description
- Include key acceptance criteria and links to artifacts

**Linear → Workspace:**

- Status changes in Linear don't need to sync to workspace
- Major decisions from Linear discussions should be added to story card's "Decisions Log"

### 8.3. Artifact Locations Reference

```
docs/project/specs/stories/[actor]/[story-name]/
├── story-card-[ID].md          # Full story specification
├── sequence-diagram.puml        # Technical flow diagram
├── actions-queries.md           # API operations definition
├── api.yml                      # OpenAPI specification
└── mockups/
    ├── README.md                # Mockup descriptions
    └── *.png                    # UI mockups
```

---

## Appendix A: Linear MCP Tools Reference

### Teams

- `mcp0_list_teams` - List all teams
- `mcp0_get_team` - Get team by ID/name

### Projects

- `mcp0_list_projects` - List projects (filter by team, state, etc.)
- `mcp0_get_project` - Get project details
- `mcp0_create_project` - Create new project
- `mcp0_update_project` - Update project

### Issues

- `mcp0_list_issues` - List issues (many filter options)
- `mcp0_get_issue` - Get issue details
- `mcp0_create_issue` - Create new issue
- `mcp0_update_issue` - Update issue

### Labels

- `mcp0_list_issue_labels` - List available labels
- `mcp0_create_issue_label` - Create new label

### Other

- `mcp0_list_cycles` - List sprints/cycles
- `mcp0_list_users` - List team members
- `mcp0_create_comment` - Add comment to issue
- `mcp0_list_comments` - List comments on issue

---

## Appendix B: Example Project Description Template

```markdown
# [Story Title]

**Story ID:** [XXX]  
**Phase:** [1a-MVP / 1b-Enhanced / 2]  
**Status:** [Ready for Dev / In Progress / Done]

---

## Story

**When** [circumstance/trigger]  
**I want to** [motivation/action]  
**So I can** [goal/outcome]

---

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3
      ...

---

## Key Scenarios

### Scenario 1: [Name]

Given [context], When [action], Then [outcome]

### Scenario 2: [Name]

Given [context], When [action], Then [outcome]

---

## Workspace Artifacts

- **Full Story Card:** `docs/project/specs/stories/[actor]/[story]/story-card-[ID].md`
- **Sequence Diagram:** `docs/project/specs/stories/[actor]/[story]/sequence-diagram.puml`
- **Actions & Queries:** `docs/project/specs/stories/[actor]/[story]/actions-queries.md`
- **Mockups:** `docs/project/specs/stories/[actor]/[story]/mockups/`

---

## Technical Notes

- Backend Service: `svc-[service-name]`
- API Endpoint: `GET /api/[endpoint]`
- Data Model: See `docs/project/specs/entities/[entity].md`
```

---

## Appendix C: Practical Implementation Example

### Setup Completed (2025-11-29)

**Workspace:** Thiru AI Labs (`linear.app/thiru-ai-labs`)

**Team Created:** SMW (identifier: SMW)

### Labels Created

| Label               | Color            | Purpose                    |
| ------------------- | ---------------- | -------------------------- |
| `phase:1a-mvp`      | Blue (#0EA5E9)   | Phase 1a MVP work          |
| `phase:1b-enhanced` | Cyan (#06B6D4)   | Phase 1b enhanced features |
| `phase:2`           | Purple (#8B5CF6) | Phase 2 personalization    |
| `type:frontend`     | Orange (#F97316) | Frontend UI work           |
| `type:backend`      | Green (#22C55E)  | Backend service work       |
| `type:full-stack`   | Yellow (#EAB308) | Full-stack work            |

### Project Created

**Project:** Story 001: Browse Providers by Waste Category

- **URL:** https://linear.app/thiru-ai-labs/project/story-001-browse-providers-by-waste-category-68a42f7b6c24
- **Status:** Planned
- **Labels:** phase:1a-mvp, type:full-stack

### Issues Created

| ID     | Title                                  | Type     | Labels                      |
| ------ | -------------------------------------- | -------- | --------------------------- |
| SMW-5  | Display category taxonomy on home page | Frontend | phase:1a-mvp, type:frontend |
| SMW-6  | Select primary waste category          | Frontend | phase:1a-mvp, type:frontend |
| SMW-7  | Filter by waste type within category   | Frontend | phase:1a-mvp, type:frontend |
| SMW-8  | Filter by specific product             | Frontend | phase:1a-mvp, type:frontend |
| SMW-9  | Set distance radius for search         | Frontend | phase:1a-mvp, type:frontend |
| SMW-10 | View filtered results list             | Frontend | phase:1a-mvp, type:frontend |
| SMW-11 | Implement GET /merchants API endpoint  | Backend  | phase:1a-mvp, type:backend  |

### GitHub Repository Links

All artifacts linked to: `https://github.com/nickthiru3/smw-docs`

Artifact paths:

- Story card: `project/specs/stories/consumers/browse-providers-by-waste-category/story-card-001.md`
- Sequence diagram: `project/specs/stories/consumers/browse-providers-by-waste-category/sequence-diagram.puml`
- Actions & Queries: `project/specs/stories/consumers/browse-providers-by-waste-category/actions-queries.md`
- Mockups: `project/specs/stories/consumers/browse-providers-by-waste-category/mockups/`

---

## Revision History

| Date       | Version | Changes                                             |
| ---------- | ------- | --------------------------------------------------- |
| 2025-11-29 | 1.0     | Initial guide creation                              |
| 2025-11-29 | 1.1     | Added practical implementation example (Appendix C) |
