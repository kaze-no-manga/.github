# Code Conventions

## TypeScript

### Strict Mode

Always use TypeScript strict mode. No `any` types unless absolutely necessary.

```typescript
// ❌ Bad
function processData(data: any) { }

// ✅ Good
function processData(data: MangaData) { }

// ✅ Acceptable when truly unknown
function handleError(error: unknown) {
  if (error instanceof Error) {
    console.error(error.message)
  }
}
```

### Type Definitions

- Use `interface` for object shapes that can be extended
- Use `type` for unions, intersections, and utilities
- Export types that are used across files
- Co-locate types with their usage when possible

```typescript
// Interfaces for object shapes
interface User {
  id: string
  email: string
  name: string | null
}

// Types for unions and complex types
type MangaSource = 'mangapark' | 'omegascans'
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string }

// Utility types
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>
```

### Null vs Undefined

- Use `null` for intentional absence of value
- Use `undefined` for uninitialized or missing values
- Be consistent within a module

```typescript
// ✅ Good - consistent use
interface User {
  name: string
  bio: string | null        // Explicitly no bio
  avatar?: string           // Optional field
}
```

## Naming Conventions

### Variables and Functions

```typescript
// camelCase for variables and functions
const mangaList = []
const userPreferences = {}

function getMangaById(id: string) { }
function calculateReadingProgress(chapters: Chapter[]) { }
```

### Types and Interfaces

```typescript
// PascalCase for types and interfaces
interface MangaData { }
type ChapterStatus = 'read' | 'unread'
```

### Constants

```typescript
// UPPER_SNAKE_CASE for true constants
const MAX_CHAPTERS_PER_PAGE = 50
const API_TIMEOUT_MS = 5000
const DEFAULT_THEME = 'light' // Not a constant, use camelCase

// Enums (avoid when possible, use union types instead)
enum Status {
  Active = 'ACTIVE',
  Inactive = 'INACTIVE',
}
```

### Files and Folders

```typescript
// kebab-case for files and folders
manga-card.tsx
user-library.ts
chapter-list/

// Exception: React components can use PascalCase
MangaCard.tsx
UserLibrary.tsx
```

### Database

```typescript
// snake_case for tables and columns (SQL convention)
const users = pgTable('users', {
  id: uuid('id').primaryKey(),
  created_at: timestamp('created_at').notNull(),
  updated_at: timestamp('updated_at').notNull(),
})
```

### AWS Resources

```typescript
// kebab-case for resource names
const bucket = new s3.Bucket(this, 'MangaImages', {
  bucketName: 'kaze-prod-manga-images',
})

const lambda = new lambda.Function(this, 'ScraperFunction', {
  functionName: 'kaze-prod-scraper-handler',
})
```

## Code Organization

### Import Order

1. External dependencies (node_modules)
2. Internal absolute imports (from packages)
3. Internal relative imports (same repo)
4. Type imports (separate)

```typescript
// 1. External dependencies
import { useState } from 'react'
import { eq } from 'drizzle-orm'

// 2. Internal absolute (from packages)
import { MangaSchema } from '@kaze-no-manga/models'
import { colors } from '@kaze-no-manga/brand'

// 3. Internal relative
import { MangaCard } from './MangaCard'
import { useLibrary } from '../hooks/useLibrary'

// 4. Type imports (if not inline)
import type { Manga, Chapter } from '@kaze-no-manga/models'
```

### File Structure

```typescript
// 1. Imports
import { type ReactNode } from 'react'

// 2. Types and interfaces
interface Props {
  children: ReactNode
}

// 3. Constants
const MAX_ITEMS = 10

// 4. Main code (functions, components, classes)
export function Component({ children }: Props) {
  return <div>{children}</div>
}

// 5. Helper functions (if not exported)
function helperFunction() { }
```

## Functions

### Function Declarations vs Expressions

```typescript
// ✅ Prefer function declarations for top-level functions
export function getManga(id: string) {
  return db.query.manga.findFirst({ where: eq(manga.id, id) })
}

// ✅ Use arrow functions for callbacks and inline functions
const filtered = manga.filter(m => m.status === 'ongoing')

// ✅ Use arrow functions for React components (consistency)
export const MangaCard = ({ manga }: Props) => {
  return <div>{manga.title}</div>
}
```

