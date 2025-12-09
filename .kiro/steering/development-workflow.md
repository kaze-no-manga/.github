# Development Workflow

## Git Workflow

### Branch Strategy

**Main Branch:**
- `main` - Production-ready code
- Protected branch (requires PR + reviews)
- All commits must pass CI

**Feature Branches:**
- `feature/feature-name` - New features
- `fix/bug-description` - Bug fixes
- `chore/task-description` - Maintenance tasks
- `docs/what-changed` - Documentation updates

**Example:**
```bash
# Create feature branch
git checkout -b feature/manga-search

# Work on feature
git add .
git commit -m "feat: add manga search functionality"

# Push to remote
git push origin feature/manga-search

# Create PR on GitHub
```

### Branch Naming

Format: `type/short-description`

**Types:**
- `feature/` - New features
- `fix/` - Bug fixes
- `chore/` - Maintenance (deps, config)
- `docs/` - Documentation
- `refactor/` - Code refactoring
- `test/` - Test additions/fixes

**Examples:**
- `feature/user-authentication`
- `fix/chapter-ordering-bug`
- `chore/update-dependencies`
- `docs/api-documentation`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

**Format:**
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `style` - Formatting (no code change)
- `refactor` - Code refactoring
- `test` - Adding tests
- `chore` - Maintenance

**Examples:**
```bash
# Feature
git commit -m "feat(library): add manga to library"

# Bug fix
git commit -m "fix(reader): correct chapter ordering"

# Breaking change
git commit -m "feat(api)!: change manga schema

BREAKING CHANGE: manga.author is now manga.authors (array)"

# With scope and body
git commit -m "feat(scraper): add MangaPark support

- Implement MangaPark scraper
- Add rate limiting
- Add error handling"
```

### Pull Request Process

1. **Create PR** from feature branch to `main`
2. **Fill PR template** with description, changes, testing
3. **Link issues** if applicable
4. **Request review** from team members
5. **Address feedback** and push changes
6. **Merge** once approved and CI passes

**PR Title Format:**
```
feat(scope): description
```

**PR Description Template:**
```markdown
## Description
Brief description of changes

## Changes
- Change 1
- Change 2

## Testing
- [ ] Unit tests added/updated
- [ ] Property tests added/updated
- [ ] Manual testing completed

## Screenshots (if applicable)
[Add screenshots]

## Related Issues
Closes #123
```

### Merge Strategy

**Squash and Merge** (preferred):
- Keeps history clean
- One commit per PR
- Easier to revert

**Merge Commit** (for large features):
- Preserves feature branch history
- Use for multi-commit features

**Rebase and Merge** (avoid):
- Can cause issues with shared branches

## Development Phases

### Phase 1: Foundation (Current)

**Repositories:**
1. `brand` - Design tokens + Tailwind preset
2. `models` - TypeScript types + Zod schemas + GraphQL SDL
3. `database` - Drizzle schema + migrations

**Order:**
1. Create `brand` package
2. Create `models` package (depends on `brand`)
3. Create `database` repo (uses `models`)

### Phase 2: Backend Core

**Repositories:**
1. `scraper` - Manga source scrapers (depends on `models`)
2. `backend` - GraphQL API + Lambda resolvers

**Order:**
1. Create `scraper` package
2. Create `backend` repo (uses `models`, `scraper`, `database`)

### Phase 3: Frontend

**Repositories:**
1. `web` - React Router v7 SSR

**Order:**
1. Create `web` repo (uses `brand`, `models`)
2. Integrate with `backend` GraphQL API

### Phase 4: Jobs & Notifications

**Repositories:**
1. `backend` - Add Step Functions, SQS, notifications

### Phase 5: Mobile

**Repositories:**
1. `mobile` - Flutter app

### Phase 6: Experiments

**Repositories:**
1. `mcp-server` - MCP for LLMs
2. `telegram-bot` - Telegram integration

## Local Development

### Initial Setup

```bash
# Clone repository
git clone https://github.com/kaze-no-manga/repo-name
cd repo-name

# Install dependencies
npm install

# Setup environment variables
cp .env.example .env
# Edit .env with your values

# Run tests
npm test

# Build
npm run build
```

### Development Loop

```bash
# Start development server (if applicable)
npm run dev

# Run tests in watch mode
npm run test:watch

# Type checking
npm run typecheck

# Linting
npm run lint

# Format code
npm run format
```

### Working with Local Packages

**Link local package:**
```bash
# In package directory (e.g., models)
npm link

# In consumer directory (e.g., backend)
npm link @kaze-no-manga/models
```

**Unlink:**
```bash
npm unlink @kaze-no-manga/models
npm install
```

### Environment Variables

**Never commit `.env` files!**

**Structure:**
```bash
# .env.example (committed)
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=your-api-key-here

# .env (gitignored)
DATABASE_URL=postgresql://real-user:real-pass@localhost:5432/real-db
API_KEY=actual-api-key
```

**Loading:**
```typescript
// Use dotenv or framework's built-in support
import 'dotenv/config'

const dbUrl = process.env.DATABASE_URL
if (!dbUrl) {
  throw new Error('DATABASE_URL is required')
}
```

## Testing Workflow

### Before Committing

```bash
# Run all tests
npm test

# Check types
npm run typecheck

# Lint code
npm run lint

# Format code
npm run format
```

### Pre-commit Hook (Optional)

