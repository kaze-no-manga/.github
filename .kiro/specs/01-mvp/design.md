# Design Document - MVP Coordination

## Overview

This document defines the **organizational design** for delivering the Kaze no Manga MVP across multiple repositories. It establishes the development phases, dependency management, coordination patterns, and quality gates that ensure successful delivery.

## Architecture

### Multi-Repository Coordination Model

```
Phase 1: Foundation (Parallel Development)
┌─────────┐  ┌─────────┐  ┌──────────┐
│  brand  │  │ models  │  │ database │
│  (NPM)  │  │  (NPM)  │  │ (Schema) │
└────┬────┘  └────┬────┘  └────┬─────┘
     │            │             │
     └────────────┴─────────────┘
                  │
Phase 2: Backend (Sequential Development)
                  ▼
     ┌────────────────────────┐
     │       scraper          │
     │        (NPM)           │
     └───────────┬────────────┘
                 │
                 ▼
     ┌────────────────────────┐
     │       backend          │
     │   (AWS CDK + Lambda)   │
     └───────────┬────────────┘
                 │
Phase 3: Frontend (Depends on Backend)
                 ▼
     ┌────────────────────────┐
     │         web            │
     │  (React Router v7 SSR) │
     └────────────────────────┘
```

### Repository Dependency Graph

```
brand (0 deps)
  ↓
models (depends: brand)
  ↓
scraper (depends: models) + database (depends: models)
  ↓
backend (depends: models, scraper, database)
  ↓
web (depends: brand, models, backend API)
```

## Components and Interfaces

### Phase 1: Foundation

#### 1.1 Brand Package (`@kaze-no-manga/brand`)

**Purpose:** Provide design system and styling foundation

**Exports:**
```typescript
// Design tokens
export const colors = {
  primary: { ... },
  secondary: { ... },
  // ...
}

export const typography = {
  fontFamily: { ... },
  fontSize: { ... },
  // ...
}

// Tailwind preset
export const tailwindPreset = { ... }
```

**Deliverables:**
- Design tokens (JSON + TypeScript)
- Tailwind CSS preset
- Color palette (purple/lilac theme)
- Typography scale
- Spacing, border-radius, shadows
- Published to NPM as `@kaze-no-manga/brand@0.1.0`

**Spec Location:** `brand/.kiro/specs/01-mvp/`

---

#### 1.2 Models Package (`@kaze-no-manga/models`)

**Purpose:** Provide shared types, schemas, and GraphQL definitions

**Exports:**
```typescript
// GraphQL generated TypeScript types
export interface Manga {
  id: string
  slug: string
  title: string
  altTitles: string[]
  description?: string
  cover?: string
  status: MangaStatus
  genres: string[]
  authors: string[]
  year?: number
  totalChapters?: number
  isNsfw: boolean
  sourceName: string
  sourceId: string
  createdAt: string
  updatedAt: string
}

// GraphQL SDL
export const graphqlSchema = `
  type Manga {
    id: ID!
    slug: String!
    title: String!
    # ...
  }
`
```

**Deliverables:**
- GraphQL SDL files for all domains (User, Content, Library)
- TypeScript interfaces generated from GraphQL via Codegen
- Complete CRUD operations (queries and mutations)
- GraphQL Codegen configuration
- Published to NPM as `@kaze-no-manga/models@1.0.0`

**Dependencies:** None (GraphQL-first approach)

**Spec Location:** `models/.kiro/specs/01-mvp/`

---

#### 1.3 Database Package

**Purpose:** Provide database schema and migrations

**Exports:**
```typescript
// Drizzle schema
export const manga = pgTable('manga', {
  id: uuid('id').primaryKey(),
  title: text('title').notNull(),
  // ...
})

// Migration utilities
export { migrate } from './migrations'
```

**Deliverables:**
- Drizzle ORM schema definitions
- Migration files
- Seed data (optional)
- Database utilities
- NOT published to NPM (internal use only)

**Dependencies:** `@kaze-no-manga/models` (for type alignment)

**Spec Location:** `database/.kiro/specs/01-mvp/`

---

### Phase 2: Backend

#### 2.1 Scraper Package (`@kaze-no-manga/scraper`)

**Purpose:** Provide manga source scrapers with common interface

