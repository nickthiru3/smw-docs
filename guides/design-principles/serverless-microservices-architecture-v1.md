# Serverless Microservices Architecture: A Complete Reference Guide

## Table of Contents

1. [Introduction & Context](#introduction--context)
2. [Architecture Overview](#architecture-overview)
3. [Team Structure & Operating Model](#team-structure--operating-model)
4. [Platform Team: Infrastructure as Code](#platform-team-infrastructure-as-code)
5. [Data Architecture: Single-Table DynamoDB](#data-architecture-single-table-dynamodb)
6. [Development Workflow](#development-workflow)
7. [Service Discovery & Communication](#service-discovery--communication)
8. [Release Management & GitOps](#release-management--gitops)
9. [Benefits & Trade-offs](#benefits--trade-offs)
10. [Implementation Roadmap](#implementation-roadmap)

---

## Introduction & Context

This guide presents a serverless adaptation of the microservices architecture described in "Microservices: Up and Running." The core transformation involves moving from VPC/Kubernetes/Containers to AWS Lambda/API Gateway/DynamoDB while maintaining the book's fundamental principles.

### Key Serverless Transformations

**From Book's Architecture:**

- VPC/Kubernetes Infrastructure → AWS Lambda + API Gateway
- Container Pods → Individual Lambda Functions
- Container Registry → ECR Images for Lambda
- PostgreSQL/Redis → DynamoDB Single-Table Design
- Argo CD → Custom Serverless GitOps Controller

**Preserved Principles:**

- **Coordination cost reduction** through team autonomy
- **Data independence** with logical separation
- **Loose coupling** between services
- **Independent deployability** per service
- **Platform-as-a-Service model** for infrastructure

---

## Architecture Overview

### High-Level System Architecture

```
Web Application (Public Internet)
    ↓
CloudFront CDN (Static Assets)
    ↓
API Gateway (Rate Limiting, Authentication)
    ↓
Lambda Authorizer (Custom Auth)
    ↓
BFF Lambda Functions (Domain-Specific APIs)
├── Flight BFF Lambda
├── Reservation BFF Lambda
└── User Management BFF Lambda
    ↓ ↑ (Async via SNS/SQS)
Domain Microservice Lambdas
├── Flight Service Lambda ←→ DynamoDB (Single Table)
├── Reservation Service Lambda ←→ DynamoDB (Single Table)
├── User Service Lambda ←→ DynamoDB (Single Table)
└── Notification Service Lambda ←→ SES/SNS
    ↓
EventBridge (Event Routing)
    ↓
External Integrations (Third-party APIs)
```

### Layer Breakdown

**API Layer (BFF Pattern):**

- **Purpose**: Thin orchestration layer following the book's BFF recommendations
- **Implementation**: Domain-specific API Gateways with Lambda proxies
- **Benefits**: Client-specific optimization, reduced coordination between frontend and backend teams

**Microservices Layer:**

- **Purpose**: Business capability implementation
- **Implementation**: Individual Lambda functions per service
- **Benefits**: Independent scaling, deployment, and development

**Data Layer:**

- **Purpose**: Persistent storage with data independence
- **Implementation**: Single DynamoDB table with logical partitioning
- **Benefits**: Cost efficiency while maintaining data boundaries

---

## Team Structure & Operating Model

### Team Topology

Following the book's **x-as-a-service pattern**, teams are organized to minimize coordination costs:

**Platform Team:**

- **Skills**: Deep CDK/AWS expertise, infrastructure patterns
- **Responsibilities**: Generic CDK constructs, CI/CD templates, monitoring setup
- **Deliverables**: Reusable infrastructure libraries

**API Teams (BFF):**

- **Skills**: Node.js/Python, API design, service orchestration
- **Responsibilities**: Handler logic, route configuration, client optimization
- **Deliverables**: Lambda handlers + YAML configuration

**Microservices Teams:**

- **Skills**: Business domain knowledge, chosen programming language
- **Responsibilities**: Business logic, data models, service contracts
- **Deliverables**: Lambda handlers + YAML configuration

**Release Team:**

- **Skills**: Basic CDK, environment management, deployment coordination
- **Responsibilities**: Environment instantiation, release coordination
- **Deliverables**: Environment configurations

### Skill Requirements & Coordination Reduction

**Critical Design Decision**: Only the Platform Team requires CDK expertise. All other teams work with:

- Handler code (business logic)
- YAML configuration files
- Docker basics
- Domain-specific knowledge

This mirrors the book's clean skill separation and reduces coordination costs.

---

## Platform Team: Infrastructure as Code

### Generic CDK Construct Libraries

**Microservice Construct:**

```typescript
// @company/serverless-microservice-constructs
export class MicroserviceLambdaConstruct extends Construct {
  public readonly lambda: Function;

  constructor(scope: Construct, id: string, props: MicroserviceConfig) {
    super(scope, id);

    // Platform handles all infrastructure complexity
    this.lambda = new Function(this, "ServiceLambda", {
      functionName: props.functionName,
      code: Code.fromEcrImage(
        Repository.fromRepositoryName(this, "Repo", props.ecrRepository),
        { tagOrDigest: props.imageTag }
      ),
      handler: Handler.FROM_IMAGE,
      runtime: Runtime.FROM_IMAGE,
      environment: props.environment,
      timeout: Duration.seconds(props.timeout || 30),
      memorySize: props.memorySize || 512,
    });

    // Platform provides observability, security, monitoring
    this.setupObservability(props.serviceName);
    this.setupPermissions(props.permissions);
  }
}
```

**API Gateway Construct:**

```typescript
// @company/serverless-api-constructs
export class ServerlessApiConstruct extends Construct {
  public readonly api: RestApi;

  constructor(scope: Construct, id: string, configPath: string) {
    super(scope, id);

    const config = this.loadApiConfig(configPath);

    this.api = new RestApi(this, "Api", {
      description: config.description,
      endpointTypes: [EndpointType.REGIONAL],
      deploy: false,
    });

    // Auto-configure routes from config
    this.setupRoutesFromConfig(config);
  }
}
```

### Benefits of Platform Constructs

1. **Reduces Coordination Costs**: Teams don't need infrastructure expertise
2. **Standardization**: Common patterns across all services
3. **Self-Service**: Teams can deploy independently
4. **Expertise Centralization**: Platform team handles AWS complexities

---

## Data Architecture: Single-Table DynamoDB

### Design Principles

Following the book's guidance that **"microservices can share physical database clusters as long as they never share the same logical table space"**, we implement logical data separation through partition key conventions.

### Table Design

```typescript
// Platform Team: DynamoDB Construct
export class SingleTableDynamoConstruct extends Construct {
  constructor(scope: Construct, id: string, props: { envName: string }) {
    super(scope, id);

    this.table = new TableV2(this, "SharedTable", {
      partitionKey: { name: "PK", type: AttributeType.STRING },
      sortKey: { name: "SK", type: AttributeType.STRING },
      globalSecondaryIndexes: [
        {
          indexName: "GSI1",
          partitionKey: { name: "GSI1PK", type: AttributeType.STRING },
          sortKey: { name: "GSI1SK", type: AttributeType.STRING },
        },
      ],
      billingMode: BillingMode.PAY_PER_REQUEST,
      removalPolicy: RemovalPolicy.RETAIN,
    });
  }
}
```

### Data Partitioning Strategy

**Partition Key Convention:**

- Format: `SERVICE#<entity_id>`
- Examples:
  - `FLIGHTS#flight_123`
  - `RESERVATIONS#reservation_456`
  - `USERS#user_789`

**Benefits:**

- **Logical Separation**: Each service owns its data partition
- **Cost Efficiency**: Single table vs. multiple database clusters
- **Operational Simplicity**: One table to manage and monitor

### Access Control Implementation

**Service-Level IAM Policies:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:region:account:table/SharedTable",
      "Condition": {
        "ForAllValues:StringLike": {
          "dynamodb:LeadingKeys": ["FLIGHTS#*"]
        }
      }
    }
  ]
}
```

**Application-Level Safeguards:**

```typescript
// SDK wrapper enforcing partition boundaries
class ServiceDynamoClient {
  constructor(private serviceName: string) {}

  async getItem(key: string) {
    if (!key.startsWith(`${this.serviceName}#`)) {
      throw new Error(`Access denied: Invalid partition key prefix`);
    }
    // Proceed with DynamoDB operation
  }
}
```

### Data Independence Enforcement

**Critical Principles:**

1. **No Data Co-ownership**: Services never share logical data space
2. **Independent Deployability**: Data changes don't affect other services
3. **Autonomous Development**: Teams modify data structures independently

---

## Development Workflow

### Microservices Team Workflow

**1. Handler Development (No CDK Required):**

```typescript
// flight-service/src/index.ts
export const handler = async (event: APIGatewayProxyEvent) => {
  const flights = await flightService.searchFlights(
    event.queryStringParameters
  );
  return {
    statusCode: 200,
    body: JSON.stringify(flights),
  };
};
```

**2. Service Configuration:**

```yaml
# flight-service/service-config.yaml
serviceName: flight-service
functionName: FlightService
ecrRepository: company/flight-service
imageTag: latest
environment:
  NODE_ENV: production
  TABLE_NAME: shared-table
  SERVICE_PREFIX: FLIGHTS
permissions:
  - dynamodb:GetItem
  - dynamodb:Query
memorySize: 512
timeout: 30
```

### API Team Workflow

**1. BFF Handler Development:**

```typescript
// flight-bff/src/index.ts
export const handler = async (event: APIGatewayProxyEvent) => {
  // Orchestrate multiple services
  const flightService = new FlightServiceClient();
  const reservationService = new ReservationServiceClient();

  const flights = await flightService.getFlights();
  const availability = await reservationService.getAvailability();

  return {
    statusCode: 200,
    body: JSON.stringify({ flights, availability }),
  };
};
```

**2. API Configuration:**

```yaml
# flight-bff/api-config.yaml
apiName: flight-bff
routes:
  - path: /flights
    method: GET
    functionName: FlightBffHandler
  - path: /flights/search
    method: POST
    functionName: FlightSearchHandler
services:
  - name: flight-service
    ecrRepository: company/flight-bff
    imageTag: latest
```

### Repository Structure

```
├── platform-constructs/ (Platform team)
│   ├── lib/microservice-construct.ts
│   ├── lib/api-construct.ts
│   ├── lib/database-construct.ts
│   └── lib/service-discovery.ts
│
├── flight-service/ (Microservices team)
│   ├── src/index.ts
│   ├── service-config.yaml
│   ├── Dockerfile
│   └── package.json
│
├── flight-bff/ (API team)
│   ├── src/index.ts
│   ├── api-config.yaml
│   ├── Dockerfile
│   └── package.json
│
└── environment-dev/ (Release team)
    ├── lib/app-stack.ts
    ├── release-config.yaml
    └── bin/app.ts
```

---

## Service Discovery & Communication

### Platform-Provided Service Registry

```typescript
// Platform team: Service discovery abstraction
export class ServiceRegistryConstruct extends Construct {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    this.serviceRegistry = new StringParameter(this, "ServiceRegistry", {
      parameterName: "/services/registry",
      stringValue: JSON.stringify({
        "flight-service":
          "arn:aws:lambda:us-east-1:123456789012:function:FlightService",
        "reservation-service":
          "arn:aws:lambda:us-east-1:123456789012:function:ReservationService",
      }),
    });
  }
}
```

### Service Client Library

```typescript
// @company/service-client-library
export class ServiceClient {
  static async invoke(serviceName: string, payload: any) {
    const serviceArn = await this.getServiceArn(serviceName);
    return await lambda
      .invoke({
        FunctionName: serviceArn,
        Payload: JSON.stringify(payload),
      })
      .promise();
  }

  private static async getServiceArn(serviceName: string): Promise<string> {
    // Retrieve from Parameter Store service registry
    const registry = await this.getServiceRegistry();
    return registry[serviceName];
  }
}
```

---

## Release Management & GitOps

### Serverless GitOps Controller

```typescript
// Custom GitOps controller for Lambda functions
export class ServerlessGitOpsConstruct extends Construct {
  constructor(scope: Construct, id: string, props: GitOpsProps) {
    super(scope, id);

    const gitOpsController = new Function(this, "GitOpsController", {
      runtime: Runtime.NODEJS_18_X,
      handler: "index.handler",
      code: Code.fromAsset("src/gitops-controller"),
      environment: {
        RELEASE_REPO: props.releaseRepo,
        SERVICES_CONFIG: JSON.stringify(props.services),
      },
      timeout: Duration.minutes(5),
    });

    // Trigger on schedule (like Argo CD sync)
    new Rule(this, "GitOpsSyncRule", {
      schedule: Schedule.rate(Duration.minutes(1)),
      targets: [new LambdaFunction(gitOpsController)],
    });
  }
}
```

### Release Repository Structure

```
flight-system-release/
├── environments/
│   ├── staging/
│   │   └── services.yaml
│   └── production/
│       └── services.yaml
├── release-config.yaml
└── history/
    └── rollback-configs/
```

### Release Configuration

```yaml
# environments/staging/services.yaml
services:
  flight-service:
    imageTag: "v1.2.3"
    functionName: "StagingFlightService"
    ecrRepository: "company/flight-service"
  reservation-service:
    imageTag: "v1.1.0"
    functionName: "StagingReservationService"
    ecrRepository: "company/reservation-service"
```

---

## Benefits & Trade-offs

### Benefits

**Alignment with Book Principles:**

1. **Coordination Cost Reduction**: Teams work independently with clear interfaces
2. **Data Independence**: Logical separation ensures service autonomy
3. **Independent Deployability**: Services deploy without affecting others
4. **Platform-as-a-Service**: Infrastructure team provides self-service capabilities

**Serverless Advantages:**

- **Cost Efficiency**: Pay-per-use model vs. always-on containers
- **Automatic Scaling**: No capacity planning required
- **Operational Simplicity**: Managed services reduce operational burden
- **Faster Development**: No container orchestration complexity

### Trade-offs

**vs. Book's Container Approach:**

- **Cold Start Latency**: Initial Lambda invocation delay
- **Execution Time Limits**: 15-minute Lambda timeout constraint
- **Vendor Lock-in**: Tight coupling to AWS services
- **Debugging Complexity**: Distributed tracing more challenging

**Mitigation Strategies:**

- **Cold Start**: Use provisioned concurrency for critical paths
- **Time Limits**: Break long-running processes into async workflows
- **Vendor Lock-in**: Abstract AWS services behind interfaces
- **Debugging**: Implement comprehensive logging and tracing

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-2)

1. **Platform Team**: Create base CDK construct libraries
2. **Database Team**: Implement single-table DynamoDB design
3. **Platform Team**: Set up basic service registry

### Phase 2: Core Services (Weeks 3-4)

1. **Microservices Teams**: Implement first service with configuration-driven approach
2. **API Teams**: Create first BFF Lambda with orchestration logic
3. **Release Team**: Set up basic deployment pipeline

### Phase 3: GitOps & Advanced Features (Weeks 5-6)

1. **Platform Team**: Implement serverless GitOps controller
2. **All Teams**: Add monitoring, logging, and observability
3. **Release Team**: Configure multi-environment deployment

### Phase 4: Production Readiness (Weeks 7-8)

1. **Security**: Implement comprehensive IAM policies
2. **Performance**: Add provisioned concurrency and optimization
3. **Reliability**: Set up comprehensive monitoring and alerting
4. **Documentation**: Create team-specific operational guides

---

## Configuration Examples

### Complete Service Configuration

```yaml
# Microservice Configuration
serviceName: flight-service
functionName: FlightService
ecrRepository: company/flight-service
imageTag: v1.2.3
environment:
  NODE_ENV: production
  TABLE_NAME: shared-table
  SERVICE_PREFIX: FLIGHTS
  LOG_LEVEL: info
permissions:
  - dynamodb:GetItem
  - dynamodb:Query
  - dynamodb:PutItem
memorySize: 512
timeout: 30
reservedConcurrency: 100
```

### API Gateway Configuration

```yaml
# BFF API Configuration
apiName: flight-bff
description: Flight booking BFF API
routes:
  - path: /flights
    method: GET
    functionName: FlightSearchHandler
    cors: true
    auth: required
  - path: /flights/{id}
    method: GET
    functionName: FlightDetailsHandler
    cors: true
    auth: required
services:
  - name: flight-service
    ecrRepository: company/flight-bff
    imageTag: latest
```

---

## Conclusion

This serverless microservices architecture successfully adapts the book's principles to a Lambda-based environment while maintaining the core benefits of **coordination cost reduction**, **data independence**, and **team autonomy**. The configuration-driven approach ensures that teams can focus on business logic rather than infrastructure complexity, aligning with the book's vision of **platform-as-a-service** within organizations.

The architecture provides a foundation that can be extended and refined based on specific implementation needs while preserving the fundamental principles that make microservices architectures successful.
