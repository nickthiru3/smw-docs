# Shared CDK Constructs Initiative

This directory contains planning and tracking documents for the shared CDK constructs library initiative.

## Overview

The goal is to create a shared NPM package (`@thiru-ai-labs/cdk-constructs`) containing reusable CDK constructs that can be consumed by all microservice templates. This follows DRY principles and ensures consistency across services while simplifying maintenance.

## Documents

| Document                                                       | Purpose                                                   |
| -------------------------------------------------------------- | --------------------------------------------------------- |
| [01-shared-constructs-plan.md](./01-shared-constructs-plan.md) | Comprehensive plan for the shared constructs architecture |
| [02-implementation-tracker.md](./02-implementation-tracker.md) | Tracks progress, changes made, and remaining work         |

## Related Projects

| Project              | Description                                                       |
| -------------------- | ----------------------------------------------------------------- |
| `infra-contracts`    | Types-only package for SSM parameter contracts between services   |
| `svc-users-template` | Users/Auth microservice template (will consume shared constructs) |
| `svc-ai-template`    | AI/Agentic microservice template (will consume shared constructs) |
| `cdk-constructs`     | **NEW** - Shared CDK constructs library (to be created)           |

## Quick Links

- [Template Conversion Progress](../ai/03-template-conversion-progress.md) - Previous template work
- [Template Comparison](../ai/04-template-comparison.md) - Differences between templates
- [Technology Decisions](../../../../business/docs/ai-agentic-products-strategy/03-technical-stack/3.1-technology-decisions.md) - Architecture decisions

## Status

**Current Phase:** Planning & Documentation  
**Next Phase:** Implementation
