# Serverless Microservices Architecture Guide

_Version 3.0_

This document serves as the complete technical reference for the serverless microservices architecture powering the full-stack application. It details the core principles, operating model, technical patterns, and development workflows. All teams should consider this guide the "source of truth" for architectural decisions.

**Major Changes in v3.0:**

- **Revolutionary BFF Architecture**: BFF layer is now embedded within frontend applications using SvelteKit `+server.ts` files
- **Simplified Team Structure**: Eliminated separate BFF API teams; frontend teams now own their BFF logic
- **Enhanced Security**: Microservices are never exposed to public internet; only accessible via BFF layer
- **Reduced Coordination**: Frontend developers can modify their BFF logic without backend team coordination

## Table of Contents

1. [Guiding Principles & Philosophy](#1-guiding-principles--philosophy)
2. [Architecture Overview & Core Components](#2-architecture-overview--core-components)
   - [High-Level Diagram](#high-level-diagram)
   - [Core AWS Components](#core-aws-components)
3. [Team Structure & Operating Model](#3-team-structure--operating-model)
4. [The Frontend-Embedded BFF Pattern](#4-the-frontend-embedded-bff-pattern)
   - [SvelteKit Implementation](#sveltekit-implementation)
   - [Security & Authentication](#security--authentication)
   - [Data Orchestration & Tailoring](#data-orchestration--tailoring)
5. [The "Microservice Construct": A Self-Contained Unit](#5-the-microservice-construct-a-self-contained-unit)
6. [Data Architecture: Strict Data Isolation](#6-data-architecture-strict-data-isolation)
7. [Service Communication & Discovery](#service-communication--discovery)
8. [Development & Release Workflow](#development--release-workflow)
9. [Repository & Project Structure](#repository--project-structure)
10. [Managing Coordination Costs in Our Architecture](#managing-coordination-costs-in-our-architecture)
11. [Benefits & Acknowledged Trade-offs](#benefits--acknowledged-trade-offs)

---

## 1. Guiding Principles & Philosophy

Our architecture is built on a foundation of principles designed to maximize development velocity while ensuring system stability. These principles are derived from the book _Microservices: Up and Running_ and adapted for our serverless context.

- **Minimize Coordination Costs:** This is our primary driver. The architecture must allow teams to develop, test, and deploy their services with minimal dependency on other teams.
- **Frontend-Owned BFF Logic:** The BFF layer is owned and coded by frontend developers within their application, eliminating coordination overhead with separate backend teams.
- **High Autonomy, High Alignment:** Teams are autonomous in their implementation but aligned through shared patterns and automated guardrails defined in this guide.
- **Strict Data Isolation:** Each microservice owns and controls its own data store. There is no shared data layer. Changes to a service's data model must not impact any other service.
- **You Build It, You Run It (with Guardrails):** Feature teams are responsible for the entire lifecycle of their services. A Platform Team provides the tools and "golden paths" to make this possible.
- **Infrastructure as Code (IaC) is Mandatory:** All infrastructure components must be defined as code (using AWS CDK) and deployed via an automated pipeline. This ensures consistency and repeatability.
- **Right Tool for the Job (Orchestration vs. Choreography):** We use AWS Step Functions for orchestrating complex business transactions and SNS/SQS for choreographing simple, decoupled events.

## 2. Architecture Overview & Core Components

We employ a serverless architecture on AWS, replacing traditional containers and Kubernetes with managed services to reduce operational overhead. The key innovation in v3.0 is the **frontend-embedded BFF pattern**.

### High-Level Diagram

```
Browser/Mobile Client
       ↓
Frontend Application (SvelteKit + SST)
├── +page.svelte (Client-side UI)
├── +page.server.ts (SSR Load Functions)
└── +server.ts (BFF API Routes) ← **THIS IS THE BFF LAYER**
       ↓ (Internal AWS network, not public)
Domain Microservice Lambdas
├── Deals Service Lambda ←→ DynamoDB (Single Table)
├── Users Service Lambda ←→ DynamoDB (Single Table)
├── Reservations Service Lambda ←→ DynamoDB (Single Table)
└── Notifications Service Lambda ←→ SES/SNS
    ↓
EventBridge (Event Routing)
    ↓
External Integrations (Third-party APIs)
```

### Core AWS Components

- **Frontend Hosting:** SvelteKit applications deployed via SST (Serverless Stack Toolkit) to AWS Lambda + CloudFront + S3
- **BFF Layer:** SvelteKit `+server.ts` files running on AWS Lambda (same Lambda as the frontend app)
- **Compute:** AWS Lambda functions (one per microservice, plus the frontend Lambda)
- **Data Storage:** Amazon DynamoDB with single-table design per microservice
- **Object Storage:** Amazon S3 (one dedicated bucket per service)
- **Asynchronous Communication:** Amazon SNS (for publishing events) and Amazon SQS (for subscribing to events)
- **Workflow Orchestration:** AWS Step Functions (invoked by BFF layer) for complex, multi-service transactions
- **Service Discovery:** AWS Systems Manager Parameter Store for registering and discovering service endpoints
- **Infrastructure as Code:** AWS Cloud Development Kit (CDK) in TypeScript

## 3. Team Structure & Operating Model

Our team structure has been significantly simplified in v3.0 to eliminate coordination overhead:

- **Platform Team:**

  - **Type:** Enabling Team
  - **Responsibility:** Designs team structures, establishes "golden path" standards and guardrails, maintains this architecture guide, provides shared tooling and CDK constructs
  - **Skills:** Deep system architecture knowledge, understanding of organizational design, strong leadership
  - **Collaborative Interactions:** Facilitating interaction model - consults with and supports feature teams

- **Feature Teams** (`web-app`, `mobile-app`, etc.):

  - **Type:** Stream-Aligned Teams
  - **Responsibility:** End-to-end ownership of a complete user experience, including:
    - Frontend application (UI/UX)
    - **BFF logic embedded within the frontend** (`+server.ts` files)
    - Coordination with microservice teams for data needs
  - **Skills:** Frontend development (SvelteKit, React Native, etc.), API design, understanding of user experience needs
  - **Team Composition:** Frontend developers, UX designers, product managers
  - **Collaborative Interactions:** X-as-a-Service with microservice teams

- **Microservice Teams** (`ms-deals`, `ms-users`, etc.):
  - **Type:** Stream-Aligned Teams
  - **Responsibility:** Business capability implementation as reusable services:
    - Domain logic and business rules
    - Data management and persistence
    - Event publishing for system-wide coordination
    - **Internal APIs consumed only by BFF layers** (not public)
  - **Skills:** Backend development, domain expertise, data modeling, event-driven architecture
  - **Team Composition:** Backend developers, domain experts, data engineers
  - **Collaborative Interactions:** X-as-a-Service with feature teams

## 4. The Frontend-Embedded BFF Pattern

This is the revolutionary change in v3.0. Instead of separate BFF services, the BFF logic lives within the frontend application itself.

### SvelteKit Implementation

**File Structure:**

```
src/
├── routes/
│   ├── +page.svelte              # Client-side UI components
│   ├── +page.server.ts           # SSR load functions (initial page data)
│   └── api/
│       ├── deals/
│       │   └── +server.ts        # BFF API route for deals
│       ├── users/
│       │   └── +server.ts        # BFF API route for users
│       └── checkout/
│           └── +server.ts        # BFF orchestration for checkout flow
```

**Data Flow Patterns:**

1. **Initial Page Load (SSR):**

   ```typescript
   // +page.server.ts
   export async function load({ fetch }) {
     // Fetch essential data for SSR
     const deals = await fetch("/api/deals");
     return { deals: await deals.json() };
   }
   ```

2. **Client-Side Interactions:**

   ```typescript
   // +page.svelte
   async function handleUserAction() {
     // Subsequent API calls go to BFF routes
     const response = await fetch("/api/deals/search", {
       method: "POST",
       body: JSON.stringify({ query: searchTerm }),
     });
     const results = await response.json();
   }
   ```

3. **BFF API Route (Orchestration):**

   ```typescript
   // api/deals/+server.ts
   export async function POST({ request }) {
     const { query } = await request.json();

     // Orchestrate multiple microservice calls
     const [deals, userPrefs, inventory] = await Promise.all([
       fetch("https://deals-ms.internal/search", {
         method: "POST",
         body: JSON.stringify({ query }),
         headers: { Authorization: `Bearer ${await getServiceToken()}` },
       }),
       fetch("https://users-ms.internal/preferences", {
         headers: { Authorization: `Bearer ${await getServiceToken()}` },
       }),
       fetch("https://inventory-ms.internal/availability", {
         method: "POST",
         body: JSON.stringify({ dealIds: dealIds }),
       }),
     ]);

     // Aggregate and tailor response for frontend
     return json({
       deals: deals.data.map((deal) => ({
         id: deal.id,
         title: deal.title,
         price: deal.price,
         available: inventory.data[deal.id]?.available || false,
         recommended: userPrefs.data.categories.includes(deal.category),
       })),
     });
   }
   ```

### Security & Authentication

**Key Security Benefits:**

- **Microservices are never exposed to public internet**
- **No CORS issues** between frontend and BFF (same origin)
- **IAM-based service authentication** between BFF and microservices

**Authentication Flow:**

```typescript
// BFF authenticates with microservices using IAM roles
import { fromNodeProviderChain } from "@aws-sdk/credential-providers";

async function getServiceToken(): Promise<string> {
  // SST automatically provides IAM credentials to Lambda
  const credentials = fromNodeProviderChain();
  return generateSignedRequest(credentials);
}
```

### Data Orchestration & Tailoring

The BFF layer's primary value is orchestrating multiple microservice calls and tailoring responses for the specific frontend needs:

**Example: Complex Checkout Flow**

```typescript
// api/checkout/+server.ts
export async function POST({ request }) {
  const { dealId, userId, paymentMethod } = await request.json();

  // Step 1: Validate deal availability
  const deal = await dealsMicroservice.getDeal(dealId);
  if (!deal.available) {
    return json({ error: "Deal no longer available" }, { status: 400 });
  }

  // Step 2: Check user eligibility
  const user = await usersMicroservice.getUser(userId);
  if (user.accountStatus !== "active") {
    return json({ error: "Account not eligible" }, { status: 400 });
  }

  // Step 3: Process payment
  const payment = await paymentsMicroservice.processPayment({
    amount: deal.price,
    method: paymentMethod,
    userId,
  });

  // Step 4: Create reservation
  const reservation = await reservationsMicroservice.createReservation({
    dealId,
    userId,
    paymentId: payment.id,
  });

  // Step 5: Send confirmation
  await notificationsMicroservice.sendConfirmation({
    userId,
    reservationId: reservation.id,
    dealTitle: deal.title,
  });

  // Return tailored response for frontend
  return json({
    success: true,
    reservationId: reservation.id,
    confirmationNumber: reservation.confirmationNumber,
    estimatedDelivery: reservation.estimatedDelivery,
    // Only include data the frontend actually needs
    deal: {
      title: deal.title,
      image: deal.primaryImage,
    },
  });
}
```

## 5. The "Microservice Construct": A Self-Contained Unit

Each microservice lives in its **own GitHub repository** and is defined as a deployable **CDK `app` project**. This CDK application defines a stack that encapsulates all resources owned by the service.

A typical `ms-deals` CDK stack would define:

- **Lambda Function:** The compute layer that handles business logic
- **DynamoDB Table:** Single-table design for all service data
- **S3 Bucket:** For any file storage needs (images, documents, etc.)
- **SNS Topics:** For publishing domain events to other services
- **SQS Queues:** For subscribing to events from other services
- **IAM Roles & Policies:** Scoped permissions for the service
- **API Gateway:** **Internal-only endpoints** (not publicly accessible)
- **CloudWatch Alarms:** Monitoring and alerting for the service

**Key Constraint:** Microservices expose **internal APIs only**. They are never directly accessible from the public internet. All public access goes through the BFF layer.

## 6. Data Architecture: Strict Data Isolation

Each microservice owns and controls its own data store. There is **no shared data layer**. This principle is non-negotiable and ensures that services can evolve independently.

**Data Ownership Rules:**

- Each microservice has its own DynamoDB table(s)
- No service may directly query another service's database
- All inter-service data access must happen via API calls
- Services may cache data from other services, but the source of truth remains with the owning service

**Single Table Design:**
Each microservice uses DynamoDB's single-table design pattern:

```
PK (Partition Key) | SK (Sort Key) | GSI1PK | GSI1SK | Data
DEAL#123          | METADATA      | ACTIVE | 2024-01-15 | { title, price, ... }
DEAL#123          | REVIEW#456    | USER#789 | 2024-01-10 | { rating, comment, ... }
DEAL#123          | INVENTORY     | LOCATION#NYC | 50 | { quantity, reserved, ... }
```

## 7. Service Communication & Discovery

**Synchronous Communication (BFF ↔ Microservices):**

- **Protocol:** HTTPS with IAM authentication
- **Discovery:** AWS Systems Manager Parameter Store
- **Pattern:** Request-response for immediate data needs

**Asynchronous Communication (Microservice ↔ Microservice):**

- **Protocol:** SNS/SQS with EventBridge routing
- **Pattern:** Event-driven for eventual consistency and decoupling

**Service Registration Example:**

```typescript
// Each microservice registers its endpoints
await ssm
  .putParameter({
    Name: "/services/deals-ms/api-url",
    Value: "https://deals-ms-api.internal.company.com",
    Type: "String",
  })
  .promise();
```

**Service Discovery in BFF:**

```typescript
// BFF discovers microservice endpoints
const dealsApiUrl = await ssm
  .getParameter({
    Name: "/services/deals-ms/api-url",
  })
  .promise();
```

## 8. Development & Release Workflow

**Feature Development Flow:**

1. **Frontend Developer** identifies need for new data/functionality
2. **Frontend Developer** implements BFF API route in `+server.ts`
3. If new microservice capabilities needed:
   - **Frontend Developer** creates issue/request to relevant microservice team
   - **Microservice Team** implements and deploys new internal API endpoint
   - **Frontend Developer** updates BFF route to consume new endpoint
4. **Frontend Developer** deploys complete feature (frontend + BFF) as single unit

**Key Benefits:**

- **No coordination delays** for simple BFF changes
- **Single deployment unit** for frontend + BFF
- **Rapid iteration** on data requirements

## 9. Repository & Project Structure

**Frontend Applications:**

```
super-deals-web-app/
├── src/
│   ├── routes/
│   │   ├── +page.svelte
│   │   ├── +page.server.ts
│   │   └── api/           # BFF API routes
│   │       ├── deals/+server.ts
│   │       ├── users/+server.ts
│   │       └── checkout/+server.ts
├── sst.config.ts          # SST deployment config
├── package.json
└── README.md
```

**Microservices:**

```
super-deals-deals-ms/
├── lib/
│   ├── lambda/
│   │   └── handler.ts     # Business logic
│   ├── db/
│   │   └── construct.ts   # DynamoDB table definition
│   └── service-stack.ts   # CDK stack definition
├── bin/
│   └── app.ts            # CDK app entry point
├── cdk.json
├── package.json
└── README.md
```

## 10. Managing Coordination Costs in Our Architecture

The v3.0 architecture dramatically reduces coordination costs by eliminating the separate BFF API teams. However, some coordination points remain:

**Remaining Coordination Points:**

1. **New Microservice Capabilities:**

   - **When:** Frontend needs new data/functionality not yet available
   - **Process:** Frontend team creates request → Microservice team implements → Frontend team integrates
   - **Mitigation:** Well-defined API contracts and comprehensive microservice capabilities

2. **Cross-Service Transactions:**

   - **When:** Business processes span multiple microservices
   - **Process:** Feature team coordinates with multiple microservice teams
   - **Mitigation:** Use Step Functions for orchestration, minimize cross-service dependencies

3. **Schema Evolution:**
   - **When:** Microservice needs to change its API contract
   - **Process:** Backward-compatible changes preferred, coordinate breaking changes
   - **Mitigation:** API versioning, gradual migration strategies

**Eliminated Coordination Points:**

- ✅ **BFF API Changes:** Frontend developers can modify BFF logic independently
- ✅ **Data Shape Optimization:** No need to negotiate API responses with separate BFF team
- ✅ **Frontend-Specific Features:** Can be implemented entirely within frontend repository

## 11. Benefits & Acknowledged Trade-offs

### Benefits

**Dramatically Reduced Coordination:**

- Frontend developers can modify their BFF logic without any backend team involvement
- Single repository for frontend + BFF eliminates cross-repo coordination
- Rapid iteration on user experience and data requirements

**Enhanced Security:**

- Microservices are never exposed to public internet
- IAM-based authentication between BFF and microservices
- No CORS complexity between frontend and BFF

**Improved Developer Experience:**

- Frontend developers have complete control over their data layer
- Single deployment unit for frontend + BFF
- Familiar SvelteKit patterns for API development

**Better Performance:**

- BFF can optimize data fetching for specific UI needs
- Reduced network hops for frontend-to-backend communication
- Efficient data aggregation and caching strategies

### Trade-offs

**Increased Frontend Complexity:**

- Frontend developers must understand backend integration patterns
- More complex deployment pipeline (frontend + Lambda functions)
- Need to handle service authentication and error scenarios

**Potential Code Duplication:**

- Multiple frontend apps may implement similar BFF logic
- Mitigation: Shared libraries for common patterns

**Lambda Cold Starts:**

- BFF routes run on Lambda and may experience cold starts
- Mitigation: Provisioned concurrency for critical paths

**Limited to Frontend-Specific Logic:**

- Complex business logic should still live in microservices
- BFF should remain a thin orchestration layer