**Exports:**
```typescript
// Common interface
export interface MangaSource {
  search(query: string): Promise<Manga[]>
  getManga(id: string): Promise<MangaDetail>
  getChapters(mangaId: string): Promise<Chapter[]>
  getChapterImages(chapterId: string): Promise<string[]>
}

// Implementations
export class MangaParkScraper implements MangaSource { ... }
export class OmegaScansScraper implements MangaSource { ... }
```

**Deliverables:**
- Common `MangaSource` interface
- MangaPark scraper implementation
- OmegaScans scraper implementation
- Rate limiting decorator
- Retry logic wrapper
- Error handling
- Published to NPM as `@kaze-no-manga/scraper@0.1.0`

**Dependencies:** `@kaze-no-manga/models`

**Spec Location:** `scraper/.kiro/specs/01-mvp/`

---

#### 2.2 Backend (AWS Infrastructure + GraphQL API)

**Purpose:** Provide GraphQL API, authentication, and data persistence

**Components:**

**Infrastructure (AWS CDK):**
- AuthStack: Cognito User Pool + Google OAuth
- DatabaseStack: RDS PostgreSQL + DynamoDB
- ApiStack: AppSync GraphQL + Lambda resolvers
- StorageStack: S3 + CloudFront (for future image storage)

**GraphQL API:**
```graphql
type Query {
  searchManga(query: String!): [Manga!]!
  getManga(id: ID!): Manga
  getChapters(mangaId: ID!): [Chapter!]!
  getLibrary: [LibraryEntry!]!
}

type Mutation {
  addToLibrary(mangaId: ID!): LibraryEntry!
  updateProgress(mangaId: ID!, chapterId: ID!): LibraryEntry!
  removeFromLibrary(mangaId: ID!): Boolean!
}
```

**Lambda Resolvers:**
- `searchManga`: Search across sources using scraper package
- `getManga`: Get manga details from database or scraper
- `getChapters`: Get chapter list from database or scraper
- `addToLibrary`: Add manga to user's library
- `updateProgress`: Mark chapter as read
- `getLibrary`: Get user's library

**Deliverables:**
- CDK infrastructure code
- Lambda resolver functions
- Database migrations (via CDK Custom Resource)
- GraphQL schema
- Deployed to AWS
- API endpoint URL available

**Dependencies:** `@kaze-no-manga/models`, `@kaze-no-manga/scraper`, `database`

**Spec Location:** `backend/.kiro/specs/01-mvp/`

---

### Phase 3: Frontend

#### 3.1 Web Application (React Router v7)

**Purpose:** Provide user-facing web application

**Routes:**
- `/` - Homepage (SSR) - Search manga
- `/manga/:slug` - Manga detail (SSR) - View manga info + chapters
- `/manga/:slug/:chapter` - Reader (CSR) - Read chapter
- `/library` - Library (CSR, auth required) - View user's library
- `/settings` - Settings (CSR, auth required) - User preferences
- `/auth/callback` - OAuth callback

**Features:**
- Google OAuth authentication
- Manga search
- Manga detail pages
- Vertical scroll reader
- Library management
- Progress tracking (manual)

**Deliverables:**
- React Router v7 application
- Lambda@Edge SSR setup
- CloudFront distribution
- GraphQL client (TanStack Query)
- UI components (using brand package)
- Deployed to CloudFront
- Public URL available

**Dependencies:** `@kaze-no-manga/brand`, `@kaze-no-manga/models`, backend API

**Spec Location:** `web/.kiro/specs/01-mvp/`

---

## Data Models

### Core Entities

Defined in `@kaze-no-manga/models` and implemented in `database`:

