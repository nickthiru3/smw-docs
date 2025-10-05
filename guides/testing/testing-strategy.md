# Testing Strategy (Overview)

This is the high-level overview of testing for `deals-ms`. It links to focused guides for each layer.

## Layers

- Unit (helpers): see `docs/guides/testing/unit-helpers-testing-guide.md`
- Handler behavior (with mocks): see `docs/guides/testing/handler-testing-guide.md`
- Contract/schema (Zod): see `docs/guides/testing/schema-testing-guide.md`
- Infrastructure/CDK templates: see `docs/guides/testing/cdk-template-testing-guide.md`
- E2E (Supertest against deployed API): see `docs/guides/testing/e2e-testing-guide.md`

## Conventions

- Mirror code structure under `test/` (e.g., tests for `lib/api/endpoints/deals/post/` live under `test/lib/api/endpoints/deals/post/`).
- Jest base directory points at `<rootDir>/test` to collect all tests under `test/`.
- Avoid using `process.env` in app code; for E2E, use `outputs.json` and `.e2e/` config files.
- No LocalStack-based integration tests for now.

## CI (Alignment)

Follow `guides/development/cicd-guide-v2.md`:

- On PRs: run unit, handler, and CDK template tests (+ lint, typecheck). Skip E2E unless explicitly enabled with secure inputs.
- On main/nightly: optionally run E2E against a stable environment, injecting tokens at runtime (never commit secrets).

## Running Locally

- Unit/handler/CDK tests: `npm test`
- E2E: follow the E2E guide to prepare `outputs.json` and `.e2e/` token files, then `npm run test:e2e`

## Mutation Testing

Follow `guides/testing/mutation-testing.md`.

## Test Coverage

Follow `guides/testing/test-coverage.md`.

## Notes

- Add new tests close to the feature being developed. Keep them fast and deterministic.
- Prefer clear, minimal assertions that protect key behaviors and contracts.
