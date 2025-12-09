# Kaze no Manga - Organization Steering Rules

This directory contains the **organization-wide steering rules** for the Kaze no Manga project. These rules apply to all repositories and should be followed by all developers and AI agents.

## Overview

Kaze no Manga is a multi-repo project with NPM public packages. Each repository has its own specific implementation, but all follow these common guidelines.

## Steering Files

### 📋 [prompt.md](./prompt.md)
**AI Agent Instructions**

Defines the role and responsibilities of AI agents working on Kaze no Manga. This file is automatically included in AI agent context.

**Key Topics:**
- Agent role and responsibilities
- Project context and current phase
- Development principles
- How to use steering rules

---

### 🏗️ [architecture.md](./architecture.md)
**Architecture Principles & Patterns**

Comprehensive guide to the multi-repo architecture, NPM packages, AWS infrastructure, and data models.

**Key Topics:**
- Multi-repository structure and dependency flow
- NPM package strategy (public packages on NPM)
- Package structure and versioning (semver)
- AWS infrastructure architecture (AppSync, Lambda, RDS, DynamoDB)
- CDK stack organization
- Database strategy (RDS + DynamoDB)
- GraphQL API design
- Frontend architecture (React Router v7, Flutter)
- Cross-cutting concerns (auth, images, notifications)

**When to Reference:**
- Setting up a new repository
- Making architectural decisions
- Understanding cross-repo dependencies
- Designing AWS infrastructure
- Planning data models

---

### 💻 [code-conventions.md](./code-conventions.md)
**Code Style & Conventions**

Detailed coding standards for TypeScript, React, Lambda, CDK, and Drizzle ORM.

**Key Topics:**
- TypeScript strict mode and type definitions
- Naming conventions (variables, functions, components, files)
- Code organization and import order
- Function patterns (async/await, destructuring)
- Error handling (Result type pattern)
- Comments and documentation
- Minimal code principle
- React-specific conventions
- Lambda handler structure
- CDK stack patterns
- Drizzle ORM queries

**When to Reference:**
- Writing new code
- Reviewing pull requests
- Refactoring existing code
- Resolving style debates

---

### 🧪 [testing-standards.md](./testing-standards.md)
**Testing Strategy & Standards**

Comprehensive testing guide covering unit tests, property-based tests, integration tests, and E2E tests.

**Key Topics:**
- Dual testing approach (unit + property-based)
- Vitest for unit tests
- fast-check for property-based tests (minimum 100 iterations)
- Property patterns (round-trip, invariant, idempotence, metamorphic, error conditions)
- Custom generators for domain objects
- React component testing
- Lambda handler testing
- Integration tests (database, API)
- E2E tests with Playwright
- Coverage requirements (80% for critical paths)
- CI/CD integration

**When to Reference:**
- Writing tests for new features
- Implementing correctness properties
- Setting up test infrastructure
- Debugging test failures
- Configuring CI/CD pipelines

---

### 📦 [package-management.md](./package-management.md)
**NPM Package Management**

Guide to publishing and managing NPM packages in the organization.

**Key Topics:**
- NPM public packages strategy
- Package structure and configuration
- Semantic versioning (semver)
- Publishing workflow (manual and automated)
- Dependency management (dependencies, devDependencies, peerDependencies)
- Cross-package dependencies
- Package documentation (README, CHANGELOG)
- NPM scripts
- TypeScript configuration
- Package size optimization
- Security and auditing

**When to Reference:**
- Creating a new NPM package
- Publishing a new version
- Managing dependencies
- Updating cross-package dependencies
- Troubleshooting package issues

---

### 🔄 [development-workflow.md](./development-workflow.md)
**Git Workflow & Development Process**

Complete guide to the development workflow, from branching to deployment.

