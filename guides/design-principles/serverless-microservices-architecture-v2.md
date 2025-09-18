# **Super-Deals: Serverless Microservices Architecture Guide**

_Version 2.0_

This document serves as the complete technical reference for the serverless microservices architecture powering the Super-Deals application. It details our core principles, operating model, technical patterns, and development workflows. All teams should consider this guide the "source of truth" for architectural decisions.

## **Table of Contents**

- [**Super-Deals: Serverless Microservices Architecture Guide**](#super-deals-serverless-microservices-architecture-guide)
  - [**Table of Contents**](#table-of-contents)
  - [**1. Guiding Principles \& Philosophy**](#1-guiding-principles--philosophy)
  - [**2. Architecture Overview \& Core Components**](#2-architecture-overview--core-components)
    - [**High-Level Diagram**](#high-level-diagram)
    - [**Core AWS Components**](#core-aws-components)
  - [**3. Team Structure \& Operating Model**](#3-team-structure--operating-model)
  - [**4. The "Microservice Construct": A Self-Contained Unit**](#4-the-microservice-construct-a-self-contained-unit)
  - [**5. Data Architecture: Strict Data Isolation**](#5-data-architecture-strict-data-isolation)
  - [**6. Service Communication \& Discovery**](#6-service-communication--discovery)
  - [**7. Development \& Release Workflow**](#7-development--release-workflow)
  - [**8. Repository \& Project Structure (work in progress)**](#8-repository--project-structure-work-in-progress)
  - [**9. Managing Coordination Costs in Our Architecture**](#9-managing-coordination-costs-in-our-architecture)
    - [**1. Securing Microservice APIs (BFF API to Microservice)**](#1-securing-microservice-apis-bff-api-to-microservice)
    - [**2. Evolving the "Golden Path" (Platform Team to Microservice Teams)**](#2-evolving-the-golden-path-platform-team-to-microservice-teams)
  - [**10. Benefits \& Acknowledged Trade-offs**](#10-benefits--acknowledged-trade-offs)
    - [**Benefits**](#benefits)
    - [**Trade-offs**](#trade-offs)

---

## **1. Guiding Principles & Philosophy**

Our architecture is built on a foundation of principles designed to maximize development velocity while ensuring system stability. These principles are derived from the book "Microservices: Up and Running" and adapted for our serverless context.

- **Minimize Coordination Costs:** This is our primary driver. The architecture must allow teams to develop, test, and deploy their services with minimal dependency on other teams.
- **High Autonomy, High Alignment:** Teams are autonomous in their implementation but aligned through shared patterns and automated guardrails defined in this guide.
- **Strict Data Isolation:** Each microservice owns and controls its own data store. There is no shared data layer. Changes to a service's data model must not impact any other service.
- **You Build It, You Run It (with Guardrails):** BFF API and Microservice teams are responsible for the entire lifecycle of their services. A Platform Team provides the tools and "golden paths" to make this possible.
- **Infrastructure as Code (IaC) is Mandatory:** All infrastructure components must be defined as code (using AWS CDK) and deployed via an automated pipeline. This ensures consistency and repeatability.
- **Right Tool for the Job (Orchestration vs. Choreography):** We use AWS Step Functions for orchestrating complex business transactions and SNS/SQS for choreographing simple, decoupled events.

## **2. Architecture Overview & Core Components**

We employ a serverless architecture on AWS, replacing traditional containers and Kubernetes with managed services to reduce operational overhead.

### **High-Level Diagram**

```
Browser/Mobile Client
       ↓
Backend-for-Frontend (BFF) API [API Gateway + Lambda]
    |
    |  (Orchestrates business workflows using Step Functions)
    |  (Discovers services via Parameter Store)
    ↓
----------------------------------------------------------------------
| Microservice Boundary | Microservice Boundary | Microservice Boundary |
|-----------------------|-----------------------|-----------------------|
|  ms-deals             |  ms-merchants         |  ms-users             |
|  - API Gateway        |  - API Gateway        |  - API Gateway        |
|  - Lambda             |  - Lambda             |  - Lambda             |
|  - DynamoDB Table     |  - DynamoDB Table     |  - S3 Bucket          |
|  - SNS Topic (Events) |  - SNS Topic (Events) |                       |
----------------------------------------------------------------------
       ↑      ↓ (Subscribes to events via SQS)
External Systems & Asynchronous Listeners
```

### **Core AWS Components**

- **Compute:** AWS Lambda for all business logic.
- **API Layer:** Amazon API Gateway for all synchronous service interfaces.
- **Data Persistence:** Amazon DynamoDB (one dedicated table per service).
- **Object Storage:** Amazon S3 (one dedicated bucket per service).
- **Asynchronous Communication:** Amazon SNS (for publishing events) and Amazon SQS (for subscribing to events).
- **Workflow Orchestration:** AWS Step Functions (managed by the BFF) for complex, multi-service transactions.
- **Service Discovery:** AWS Systems Manager Parameter Store for registering and discovering service endpoints (API URLs, SNS Topic ARNs, etc.).
- **Infrastructure as Code:** AWS Cloud Development Kit (CDK) in TypeScript.
- **CI/CD:** GitHub Actions.

## **3. Team Structure & Operating Model**

Our operating model, "Autonomous Teams with Release Gates," is designed for clarity and speed.

- **System Design Team:**

  - **Type:** Enabling Team.
  - **Responsibility:** This is a small group of senior architects and technical leaders responsible for the macro-level design of our socio-technical system. Their core responsibilities include designing the team structures, establishing the "golden path" standards and guardrails for all teams, and continually improving the overall system architecture based on feedback and metrics. They own and maintain this very architecture guide as the canonical source of truth for our standards.
  - **Skills:** Deep system architecture knowledge, understanding of organizational design, and strong leadership to align multiple teams towards a shared technical vision.
  - **Collaborative Interactions:** This team primarily uses a **facilitating** interaction model. They consult with and support the BFF API and microservice teams to help them navigate the system and adhere to established patterns, rather than acting as a gatekeeper.

- **BFF API Team (`super-deals-web-bff-api`, etc.):**

  - **Type:** Stream-Aligned Team.
  - **Responsibility:** To design, build, and maintain the Backend-for-Frontend APIs that serve specific client applications (e.g. the main web app). Their primary role is to orchestrate calls to downstream microservices to fulfill client-side requests, effectively shielding the frontend from the complexity of the microservice ecosystem. They own the complex business workflows (e.g. using Step Functions).
  - **Skills:** Expertise in API design, service orchestration, and understanding client application needs. They will be heavy users of service discovery and asynchronous communication patterns.
  - **Collaborative Interactions:** As the primary consumer of the microservices, the BFF API team works closely with microservice teams. They will frequently initiate pull requests against microservice repositories to request the access permissions they need, following the collaborative workflow defined in the guide.

- **Microservice Teams** (`ms-deals`, `ms-users`, etc.):

  - **Type:** Stream-Aligned Teams.
  - **Responsibility:** Full ownership of a business domain and its corresponding microservice(s). This includes designing, building, deploying, and monitoring their service. A key part of this ownership involves managing the service's API and security policies.
  - **Skills:** Deep domain knowledge, business logic implementation (e.g., Node.js/TypeScript), basic AWS CDK for defining their service construct.
  - **Collaborative Interactions:** While autonomous, teams must collaborate on changes that cross service boundaries. For instance, if the BFF API team needs to access a new microservice endpoint, they are responsible for initiating a pull request against the microservice's repository to update the necessary security policies. The microservice team, as the owner, is then responsible for reviewing and approving this change, ensuring a secure and auditable process.

- **Platform Team:**

  - **Type:** Enabling Team (delivering a platform as a product).
  - **Responsibility:** To act as enablers, not gatekeepers. They are responsible for implementing the vision set by the **System Design Team**. They build and maintain the "golden path"—the practical tools like CI/CD pipeline templates, shared CDK constructs, security checks, and core architectural patterns. They make it easy for feature teams to do the right thing and use the platform as a self-service utility.
  - **Skills:** Deep expertise in AWS, CDK, IaC, security, and CI/CD.
  - **Collaborative Interactions:** This team primarily uses a **facilitating** interaction model. They consult with and support the Platform and Feature teams to help them navigate the system and adhere to established patterns, rather than acting as a gatekeeper.

- **Release Manager (A Role, Not a Team):**
  - **Responsibility:** A designated individual (e.g., the feature team's lead or product owner) provides the final manual approval for a production deployment after verifying its success in the staging environment. This provides a human check and auditable release gate without creating a central bottleneck.

## **4. The "Microservice Construct": A Self-Contained Unit**

Each microservice lives in its **own GitHub repository** and is defined as a deployable **CDK `app` project**. This CDK application defines a stack that encapsulates all resources owned by the service.

A typical `ms-deals` CDK stack would define:

1.  **API Gateway:** A REST API that serves as the service's synchronous entry point.
2.  **Lambda Function:** The core business logic.
3.  **DynamoDB Table:** A dedicated table for all `ms-deals` data (e.g., Deals, Vouchers).
4.  **S3 Bucket:** A dedicated bucket for deal-related assets (e.g., images).
5.  **SNS Topic:** A topic for publishing domain events like `DealCreated`.
6.  **IAM Roles & Policies:** Finely-grained permissions granting the Lambda function least-privilege access to its own resources.

This self-contained model is the cornerstone of our independent deployability.

## **5. Data Architecture: Strict Data Isolation**

This is a non-negotiable principle of our architecture. Data coupling is a primary cause of failure in microservice implementations.

- **DynamoDB:** Each service that requires key-value persistence **must** provision its own dedicated DynamoDB table. Within this table, the service should use a **single-table design pattern** to model its various entities. We do **not** use a shared database cluster or table across microservices.
- **S3:** Each service that requires object storage **must** provision its own dedicated S3 bucket.
  - **Challenge:** S3 bucket names must be globally unique.
  - **Solution:** The CDK app will programmatically generate a unique bucket name by appending a hash to a logical ID (e.g., `super-deals-ms-deals-storage-<hash>`). The final name will be published to Parameter Store for discovery by other services.

## **6. Service Communication & Discovery**

Services must remain loosely coupled. Hardcoding ARNs or URLs is strictly forbidden. We use a centralized registry for dynamic service discovery.

1.  **Publisher's Responsibility:** When a service (e.g., `ms-deals`) is deployed via its CI/CD pipeline, a final step **publishes its key endpoints to AWS Systems Manager Parameter Store**.

    - `/services/ms-deals/api/url` → `https://abc123xyz.execute-api.us-east-1.amazonaws.com/prod/`
    - `/services/ms-deals/events/DealCreated/arn` → `arn:aws:sns:us-east-1:123456789012:ms-deals-DealCreatedTopic-a1b2c3d4`
    - `/services/ms-deals/storage/bucketName` → `super-deals-ms-deals-storage-a1b2c3d4`

2.  **Consumer's Responsibility:** When a consumer service (e.g. the BFF API) needs to interact with `ms-deals`, its CDK code reads the required endpoint from Parameter Store during deployment (`cdk deploy`) to configure its IAM policies and environment variables. This creates a binding at deployment time, not development time, ensuring loose coupling.

## **7. Development & Release Workflow**

The workflow is designed for developer autonomy and production safety.

1.  **Local Development:** A developer works on their service in its dedicated repository.
2.  **Pull Request:** On creating a PR, the CI pipeline automatically runs:
    - Linting and static analysis.
    - Unit tests.
    - IaC security scanning (e.g., `cdk-nag`).
3.  **Merge to `main`:** On merge, the CD pipeline automatically deploys the service to the **`staging`** environment. Automated integration tests are run against the `staging` environment.
4.  **Promotion to Production (The Release Gate):**
    - The pipeline **pauses** and awaits manual approval.
    - The designated **Release Manager** verifies the service's health and functionality in `staging`.
    - Upon approval, the pipeline proceeds.
5.  **Safe Production Deployment:** The pipeline uses a safe deployment strategy, such as a **canary release** with AWS Lambda Aliases, to gradually shift traffic to the new version while monitoring for errors. The deployment is automatically rolled back if error rates exceed a defined threshold.

## **8. Repository & Project Structure (work in progress)**

```
# Each microservice team owns their repo
super-deals-ms-deals/
├── bin/ms-deals-app.ts        # CDK App entrypoint
├── lib/ms-deals-stack.ts      # CDK Stack definition (API, Lambda, DynamoDB, etc.)
├── src/                        # Lambda handler source code
│   ├── handlers/
│   │   └── create-deal.ts
│   └── services/
│       └── deals-service.ts
├── test/                       # Unit and integration tests
├── cdk.json
└── package.json

# The BFF team owns its repo
super-deals-bff-api-web/
├── bin/bff-app.ts
├── lib/bff-stack.ts           # Defines BFF API Gateway, Lambdas, and Step Functions
├── src/
│   ├── handlers/
│   └── state-machines/        # Step Function definitions
└── ...
```

## **9. Managing Coordination Costs in Our Architecture**

A primary goal of this architecture is to reduce coordination costs between teams to increase development velocity. However, in any distributed system, some level of coordination is unavoidable and necessary for security and system coherence. Our model explicitly identifies and manages these points of coordination rather than pretending they don't exist.

The two primary areas where inter-team coordination is required are:

### **1. Securing Microservice APIs (BFF API to Microservice)**

A microservice team (e.g. `ms-deals`) must ensure its API is only accessible by authorized callers, such as the BFF API layer. The microservice owns its security policy, but the BFF API team is the consumer that needs access. This creates a necessary point of coordination.

- **The Scenario:** The `BFF-API` team needs to call the new `createDeal` endpoint on the `ms-deals` service.
- **The Coordination Point:** The `ms-deals` API Gateway resource policy must be updated to grant `execute-api:Invoke` permission to the BFF API team's Lambda execution role ARN.
- **The Workflow (to manage the coordination cost):**
  1.  The `BFF-API` team identifies the need to call the new endpoint.
  2.  The `BFF-API` team finds the ARN of its Lambda function (from its own pipeline outputs or AWS console).
  3.  The `BFF-API` team raises a pull request **against the `super-deals-ms-deals` repository**, specifically modifying the CDK code (`lib/ms-deals-stack.ts`) to add its Lambda role ARN to the API Gateway's resource policy.
  4.  The `ms-deals` team (as the code owners) reviews the PR. They are responsible for validating that the request is legitimate and correctly implemented. They then approve and merge it.

This process is a form of **"X-as-a-service" with collaboration**. While the microservice owns its policy (the "service"), granting access requires a collaborative pull request. This workflow keeps the process transparent, version-controlled, and auditable, which is a significant improvement over informal requests.

### **2. Evolving the "Golden Path" (Platform Team to Microservice Teams)**

The Platform Team provides the core CI/CD pipelines and shared CDK patterns as a service to all microservice teams. When the Platform Team needs to roll out a change—such as a mandatory security scanner, an updated Lambda runtime, or a new logging library—it impacts all consumer teams.

- **The Scenario:** The Platform Team decides to enforce a new, more stringent `cdk-nag` rule for IaC security across all services.
- **The Coordination Point:** This change must be propagated to every microservice's CI/CD pipeline configuration or template.
- **The Workflow (to manage the coordination cost):**
  1.  **Communication First:** The Platform Team announces the upcoming change, its benefits, and the deadline for adoption via established channels (e.g., mailing list, Slack).
  2.  **Implementation:** The Platform team updates the central "golden path" pipeline template that new services will inherit.
  3.  **Rollout Support:** For existing services, the Platform team can take two approaches:
      - **Facilitating:** Publish clear documentation and offer office hours to help microservice teams adopt the change in their repositories.
      - **Collaborating:** Use automated tooling (e.g., scripts) to open pull requests with the required changes against all service repositories, leaving the final review and merge to the owning microservice team.

This interaction mode is a mix of **facilitating** and **X-as-a-service**. The key is that the Platform Team enables the change but the microservice team, as the ultimate owner of their service and its pipeline, has the final say on merging and deploying it. This preserves team autonomy while allowing for centrally-managed improvements.

## **10. Benefits & Acknowledged Trade-offs**

### **Benefits**

- **Team Autonomy & Speed:** The primary benefit. Teams can release features independently.
- **High Scalability & Resilience:** Built on highly-available, auto-scaling managed AWS services.
- **Lower Operational Overhead:** Serverless removes the need to manage servers, containers, or clusters.
- **Enforced Best Practices:** The "golden path" pipeline and patterns bake in security and quality checks.

### **Trade-offs**

- **Increased Architectural Complexity:** A distributed system is inherently more complex than a monolith. We are trading development complexity for organizational scalability.
- **Vendor Lock-in:** We are tightly coupled to the AWS ecosystem. This is a deliberate trade-off for the velocity gained from using its managed services.
- **Debugging Challenges:** Investigating issues across multiple services requires robust distributed tracing (e.g., AWS X-Ray), which must be implemented from day one.
- **Potential for Cold Starts:** We will use Provisioned Concurrency for user-facing, latency-sensitive Lambda functions (like those in the BFF) to mitigate this.