```typescript
// User
interface User {
  id: string              // UUID
  email: string
  name: string | null
  avatar: string | null
  preferences: UserPreferences
  createdAt: string
  updatedAt: string
}

// Manga
interface Manga {
  id: string
  slug: string            // SEO-friendly URL slug
  title: string
  altTitles: string[]
  description: string | null
  cover: string | null    // Cover image URL
  status: 'ONGOING' | 'COMPLETED' | 'HIATUS' | 'CANCELLED'
  genres: string[]
  authors: string[]
  year: number | null
  totalChapters: number | null
  isNsfw: boolean
  sourceName: string      // 'mangapark', 'omegascans'
  sourceId: string        // ID in external source
  createdAt: string
  updatedAt: string
}

// Chapter
interface Chapter {
  id: string
  mangaId: string
  number: number          // Supports decimals: 5.1, 5.5
  title: string | null
  releaseDate: string | null
  images: ChapterImage[]
  createdAt: string
  updatedAt: string
}

// ChapterImage
interface ChapterImage {
  url: string             // S3 URL (optimized)
  originalUrl: string     // Original source URL
  page: number
  width: number | null
  height: number | null
}

// UserLibrary (LibraryEntry)
interface LibraryEntry {
  userId: string
  mangaId: string
  status: 'READING' | 'COMPLETED' | 'PLAN_TO_READ' | 'DROPPED' | 'ON_HOLD' | null
  rating: number | null   // 1-10
  notes: string | null
  currentChapterId: string | null
  lastReadAt: string | null
  createdAt: string
  updatedAt: string
}

// ReadingHistory
interface ReadingHistory {
  id: string
  userId: string
  mangaId: string
  chapterId: string
  chapterNumber: number
  completed: boolean
  readAt: string
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Package Dependency Consistency

*For any* repository that depends on an NPM package, the installed version must match or be compatible with the published version.

**Validates: Requirements 1.4, 4.2**

**Test Strategy:** Property-based test that generates random version combinations and verifies semver compatibility.

---

### Property 2: Phase Completion Ordering

*For any* phase N, all repositories in phase N-1 must be complete before phase N can start.

**Validates: Requirements 4.1, 4.2, 4.3**

**Test Strategy:** Manual verification via task list status. Each phase has a checkpoint task.

---

### Property 3: GraphQL Schema Consistency

*For any* GraphQL type defined in models package, the backend schema must include that type with matching fields.

**Validates: Requirements 2.2, 2.5**

**Test Strategy:** Property-based test that compares models package types with GraphQL schema.

---

### Property 4: Authentication Round-Trip

*For any* user who authenticates via Google OAuth, the system must create a user record and return a valid JWT token that can be used for subsequent requests.

**Validates: Requirements 2.3, 3.4**

**Test Strategy:** E2E test with mock OAuth flow.

---

### Property 5: Library Addition Persistence

*For any* manga added to a user's library, querying the library must return that manga.

**Validates: Requirements 3.5**

**Test Strategy:** Property-based test with random manga IDs.

---

### Property 6: Progress Tracking Idempotence

*For any* chapter marked as read, marking it as read again must not change the state or create duplicate history entries.

**Validates: Requirements 3.6**

**Test Strategy:** Property-based test that marks chapters as read multiple times.

---

### Property 7: Multi-Source Aggregation

*For any* manga with multiple sources, searching for that manga must return a single unified manga entity (not duplicates per source).

**Validates: Requirements 3.1**

**Test Strategy:** Property-based test with manga that exist in multiple sources.

---

### Property 8: Chapter Ordering Consistency

*For any* manga with chapters, the chapter list must be ordered by chapter number (supporting decimals: 5 → 5.1 → 5.5 → 6).

**Validates: Requirements 3.2**

**Test Strategy:** Property-based test with random chapter numbers including decimals.

---

### Property 9: Test Coverage Threshold

*For any* repository, the test coverage for critical paths must be at least 80%.

**Validates: Requirements 5.4**

**Test Strategy:** CI check that fails if coverage drops below 80%.

---

### Property 10: Deployment Rollback Safety

*For any* failed deployment, the system must automatically rollback to the previous working version.

**Validates: Requirements 7.5**

**Test Strategy:** Manual verification via deployment logs and CDK rollback behavior.

---

## Error Handling

### Package Publication Errors

**Scenario:** NPM publish fails due to version conflict

**Handling:**
1. Check if version already exists: `npm view @kaze-no-manga/package versions`
2. Bump version: `npm version patch`
3. Retry publish

**Recovery:** Manual intervention required

---

### Deployment Errors

**Scenario:** CDK deployment fails

**Handling:**
1. CDK automatically rolls back to previous stack state
2. CloudWatch logs capture error details
3. GitHub Actions workflow fails and notifies team

**Recovery:** Fix issue, push new commit, redeploy

---

### Cross-Repository Dependency Errors

**Scenario:** Repository A depends on package B, but B is not published yet

**Handling:**
1. Task list enforces dependency order
2. CI checks verify package availability before build
3. Clear error message: "Package @kaze-no-manga/models not found. Publish models package first."

**Recovery:** Complete dependency repository first

---

### Authentication Errors

**Scenario:** Google OAuth fails or Cognito is misconfigured

**Handling:**
1. User sees clear error message: "Authentication failed. Please try again."
2. Backend logs error details
3. CloudWatch alarm triggers if error rate > 5%

**Recovery:** Check Cognito configuration, verify Google OAuth credentials

---

## Testing Strategy

### Unit Tests (Vitest)

**Scope:** Individual functions, components, utilities

**Coverage Target:** 80% for critical paths

**Examples:**
- Package exports (brand, models, scraper)
- Lambda resolver logic
- React components
- Utility functions

---

### Property-Based Tests (fast-check)

**Scope:** Universal properties that must hold for all inputs

**Minimum Iterations:** 100 per property

**Examples:**
- Package version compatibility
- GraphQL schema consistency
- Chapter ordering
- Library operations (add, remove, update)

---

### Integration Tests

**Scope:** Cross-component interactions

**Examples:**
- Database migrations
- GraphQL resolver + database
- Scraper + external API (mocked)
- Authentication flow

---

### E2E Tests (Playwright)

**Scope:** Complete user flows

**Examples:**
- User authentication (Google OAuth)
- Search manga → View detail → Add to library
- Read chapter → Mark as read → Verify in library
- Cross-device sync (simulate multiple sessions)

---

### Infrastructure Tests (CDK Assertions)

**Scope:** AWS infrastructure correctness

**Examples:**
- Stack resources created correctly
- IAM permissions are least-privilege
- Encryption enabled on all resources
- Cross-stack references work

---

## Coordination Patterns

### Pattern 1: Dependency-Aware Development

**Problem:** Repositories have dependencies on each other

**Solution:**
1. Define clear dependency graph
2. Enforce development order via task list
3. Use checkpoints to verify dependencies are ready
4. Block dependent work until dependencies are complete

**Example:**
```
Task: 2.1 Create scraper package
  Prerequisites:
    - models package published (1.2)
    - models@0.1.0 available on NPM
