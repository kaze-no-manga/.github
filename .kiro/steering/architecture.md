# Architecture Principles

## Multi-Repository Structure

Kaze no Manga uses a **multi-repo architecture** with independent repositories connected via NPM packages.

### Repository Organization

```
kaze-no-manga/ (GitHub Organization)
├── brand/          → @kaze-no-manga/brand (NPM public)
├── models/         → @kaze-no-manga/models (NPM public)
├── scraper/        → @kaze-no-manga/scraper (NPM public)
├── database/       → Internal (Drizzle schema + migrations)
├── backend/        → AWS CDK + Lambda (AppSync GraphQL)
├── web/            → React Router v7 (Lambda@Edge SSR)
├── mobile/         → Flutter (iOS + Android)
├── mcp-server/     → Experimental (MCP for LLMs)
└── telegram-bot/   → Experimental (Telegram integration)
```

### Dependency Flow

```
brand (design tokens)
  ↓
models (types + schemas + GraphQL)
  ↓
scraper (manga sources) + database (schema)
  ↓
backend (GraphQL API + Lambda)
  ↓
web (React Router v7) + mobile (Flutter)
```

**Rules:**
- Lower layers MUST NOT depend on upper layers
- Each package MUST have a clear, single responsibility
- Breaking changes MUST follow semantic versioning

## NPM Package Strategy

### Public Packages on NPM

All shared packages are published as **public packages** on NPM:
- `@kaze-no-manga/brand` - Design tokens + Tailwind preset
- `@kaze-no-manga/models` - TypeScript types + Zod schemas + GraphQL SDL
- `@kaze-no-manga/scraper` - Manga source scrapers with common interface

### Package Structure

Every NPM package MUST follow this structure:

```
package-name/
├── src/              # Source code (TypeScript)
├── dist/             # Compiled output (gitignored)
├── package.json      # Package manifest
├── tsconfig.json     # TypeScript config
├── README.md         # Package documentation
└── .npmignore        # NPM publish exclusions
```

### Package.json Requirements

```json
{
  "name": "@kaze-no-manga/package-name",
  "version": "0.1.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "prepublishOnly": "npm run build && npm test"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/kaze-no-manga/package-name"
  },
  "license": "MIT"
}
```

### Versioning Strategy

Follow **semantic versioning** (semver):
- `MAJOR.MINOR.PATCH` (e.g., `1.2.3`)
- **MAJOR**: Breaking changes (incompatible API changes)
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

**Pre-release versions** during MVP:
- `0.x.y` - API is not stable yet
- Breaking changes allowed in MINOR version during `0.x`

### Publishing Workflow

1. Make changes in package
2. Update version: `npm version patch|minor|major`
3. Run tests: `npm test`
4. Build: `npm run build`
5. Publish: `npm publish --access public`
6. Tag release: `git tag v0.1.0 && git push --tags`

**Automated via GitHub Actions:**
- On push to `main` with version change → auto-publish to NPM

## AWS Infrastructure Architecture

### High-Level Components

```
CloudFront CDN
  ├─→ Lambda@Edge (React Router SSR)
  └─→ S3 (manga images)
       ↓
AppSync GraphQL API
  ├─→ Lambda Resolvers (Node.js)
  ├─→ VTL Resolvers (simple queries)
  └─→ Direct Resolvers (DynamoDB)
       ↓
RDS Postgres + DynamoDB Cache
  ↓
SQS → Lambda (Scraper jobs)
  ↓
S3 (image storage)
```

### CDK Stack Organization

Each major component gets its own CDK stack:

```typescript
// backend/infrastructure/
├── stacks/
│   ├── AuthStack.ts          // Cognito + OAuth
│   ├── DatabaseStack.ts      // RDS + DynamoDB
│   ├── ApiStack.ts           // AppSync + Lambda resolvers
│   ├── StorageStack.ts       // S3 + CloudFront
│   ├── JobsStack.ts          // Step Functions + SQS
│   └── FrontendStack.ts      // Lambda@Edge + CloudFront
└── app.ts                    // CDK App entry point
```

**Cross-stack communication:**
- Use **SSM Parameters** for ARNs, URLs, IDs
- Use **CloudFormation Exports** for critical resources
- NEVER hardcode resource identifiers

### Resource Naming Convention

```
{project}-{environment}-{service}-{resource}
```

Examples:
- `kaze-prod-api-graphql`
- `kaze-prod-db-postgres`
- `kaze-prod-storage-images`

### Security Principles

1. **Least Privilege**: IAM roles with minimal permissions
2. **Encryption**: All data encrypted at rest (S3, RDS, DynamoDB)
3. **Secrets**: Use AWS Secrets Manager (never hardcode)
4. **VPC**: RDS in private subnets only
5. **HTTPS**: All traffic over TLS 1.2+

## Data Architecture

### Database Strategy: RDS + DynamoDB

**RDS Postgres (Primary):**
- Relational data: Users, Manga, Chapters, Library, History
- ACID transactions
- Complex queries with joins
- Drizzle ORM for type-safe queries

**DynamoDB (Cache):**
- Session data (fast reads)
- GraphQL query cache (TTL-based)
- User preferences (low latency)
- Direct AppSync resolvers (no Lambda overhead)

### Schema Ownership