```bash
# Install husky
npm install -D husky

# Setup pre-commit hook
npx husky install
npx husky add .husky/pre-commit "npm test"
```

### Test-Driven Development (TDD)

1. **Write test** (fails)
2. **Write code** (test passes)
3. **Refactor** (test still passes)

```bash
# Watch mode for TDD
npm run test:watch
```

## CI/CD Pipeline

### GitHub Actions

**Test Workflow:**
```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run typecheck
      - run: npm run lint
      - run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

**Deploy Workflow:**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - run: npm ci
      - run: npm run build
      - run: npm run deploy
```

**Publish Workflow:**
```yaml
# .github/workflows/publish.yml
name: Publish

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'
      
      - run: npm ci
      - run: npm test
      - run: npm run build
      
      - run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Status Checks

**Required checks before merge:**
- ✅ All tests pass
- ✅ Type checking passes
- ✅ Linting passes
- ✅ Coverage meets threshold (80%)
- ✅ PR approved by reviewer

## Code Review Guidelines

### For Authors

**Before requesting review:**
- [ ] All tests pass locally
- [ ] Code is formatted and linted
- [ ] PR description is clear
- [ ] Changes are focused (single responsibility)
- [ ] Documentation updated if needed

**During review:**
- Respond to feedback promptly
- Ask questions if feedback is unclear
- Make requested changes or discuss alternatives
- Mark conversations as resolved

### For Reviewers

**What to check:**
- [ ] Code follows conventions
- [ ] Tests are adequate
- [ ] No security issues
- [ ] Performance considerations
- [ ] Documentation is clear
- [ ] Changes align with requirements

**How to review:**
- Be constructive and respectful
- Explain reasoning for suggestions
- Approve when satisfied
- Request changes if needed

**Review comments:**
```markdown
# Suggestion
Consider using a more descriptive variable name here.

# Question
Why did you choose this approach over X?

# Blocking issue
This will cause a memory leak. Please fix before merging.

# Praise
Nice solution! This is much cleaner than the previous approach.
```

## Release Process

### For NPM Packages

1. **Update version**
   ```bash
   npm version patch  # or minor/major
   ```

2. **Update CHANGELOG.md**
   ```markdown
   ## [0.2.0] - 2024-01-15
   ### Added
   - New feature
   ```

3. **Commit and tag**
   ```bash
   git add .
   git commit -m "chore: release v0.2.0"
   git tag v0.2.0
   ```

4. **Push**
   ```bash
   git push && git push --tags
   ```

5. **Publish** (automated via GitHub Actions)

6. **Create GitHub Release**
   - Go to GitHub Releases
   - Create release from tag
   - Copy CHANGELOG entry

### For Applications (Backend, Web)

1. **Merge to main** (triggers deploy)
2. **Monitor deployment** (CloudWatch, logs)
3. **Verify in production**
4. **Rollback if needed**

## Hotfix Process

For critical bugs in production:

1. **Create hotfix branch** from `main`
   ```bash
   git checkout -b hotfix/critical-bug
   ```

2. **Fix bug** and add test

3. **Create PR** with `[HOTFIX]` prefix

4. **Fast-track review** (single reviewer)

5. **Merge and deploy** immediately

6. **Post-mortem** (document what happened)

## Documentation

### README.md

Every repository MUST have a README with:
- Project description
- Installation instructions
- Usage examples
- Development setup
- Testing instructions
- License

### Code Documentation

- JSDoc for public APIs
- Comments for complex logic
- Examples in documentation

### Architecture Decisions

Document major decisions in `.kiro/specs/` or `docs/`:
- Why this approach?
- What alternatives were considered?
- What are the trade-offs?

## Collaboration

### Communication

- **GitHub Issues** - Bug reports, feature requests
- **GitHub Discussions** - Questions, ideas
- **Pull Requests** - Code review, feedback
- **Commit messages** - What and why

### Issue Templates

**Bug Report:**
```markdown
## Description
Brief description of the bug

## Steps to Reproduce
1. Step 1
2. Step 2

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- OS: 
- Browser:
- Version:
```

**Feature Request:**
```markdown
## Problem
What problem does this solve?

## Proposed Solution
How should it work?

## Alternatives
Other approaches considered

## Additional Context
Any other information
```

## Best Practices

1. **Commit often** - Small, focused commits
2. **Test before pushing** - Catch issues early
3. **Keep PRs small** - Easier to review
4. **Write clear messages** - Help future you
5. **Document decisions** - Explain the "why"
6. **Review thoroughly** - Catch issues before merge
7. **Automate everything** - CI/CD, testing, deployment
8. **Monitor production** - Catch issues quickly
9. **Learn from mistakes** - Post-mortems for incidents
10. **Communicate clearly** - Keep team informed

## Troubleshooting

### Merge Conflicts

```bash
# Update your branch
git fetch origin
git rebase origin/main

# Resolve conflicts
# Edit conflicting files
git add .
git rebase --continue

# Force push (if already pushed)
git push --force-with-lease
```

### Failed CI

```bash
# Pull latest changes
git pull origin main

# Run tests locally
npm test

# Fix issues
# Commit and push
git add .
git commit -m "fix: resolve CI issues"
git push
```

### Broken Dependencies

```bash
# Clear cache
npm cache clean --force

# Delete and reinstall
rm -rf node_modules package-lock.json
npm install

# Check for updates
npm outdated
npm update
```

## Resources

- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)