```

---

### Pattern 2: Version Pinning for Internal Packages

**Problem:** Breaking changes in one package break dependent packages

**Solution:**
1. Use exact versions for internal dependencies: `"@kaze-no-manga/models": "0.1.0"`
2. Update dependents when publishing new versions
3. Test dependents before publishing
4. Communicate breaking changes

**Example:**
```json
// scraper/package.json
{
  "dependencies": {
    "@kaze-no-manga/models": "0.1.0"  // Exact version
  }
}
```

---

### Pattern 3: Parallel Development Within Phases

**Problem:** Some repositories can be developed in parallel

**Solution:**
1. Group repositories into phases
2. Within a phase, repositories can be developed in parallel
3. Phase completes when all repositories are done
4. Next phase starts only after previous phase completes

**Example:**
```
Phase 1 (Parallel):
  - brand (no dependencies)
  - models (depends on brand, but can start after brand structure is defined)
  - database (depends on models, but can start after models types are defined)
```

---

### Pattern 4: Checkpoint Tasks

**Problem:** Need to verify phase completion before proceeding

**Solution:**
1. Add checkpoint tasks at phase boundaries
2. Checkpoint verifies all dependencies are ready
3. Checkpoint includes manual verification steps
4. Cannot proceed past checkpoint until verified

**Example:**
```
- [ ] 1.4 Checkpoint: Foundation Phase Complete
  - Verify brand@0.1.0 published to NPM
  - Verify models@0.1.0 published to NPM
  - Verify database schema defined
  - All tests passing
```

---

### Pattern 5: Spec Hierarchy

**Problem:** Need both high-level coordination and detailed implementation specs

**Solution:**
1. Organization spec (`.github/.kiro/specs/01-mvp/`) defines WHAT and WHEN
2. Repository specs (`repo/.kiro/specs/01-mvp/`) define HOW
3. Organization spec references repository specs
4. Repository specs implement organization requirements

**Example:**
```
.github/specs/01-mvp/tasks.md:
  - [ ] 1.1 Create brand package
    - See brand/.kiro/specs/01-mvp/ for details

brand/specs/01-mvp/tasks.md:
  - [ ] 1.1.1 Define color tokens
  - [ ] 1.1.2 Define typography scale
  - [ ] 1.1.3 Create Tailwind preset
  ...