### Async/Await

Always use `async/await` over `.then()` chains.

```typescript
// ❌ Bad
function getManga(id: string) {
  return db.query.manga.findFirst({ where: eq(manga.id, id) })
    .then(manga => manga)
    .catch(error => null)
}

// ✅ Good
async function getManga(id: string) {
  try {
    return await db.query.manga.findFirst({ where: eq(manga.id, id) })
  } catch (error) {
    console.error('Failed to get manga:', error)
    return null
  }
}
```

### Parameter Destructuring

Use destructuring for objects with multiple properties.

```typescript
// ❌ Bad
function createManga(manga: CreateMangaInput) {
  return {
    title: manga.title,
    author: manga.author,
    year: manga.year,
  }
}

// ✅ Good
function createManga({ title, author, year }: CreateMangaInput) {
  return { title, author, year }
}
```

## Error Handling

### Result Type Pattern

Use discriminated unions for operations that can fail.

```typescript
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: string }

async function addManga(data: MangaInput): Promise<Result<Manga>> {
  try {
    const manga = await db.insert(mangaTable).values(data)
    return { success: true, data: manga }
  } catch (error) {
    console.error('Failed to add manga:', error)
    return { success: false, error: 'Failed to add manga' }
  }
}

// Usage
const result = await addManga(data)
if (!result.success) {
  console.error(result.error)
  return
}
console.log(result.data) // TypeScript knows this is Manga
```

### Error Messages

- User-facing: Clear, actionable, non-technical
- Logs: Detailed, include context, stack traces

```typescript
// ❌ Bad
throw new Error('err')

// ✅ Good - user-facing
return { success: false, error: 'Failed to add manga. Please try again.' }

// ✅ Good - logging
console.error('Failed to add manga:', {
  mangaId: data.id,
  error: error instanceof Error ? error.message : 'Unknown error',
  stack: error instanceof Error ? error.stack : undefined,
})
```

### Try-Catch Scope

Keep try-catch blocks small and focused.

```typescript
// ❌ Bad - too broad
try {
  const user = await getUser()
  const manga = await getManga()
  const library = await getLibrary()
  return { user, manga, library }
} catch (error) {
  // Which operation failed?
}

// ✅ Good - specific
const user = await getUser() // Let it throw if critical
try {
  const manga = await getManga()
  return { user, manga }
} catch (error) {
  console.error('Failed to get manga:', error)
  return { user, manga: null }
}
```

## Comments

### When to Comment

- Explain **why**, not **what**
- Document complex algorithms
- Add context for non-obvious decisions
- Use JSDoc for public APIs

```typescript
// ❌ Bad - obvious
// Get the user's library
const library = await getUserLibrary()

// ✅ Good - explains why
// We fetch chapters in batches to avoid rate limiting from the source
const chapters = await fetchChaptersInBatches(mangaId, BATCH_SIZE)

// ✅ Good - documents public API
/**
 * Searches for manga by title across all sources.
 * 
 * @param query - Search query (minimum 2 characters)
 * @param sources - Optional list of sources to search (default: all)
 * @returns Array of manga matching the query
 * @throws {ValidationError} If query is too short
 */
export async function searchManga(
  query: string,
  sources?: MangaSource[]
): Promise<Manga[]> {
  // ...
}
```

### TODO Comments

Use TODO comments sparingly and include context.

```typescript
// ❌ Bad
// TODO: fix this

// ✅ Good
// TODO(username): Implement caching for manga search results
// See issue #123 for discussion on cache invalidation strategy
```

## Minimal Code Principle

Write only the code needed to satisfy requirements. Avoid:
- Premature abstractions
- Unused utilities
- Over-engineered solutions
- Verbose implementations

```typescript
// ❌ Bad - over-engineered
class MangaRepository {
  constructor(private db: Database) {}
  
  async findById(id: string) { }
  async findAll() { }
  async findByTitle(title: string) { }
  async findByAuthor(author: string) { }
  async create(data: CreateMangaDto) { }
  async update(id: string, data: UpdateMangaDto) { }
  async delete(id: string) { }
  // ... 20 more methods
}

// ✅ Good - minimal, direct
export async function getManga(id: string) {
  return db.query.manga.findFirst({ where: eq(manga.id, id) })
}

export async function searchManga(query: string) {
  return db.query.manga.findMany({ 
    where: like(manga.title, `%${query}%`) 
  })
}
```