- `database/` repo owns the **source of truth** for schema
- `models/` repo exports TypeScript types derived from schema
- Backend uses Drizzle schema directly
- Frontend uses types from `@kaze-no-manga/models`

### Migration Strategy

- Drizzle migrations in `database/migrations/`
- Applied via CDK Custom Resource (Lambda)
- Rollback support for failed migrations
- Never modify migrations after deployment

## GraphQL API Design

### Schema-First Approach

1. Define GraphQL SDL in `models/src/graphql/`
2. Generate TypeScript types via GraphQL Codegen
3. Implement resolvers in `backend/src/resolvers/`

### Resolver Types

**Lambda Resolvers** (complex logic):
```typescript
// backend/src/resolvers/addToLibrary.ts
export async function handler(event: AppSyncResolverEvent) {
  // Business logic
  // Database operations
  // External API calls
}
```

**VTL Resolvers** (simple queries):
```vtl
## Direct DynamoDB query (no Lambda)
{
  "version": "2018-05-29",
  "operation": "GetItem",
  "key": { "id": $util.dynamodb.toDynamoDBJson($ctx.args.id) }
}
```

**Direct Resolvers** (cache reads):
- AppSync → DynamoDB (no Lambda, lowest latency)

### Error Handling

All resolvers MUST return structured errors:

```typescript
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: { code: string; message: string } }
```

Error codes:
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Insufficient permissions
- `NOT_FOUND` - Resource doesn't exist
- `VALIDATION_ERROR` - Invalid input
- `INTERNAL_ERROR` - Server error

## Frontend Architecture

### React Router v7 (Web)

**Rendering Strategy:**
- **SSR** (Lambda@Edge): Homepage, manga detail, about
- **CSR** (Client-side): Library, settings, reader (auth required)

**Why Lambda@Edge?**
- Deploy React Router SSR to CloudFront edge locations
- Lower latency than centralized Lambda
- No vendor lock-in (vs Vercel)

**Route Organization:**
```typescript
// web/app/routes/
├── _index.tsx              // Homepage (SSR)
├── about.tsx               // About (SSR)
├── manga.$slug.tsx         // Manga detail (SSR)
├── manga.$slug.$chapter.tsx // Reader (CSR)
├── library.tsx             // Library (CSR, auth)
└── settings.tsx            // Settings (CSR, auth)
```

### Flutter (Mobile)

**Architecture:**
- Feature-based folder structure
- BLoC pattern for state management
- GraphQL client: `graphql_flutter`
- Offline-first with SQLite

**Platform Support:**
- iOS 13+
- Android 8.0+ (API 26+)

## Cross-Cutting Concerns

### Authentication Flow

1. User clicks "Login with Google"
2. Redirect to Cognito Hosted UI
3. Cognito handles OAuth with Google
4. Callback to `/auth/callback` with code
5. Exchange code for JWT tokens
6. Store tokens (httpOnly cookies for web, secure storage for mobile)
7. Include JWT in GraphQL requests (Authorization header)

### Image Handling

**Storage:**
- Original images → S3 bucket
- Optimized images → CloudFront CDN
- WebP conversion for modern browsers

**Flow:**
1. Scraper downloads image from source
2. Upload to S3: `s3://kaze-images/{mangaId}/{chapterId}/{page}.jpg`
3. CloudFront serves with cache headers
4. Lambda@Edge for on-the-fly optimization (future)

### Notification System

**Email (SES):**
- HTML templates in `backend/src/templates/`
- Transactional emails (new chapters, account)
- Batch sending for multiple chapters

**Push (SNS + FCM/APNS):**
- Web: FCM (Firebase Cloud Messaging)
- Android: FCM
- iOS: APNS (Apple Push Notification Service)
- User preferences stored in DynamoDB

## Development Principles

1. **Incremental**: Build in small, testable phases
2. **Type-Safe**: TypeScript strict mode everywhere
3. **Tested**: Unit + property-based tests for all logic
4. **Documented**: README in every repo
5. **Secure**: Security-first mindset
6. **Scalable**: Design for growth from day one
7. **Observable**: Logging and monitoring built-in

## Technology Constraints

**Required:**
- TypeScript 5.x+ (strict mode)
- Node.js 20.x+ (LTS)
- AWS CDK for infrastructure
- Drizzle ORM for database
- Vitest for testing
- fast-check for property-based testing

**Forbidden:**
- No `any` types (use `unknown` if needed)
- No client-side state libraries (Redux, Zustand) in web app
- No direct database access from frontend
- No hardcoded secrets or credentials
- No synchronous blocking operations in Lambda

## Performance Targets

- **Web**: First Contentful Paint < 1.5s
- **API**: GraphQL queries < 200ms (p95)
- **Reader**: Chapter load < 1s
- **Mobile**: App launch < 2s

## Monitoring & Observability

**Metrics:**
- CloudWatch for AWS resources
- Custom metrics for business logic
- Error tracking (CloudWatch Logs Insights)

**Logging:**
- Structured JSON logs
- Correlation IDs for request tracing
- Log levels: ERROR, WARN, INFO, DEBUG

**Alerts:**
- Lambda errors > 1% error rate
- API latency > 500ms (p95)
- Database connection pool exhaustion
- S3 upload failures
