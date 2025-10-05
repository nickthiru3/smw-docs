# Infrastructure Testing Guide: CDK Template Assertions

This guide explains how we validate our AWS CDK stacks using `aws-cdk-lib/assertions`. Template tests ensure infra resources exist with the right configuration and guard against accidental drifts.

## Scope

- Stacks and constructs under `lib/` (service infra only; CI/CD pipeline tests are out of current scope).
- For `deals-ms`, see `ServiceStack` and `ApiConstruct`.

## Where tests live

- `test/lib/service-stack.test.ts` contains assertions for:
  - API Gateway `RestApi`, `Deployment`, `Stage` (logging, metrics, throttling).
  - Validation features: `GatewayResponse` (BAD_REQUEST_BODY), `RequestValidator`, `Model`.
  - Deals endpoint: method, integration, authorizer fields present if used.
  - Lambda: runtime, memory, timeout, `TABLE_NAME` environment, IAM policy restricted to `dynamodb:PutItem` on the app table.
  - DynamoDB: PK/SK, GSI1, PITR enabled.

## Example assertions

```ts
import { Template, Match } from "aws-cdk-lib/assertions";

// Stage settings
template.hasResourceProperties("AWS::ApiGateway::Stage", {
  MethodSettings: Match.arrayWith([
    Match.objectLike({
      HttpMethod: "*",
      ResourcePath: "/*",
      LoggingLevel: "INFO",
      MetricsEnabled: true,
      DataTraceEnabled: true,
    }),
  ]),
});

// Access logs LogGroup
template.hasResourceProperties("AWS::Logs::LogGroup", Match.objectLike({
  LogGroupName: `/apigateway/${serviceName}/${envName}/access`,
  RetentionInDays: 30,
}));
```

See the full file for more examples: `test/lib/service-stack.test.ts`.

## Running tests

- `npm test`

## Tips

- Prefer `Match.objectLike` when the full resource is verbose; assert only the critical parts.
- Use `resourceCountIs` for cardinality when name matching is brittle.
- Keep pipeline/CD pipeline tests skipped or out-of-scope if not needed to avoid noise during app development.

## CI (Quick Note)

- Run CDK template tests on every PR alongside unit and handler tests.
- They are fast and do not require AWS credentials.
- See `guides/development/cicd-guide-v2.md` for how they fit into the pipeline. E2E and LocalStack remain outside PRs unless explicitly enabled.