## React-Specific Conventions

### Component Structure

```typescript
// 1. Imports
import { type ReactNode } from 'react'
import { Button } from '@/components/ui/button'

// 2. Types
interface Props {
  children: ReactNode
  onClose: () => void
}

// 3. Component
export function Modal({ children, onClose }: Props) {
  return (
    <div className="modal">
      {children}
      <Button onClick={onClose}>Close</Button>
    </div>
  )
}
```

### Hooks

```typescript
// Custom hooks start with 'use'
export function useLibrary() {
  const [manga, setManga] = useState<Manga[]>([])
  
  useEffect(() => {
    loadLibrary().then(setManga)
  }, [])
  
  return { manga }
}
```

### Props Naming

```typescript
// ✅ Good - clear, semantic names
interface MangaCardProps {
  manga: Manga
  onSelect: (id: string) => void
  isSelected: boolean
}

// ❌ Bad - vague names
interface Props {
  data: any
  onClick: () => void
  active: boolean
}
```

## Lambda-Specific Conventions

### Handler Structure

```typescript
import type { AppSyncResolverEvent } from 'aws-lambda'

export async function handler(event: AppSyncResolverEvent<Args>) {
  // 1. Extract arguments
  const { mangaId } = event.arguments
  
  // 2. Validate input
  if (!mangaId) {
    return { success: false, error: 'Missing mangaId' }
  }
  
  // 3. Execute logic
  try {
    const manga = await getManga(mangaId)
    return { success: true, data: manga }
  } catch (error) {
    console.error('Error:', error)
    return { success: false, error: 'Internal error' }
  }
}
```

### Environment Variables

```typescript
// ✅ Good - validate at startup
const DB_HOST = process.env.DB_HOST
if (!DB_HOST) {
  throw new Error('DB_HOST environment variable is required')
}

// ❌ Bad - no validation
const dbHost = process.env.DB_HOST // might be undefined
```

## CDK-Specific Conventions

### Stack Structure

```typescript
import * as cdk from 'aws-cdk-lib'
import { Construct } from 'constructs'

export class ApiStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props)
    
    // 1. Import cross-stack resources
    const dbArn = cdk.Fn.importValue('DatabaseArn')
    
    // 2. Create resources
    const api = new appsync.GraphqlApi(this, 'Api', {
      name: 'kaze-api',
    })
    
    // 3. Export resources for other stacks
    new cdk.CfnOutput(this, 'ApiUrl', {
      value: api.graphqlUrl,
      exportName: 'ApiUrl',
    })
  }
}
```

## Drizzle ORM Conventions

### Schema Definition

```typescript
import { pgTable, uuid, text, timestamp } from 'drizzle-orm/pg-core'

export const manga = pgTable('manga', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: text('title').notNull(),
  slug: text('slug').notNull().unique(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})
```

### Queries

```typescript
// ✅ Good - type-safe queries
const result = await db
  .select()
  .from(manga)
  .where(eq(manga.id, mangaId))
  .limit(1)

// ✅ Good - use query builder
const result = await db.query.manga.findFirst({
  where: eq(manga.id, mangaId),
  with: {
    chapters: true,
  },
})
```

## General Best Practices

1. **DRY (Don't Repeat Yourself)** - but don't abstract too early
2. **KISS (Keep It Simple, Stupid)** - simple solutions over clever ones
3. **YAGNI (You Aren't Gonna Need It)** - don't build for hypothetical futures
4. **Single Responsibility** - each function/module does one thing well
5. **Explicit over Implicit** - be clear about intent
6. **Fail Fast** - validate early, throw errors for invalid states
7. **Immutability** - prefer `const` over `let`, avoid mutations

## Code Review Checklist

- [ ] TypeScript strict mode enabled
- [ ] No `any` types (unless justified)
- [ ] Proper error handling
- [ ] Clear variable and function names
- [ ] Comments explain "why", not "what"
- [ ] Tests included for new functionality
- [ ] No hardcoded secrets or credentials
- [ ] Follows naming conventions
- [ ] Minimal code (no premature abstractions)
- [ ] Type-safe database queries
