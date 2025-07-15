# SuperDeals Style Guide

This document outlines the coding standards, project structure, and best practices for the SuperDeals project. Following these guidelines ensures consistency, maintainability, and quality across the codebase.

## Table of Contents

1. [Project Structure](#project-structure)
2. [TypeScript Standards](#typescript-standards)
3. [AWS CDK Best Practices](#aws-cdk-best-practices)
4. [Testing Standards](#testing-standards)
5. [Environment Configuration](#environment-configuration)
6. [Documentation](#documentation)
7. [Git Workflow](#git-workflow)
8. [Security Guidelines](#security-guidelines)

## Project Structure

### Directory Layout

```
project-root/
├── src/                    # Source code
│   ├── constructs/         # CDK constructs
│   ├── constants/          # Configuration constants
│   ├── types/              # TypeScript type definitions
│   └── index.ts            # Entry point
├── test/                   # Test files
│   └── constructs/         # Tests for constructs
├── scripts/                # Utility scripts
│   ├── setup.sh            # Project setup script
│   └── setup-env.sh        # Environment setup script
├── .env.example            # Example environment variables
├── .npmrc                  # NPM configuration
├── package.json            # Project dependencies and scripts
├── tsconfig.json           # TypeScript configuration
└── README.md               # Project documentation
```

### Naming Conventions

- **Directories**: kebab-case (e.g., `network-construct`)
- **Files**: kebab-case (e.g., `eks-construct.ts`)
- **Classes**: PascalCase (e.g., `EksConstruct`)
- **Interfaces**: PascalCase with 'I' prefix (e.g., `IEksProps`)
- **Variables and Functions**: camelCase (e.g., `getClusterConfig`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DEFAULT_REGION`)

## TypeScript Standards

### Code Style

- Use TypeScript strict mode
- Prefer `interface` over `type` for object shapes
- Use explicit return types for functions
- Use `readonly` for immutable properties
- Prefer `const` over `let`
- Use optional chaining (`?.`) and nullish coalescing (`??`) where appropriate

### Imports

- Use absolute imports with path aliases (e.g., `#src/constants`)
- Group imports in the following order:
  1. External modules
  2. Internal modules
  3. Parent directories
  4. Sibling files

```typescript
// External modules
import { Construct } from "constructs";
import { Stack, CfnOutput } from "aws-cdk-lib";

// Internal modules with path aliases
import { DEFAULT_TAGS } from "#src/constants/default-tags";
```

## AWS CDK Best Practices

### Construct Design

- Keep constructs focused and single-purpose
- Use props interfaces for configuration
- Validate inputs in the constructor
- Tag all resources appropriately
- Export outputs for cross-stack references

### Resource Naming

- Use meaningful names for all resources
- Include environment name in resource names (e.g., `prod-cluster`)
- Use tags for additional metadata:
  - `Environment`: The deployment environment (e.g., dev, staging, prod)
  - `ManagedBy`: Indicates the infrastructure management tool (e.g., CDK)
  - `Project`: The project name (e.g., SuperDeals)

### Security

- Follow the principle of least privilege for IAM roles
- Enable encryption at rest and in transit
- Use AWS Secrets Manager or Parameter Store for sensitive data
- Enable VPC flow logs

## Testing Standards

### Test Structure

- Place test files next to the code they test with `.test.ts` suffix
- Use `describe` blocks to group related tests
- Each test should focus on a single behavior
- Follow the Arrange-Act-Assert pattern

### Test Naming

- Use descriptive test names that explain the expected behavior
- Start test names with "should" (e.g., "should create an EKS cluster")
- Group related tests in nested `describe` blocks

### Assertions

- Use the `@aws-cdk/assertions` library for CDK assertions
- Test both positive and negative cases
- Verify resource properties, counts, and relationships

## Environment Configuration

### Environment Variables

- Store all environment variables in `.env` file (not committed to version control)
- Provide a `.env.example` with placeholder values
- Document all required environment variables in the README
- Use descriptive, consistent variable names in UPPER_SNAKE_CASE

### AWS Profiles

- Use named profiles for AWS credentials
- Document required AWS permissions in the README
- Assume roles for production environments

## Documentation

### README.md

Every project should include a README.md with the following sections:

1. **Project Overview**: Brief description of the project
2. **Features**: Key features and functionality
3. **Prerequisites**: Required tools and dependencies
4. **Getting Started**: Setup and installation instructions
5. **Configuration**: Environment variables and settings
6. **Usage**: How to use the project
7. **Testing**: How to run tests
8. **Deployment**: Deployment instructions
9. **Contributing**: Guidelines for contributors

### Code Comments

- Use JSDoc for public APIs and complex logic
- Keep comments up-to-date with code changes
- Prefer self-documenting code over excessive comments

## Git Workflow

### Branching Strategy

- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/*`: Feature branches
- `bugfix/*`: Bug fix branches
- `release/*`: Release preparation branches

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code changes that neither fix bugs nor add features
- `test`: Adding or modifying tests
- `chore`: Changes to the build process or auxiliary tools

## Security Guidelines

### Secrets Management

- Never commit secrets to version control
- Use AWS Secrets Manager or Parameter Store for secrets
- Rotate credentials and secrets regularly

### Infrastructure Hardening

- Enable AWS Config rules for compliance monitoring
- Use AWS CloudTrail for auditing
- Implement network security best practices (security groups, NACLs)

### Dependencies

- Keep dependencies up to date
- Regularly audit for vulnerabilities
- Pin dependency versions for production

## Code Review Guidelines

### What to Look For

- Code correctness and functionality
- Adherence to style guide
- Proper error handling
- Test coverage
- Security considerations
- Performance implications

### Review Process

1. Create a pull request with a clear description
2. Request reviews from at least one team member
3. Address all review comments
4. Ensure all tests pass
5. Get approval before merging

## Continuous Integration/Deployment

### CI/CD Pipeline

- Run tests on every push
- Lint and format code
- Build and deploy on merge to main
- Use deployment environments (dev, staging, prod)

### Automated Testing

- Unit tests for all business logic
- Integration tests for AWS resources
- End-to-end tests for critical paths

## Monitoring and Logging

### Logging

- Use structured logging
- Include relevant context in log messages
- Set appropriate log levels

### Monitoring

- Set up CloudWatch Alarms for critical metrics
- Monitor error rates and latency
- Set up dashboards for key performance indicators

---

*Last Updated: July 10, 2025*