```

---

## Deployment Strategy

### NPM Packages (brand, models, scraper)

**Trigger:** Git tag push (`v0.1.0`)

**Pipeline:**
1. Run tests
2. Build package
3. Publish to NPM with `--access public`
4. Create GitHub release

**Rollback:** Publish previous version with higher version number

---

### Backend (AWS Infrastructure)

**Trigger:** Merge to `main` branch

**Pipeline:**
1. Run tests
2. CDK synth
3. CDK diff (review changes)
4. CDK deploy (with approval for production)
5. Run smoke tests

**Rollback:** CDK automatically rolls back on failure, or manual rollback via CDK

---

### Web Application (CloudFront + Lambda@Edge)

**Trigger:** Merge to `main` branch

**Pipeline:**
1. Run tests
2. Build application
3. Deploy to S3
4. Invalidate CloudFront cache
5. Deploy Lambda@Edge functions
6. Run E2E tests

**Rollback:** Redeploy previous version

---

## Monitoring & Observability

### Metrics

**Package Downloads:**
- Track NPM downloads for each package
- Alert if downloads drop significantly

**API Performance:**
- GraphQL query latency (p50, p95, p99)
- Lambda execution time
- Database query time

**User Activity:**
- Active users
- Searches per day
- Chapters read per day
- Library additions per day

**Error Rates:**
- Lambda errors
- GraphQL errors
- Authentication failures
- Scraper failures

---

### Logging

**Structured Logs:**
```json
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "backend",
  "function": "searchManga",
  "userId": "user-123",
  "error": "Scraper timeout",
  "duration": 5000
}
```

**Log Aggregation:**
- CloudWatch Logs for all Lambda functions
- CloudWatch Logs Insights for querying
- Correlation IDs for request tracing

---

### Alerts

**Critical Alerts (PagerDuty):**
- API down (5xx errors > 10%)
- Database connection failures
- Authentication service down

**Warning Alerts (Slack):**
- High latency (p95 > 500ms)
- Error rate increase (> 1%)
- Scraper failures (> 20%)

---

## Success Criteria

### Phase 1 Success Criteria

- [ ] brand@0.1.0 published to NPM
- [ ] models@0.1.0 published to NPM
- [ ] database schema defined and migrations created
- [ ] All packages have 80%+ test coverage
- [ ] All packages have comprehensive README

### Phase 2 Success Criteria

- [ ] scraper@0.1.0 published to NPM
- [ ] Backend deployed to AWS
- [ ] GraphQL API accessible via HTTPS
- [ ] Authentication working (Google OAuth)
- [ ] Database migrations applied
- [ ] All resolvers functional
- [ ] 80%+ test coverage

### Phase 3 Success Criteria

- [ ] Web app deployed to CloudFront
- [ ] All routes functional
- [ ] Authentication working
- [ ] Search, detail, reader, library pages working
- [ ] E2E tests passing
- [ ] 80%+ test coverage

### MVP Success Criteria

- [ ] All three phases complete
- [ ] Production deployment successful
- [ ] Monitoring and alerts configured
- [ ] Documentation complete
- [ ] Can handle 100 concurrent users
- [ ] All critical user flows working
- [ ] Ready for public launch

---

## Risk Mitigation

### Risk 1: Dependency Blocking

**Risk:** Repository A is blocked waiting for repository B

**Mitigation:**
- Clear dependency graph
- Parallel development where possible
- Mock dependencies during development
- Regular sync meetings

---

### Risk 2: Breaking Changes

**Risk:** Package update breaks dependent repositories

**Mitigation:**
- Semantic versioning
- Exact version pinning for internal packages
- Test dependents before publishing
- Changelog for all changes

---

### Risk 3: Deployment Failures

**Risk:** Deployment fails and breaks production

**Mitigation:**
- Automated rollback
- Smoke tests after deployment
- Blue/green deployment (future)
- Staging environment (future)

---

### Risk 4: External API Failures

**Risk:** Manga source APIs go down or change

**Mitigation:**
- Multiple sources (MangaPark + OmegaScans)
- Graceful degradation
- Caching
- Error handling and retries

---

### Risk 5: Scope Creep

**Risk:** MVP scope expands beyond original plan

**Mitigation:**
- Clear MVP definition
- Post-MVP backlog
- Regular scope reviews
- Focus on core features only