**Key Topics:**
- Git workflow (main branch, feature branches)
- Branch naming conventions
- Conventional commits
- Pull request process
- Code review guidelines
- Development phases (Foundation, Backend, Frontend, Jobs, Mobile, Experiments)
- Local development setup
- Testing workflow
- CI/CD pipeline (GitHub Actions)
- Release process (NPM packages and applications)
- Hotfix process
- Documentation standards

**When to Reference:**
- Starting a new feature
- Creating a pull request
- Reviewing code
- Setting up local development
- Deploying to production
- Handling hotfixes

---

### ☁️ [aws-infrastructure.md](./aws-infrastructure.md)
**AWS Infrastructure Guidelines**

Detailed guide to AWS CDK and infrastructure patterns.

**Key Topics:**
- Infrastructure as Code with AWS CDK (TypeScript)
- CDK project structure
- Stack organization (Auth, Database, API, Storage, Jobs, Frontend)
- Resource naming conventions
- Cross-stack communication (SSM, CloudFormation exports)
- Lambda functions (structure, best practices, permissions)
- AppSync GraphQL API (Lambda resolvers, VTL resolvers)
- RDS PostgreSQL (cluster, migrations)
- DynamoDB (tables, GSI, TTL)
- S3 & CloudFront (buckets, distributions)
- Cognito (user pools, OAuth)
- Step Functions (state machines, EventBridge)
- SQS (queues, DLQ)
- Secrets management
- Monitoring & logging (CloudWatch, alarms)
- Security best practices (IAM, encryption, VPC)
- Cost optimization
- Testing infrastructure (CDK assertions)

**When to Reference:**
- Creating AWS infrastructure
- Writing CDK code
- Configuring Lambda functions
- Setting up AppSync resolvers
- Managing databases
- Implementing security
- Optimizing costs
- Troubleshooting deployments

---

## How to Use These Rules

### For Developers

1. **Read First**: Start with `architecture.md` to understand the big picture
2. **Reference Often**: Keep these files open while coding
3. **Follow Strictly**: These are not suggestions, they are requirements
4. **Suggest Improvements**: If you find issues, open a PR to update the rules

### For AI Agents

1. **Context**: `prompt.md` is automatically included in your context
2. **Reference**: Always check relevant steering files before making decisions
3. **Cite**: When suggesting solutions, reference the specific steering file
4. **Consistency**: Ensure all suggestions align with these rules

### For Code Reviews

Use these files as the source of truth:
- ✅ "This follows the pattern in `code-conventions.md`"
- ❌ "This violates the naming convention in `code-conventions.md`"
- 💡 "Consider the approach suggested in `testing-standards.md`"

## Repository-Specific Rules

Each repository can extend these rules with its own `.kiro/steering/` files:

```
repo/.kiro/steering/
├── repo-specific-rules.md    # Additional rules for this repo
└── examples.md                # Code examples specific to this repo
```

**Precedence:**
1. Repository-specific rules (highest priority)
2. Organization-wide rules (this directory)
3. Framework/library conventions (lowest priority)

## Updating Steering Rules

### When to Update

- New patterns emerge
- Technology changes
- Team consensus on improvements
- Lessons learned from incidents

### How to Update

1. Open an issue in `.github` repo
2. Discuss with team
3. Create PR with changes
4. Get approval from maintainers
5. Merge and communicate changes

### Versioning

Steering rules are **living documents** and don't have versions. Changes are tracked via Git history.

## Quick Reference

| Need to... | Check... |
|------------|----------|
| Understand the architecture | `architecture.md` |
| Write TypeScript code | `code-conventions.md` |
| Write tests | `testing-standards.md` |
| Publish an NPM package | `package-management.md` |
| Create a pull request | `development-workflow.md` |
| Deploy AWS infrastructure | `aws-infrastructure.md` |
| Configure AI agent | `prompt.md` |

## Contributing

Found an issue or have a suggestion? Open an issue or PR in the `.github` repository.

---

**Last Updated**: December 2024
**Maintained by**: Kaze no Manga Team
