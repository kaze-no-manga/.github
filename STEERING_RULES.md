# Organization Steering Rules

This document explains the steering rules structure for the Kaze no Manga organization.

## What Are Steering Rules?

Steering rules are **organization-wide guidelines** that ensure consistency across all repositories. They define:
- Architecture patterns
- Code conventions
- Testing standards
- Development workflows
- Infrastructure patterns

## Location

All steering rules are in `.kiro/steering/`:

```
.kiro/steering/
├── README.md                    # Overview and guide
├── prompt.md                    # AI agent instructions
├── architecture.md              # Architecture principles
├── code-conventions.md          # Code style and conventions
├── testing-standards.md         # Testing strategy
├── package-management.md        # NPM package guidelines
├── development-workflow.md      # Git workflow and CI/CD
└── aws-infrastructure.md        # AWS CDK patterns
```

## How They Work

### For Developers

These rules are the **source of truth** for all development decisions:
- Read them before starting work
- Reference them during code reviews
- Follow them strictly (not suggestions)
- Suggest improvements via PR

### For AI Agents

AI agents (like Kiro) automatically:
- Load `prompt.md` as context
- Reference other files when making decisions
- Ensure all suggestions align with rules
- Cite specific rules when explaining decisions

### For Each Repository

Each repo **inherits** these organization-wide rules and can **extend** them:

```
repo/.kiro/steering/
├── repo-specific.md    # Additional rules for this repo
└── examples.md         # Code examples
```

**Precedence:**
1. Repository-specific rules (highest)
2. Organization-wide rules (this repo)
3. Framework conventions (lowest)

## Key Files

### 📋 prompt.md
AI agent role and responsibilities. Automatically loaded by Kiro.

### 🏗️ architecture.md
Multi-repo structure, NPM packages, AWS infrastructure, data models.

### 💻 code-conventions.md
TypeScript, React, Lambda, CDK, Drizzle ORM conventions.

### 🧪 testing-standards.md
Unit tests (Vitest), property tests (fast-check), E2E (Playwright).

### 📦 package-management.md
NPM publishing, versioning, dependencies.

### 🔄 development-workflow.md
Git workflow, conventional commits, CI/CD, releases.

### ☁️ aws-infrastructure.md
CDK patterns, Lambda, AppSync, RDS, DynamoDB, security.

## Benefits

1. **Consistency**: All repos follow the same patterns
2. **Quality**: Enforced best practices
3. **Onboarding**: New developers have clear guidelines
4. **AI Alignment**: AI agents follow the same rules
5. **Maintainability**: Easier to maintain multiple repos

## Next Steps

Now that steering rules are in place:

1. **Go to each repository** (brand, models, scraper, etc.)
2. **Create repository-specific specs** in `.kiro/specs/`
3. **Reference these steering rules** when implementing features
4. **Extend with repo-specific rules** if needed

## Example: Creating Specs in `brand` Repo

```bash
cd ../brand

# Create spec for design tokens
kiro spec create design-tokens

# The spec will reference:
# - .kiro/steering/architecture.md (NPM package structure)
# - .kiro/steering/code-conventions.md (TypeScript conventions)
# - .kiro/steering/testing-standards.md (Testing requirements)
# - .kiro/steering/package-management.md (Publishing workflow)
```

## Updating Rules

1. Open issue in `.github` repo
2. Discuss with team
3. Create PR with changes
4. Get approval
5. Merge and communicate

## Questions?

Check `.kiro/steering/README.md` for detailed information about each file.

---

**Created**: December 2024
**Purpose**: Establish organization-wide development standards
**Status**: ✅ Complete - Ready for repository-specific specs
