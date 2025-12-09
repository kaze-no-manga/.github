# Testing Standards

## Dual Testing Approach

Every feature MUST be validated with both unit tests and property-based tests.

### Why Both?

- **Unit tests**: Verify specific examples, edge cases, and integration points
- **Property tests**: Verify universal properties that must hold for all inputs
- **Together**: Unit tests catch concrete bugs, property tests verify general correctness

## Unit Tests

### Framework: Vitest

**Why Vitest?**
- Fast (native ESM, parallel execution)
- Compatible with Jest API
- Built-in TypeScript support
- Great DX (watch mode, UI)

### Test Location

Co-locate tests with source files:

```
src/
├── manga/
│   ├── getManga.ts
│   ├── getManga.test.ts          # Unit tests
│   └── getManga.property.test.ts # Property tests
```

### Test Structure

```typescript
import { describe, it, expect, beforeEach, afterEach } from 'vitest'
import { getManga } from './getManga'

describe('getManga', () => {
  beforeEach(() => {
    // Setup
  })

  afterEach(() => {
    // Cleanup
  })

  describe('when manga exists', () => {
    it('should return manga with all fields', async () => {
      // Arrange
      const mangaId = 'test-id'
      
      // Act
      const result = await getManga(mangaId)
      
      // Assert
      expect(result).toBeDefined()
      expect(result.id).toBe(mangaId)
      expect(result.title).toBeTruthy()
    })
  })

  describe('when manga does not exist', () => {
    it('should return null', async () => {
      const result = await getManga('non-existent')
      expect(result).toBeNull()
    })
  })
})
```

### What to Test

**DO test:**
- ✅ Business logic
- ✅ Data transformations
- ✅ Error handling
- ✅ Edge cases (empty, null, boundary values)
- ✅ Integration points (API calls, database)
- ✅ User interactions (React components)

**DON'T test:**
- ❌ Third-party libraries
- ❌ Framework internals
- ❌ Trivial getters/setters
- ❌ Implementation details

### Mocking Strategy

**Mock external dependencies:**
```typescript
import { vi } from 'vitest'
import { fetchFromSource } from './api'

vi.mock('./api', () => ({
  fetchFromSource: vi.fn(),
}))

it('should handle API errors', async () => {
  vi.mocked(fetchFromSource).mockRejectedValue(new Error('API error'))
  
  const result = await getManga('id')
  
  expect(result.success).toBe(false)
  expect(result.error).toBe('Failed to fetch manga')
})
```

**Use real implementations for:**
- Internal modules
- Database (use test database or in-memory)
- Simple utilities

### React Component Tests

Use React Testing Library:

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { MangaCard } from './MangaCard'

describe('MangaCard', () => {
  const manga = {
    id: '1',
    title: 'Test Manga',
    coverUrl: 'https://example.com/cover.jpg',
  }

  it('should render manga title', () => {
    render(<MangaCard manga={manga} />)
    expect(screen.getByText('Test Manga')).toBeInTheDocument()
  })

  it('should call onSelect when clicked', () => {
    const onSelect = vi.fn()
    render(<MangaCard manga={manga} onSelect={onSelect} />)
    
    fireEvent.click(screen.getByRole('button'))
    
    expect(onSelect).toHaveBeenCalledWith('1')
  })
})
```

### Lambda Handler Tests

```typescript
import type { AppSyncResolverEvent } from 'aws-lambda'
import { handler } from './addToLibrary'

describe('addToLibrary handler', () => {
  it('should add manga to library', async () => {
    const event: AppSyncResolverEvent<{ mangaId: string }> = {
      arguments: { mangaId: 'manga-1' },
      identity: { sub: 'user-1' },
      // ... other required fields
    }

    const result = await handler(event)

    expect(result.success).toBe(true)
    expect(result.data.mangaId).toBe('manga-1')
  })

  it('should return error for missing mangaId', async () => {
    const event: AppSyncResolverEvent<{}> = {
      arguments: {},
      identity: { sub: 'user-1' },
    }

    const result = await handler(event)

    expect(result.success).toBe(false)
    expect(result.error).toContain('mangaId')
  })
})
```

## Property-Based Tests

### Framework: fast-check

**Why fast-check?**
- Powerful generators for complex data
- Shrinking (finds minimal failing case)
- Replay support (reproduce failures)
- TypeScript-first

### Minimum Iterations

Every property test MUST run at least **100 iterations**.

```typescript
fc.assert(
  fc.property(/* ... */),
  { numRuns: 100 } // Minimum
)
```

### Property Test Structure

```typescript
import { test } from 'vitest'
import * as fc from 'fast-check'
import { addManga, getManga } from './manga'

// **Feature: manga-library, Property 1: Manga addition persistence**
test('added manga can be retrieved with same data', async () => {
  await fc.assert(
    fc.property(
      fc.record({
        title: fc.string({ minLength: 1, maxLength: 200 }),
        author: fc.string({ minLength: 1, maxLength: 100 }),
        year: fc.integer({ min: 1900, max: 2100 }),
      }),
      async (mangaData) => {
        // Add manga
        const added = await addManga(mangaData)
        
        // Retrieve manga
        const retrieved = await getManga(added.id)
        
        // Property: retrieved data matches added data
        expect(retrieved).toEqual(added)
      }
    ),
    { numRuns: 100 }
  )
})
```

### Tagging Convention

Each property test MUST include a comment referencing the design document:

```typescript
// **Feature: {feature-name}, Property {number}: {property-description}**
// **Validates: Requirements {requirement-id}**
```

Example:
```typescript
// **Feature: manga-tracker, Property 3: Chapter ordering consistency**
// **Validates: Requirements 2.1, 2.3**
test('chapters are always ordered correctly', async () => {
  // ...
})
```

### Common Property Patterns

#### 1. Round-Trip Properties

**Pattern**: `decode(encode(x)) === x`

```typescript
// **Property: Serialization round-trip**
test('manga serialization preserves data', () => {
  fc.assert(
    fc.property(
      mangaArbitrary,
      (manga) => {
        const serialized = JSON.stringify(manga)
        const deserialized = JSON.parse(serialized)
        expect(deserialized).toEqual(manga)
      }
    ),
    { numRuns: 100 }
  )
})
```

#### 2. Invariant Properties

**Pattern**: Property that remains true after transformation

```typescript
// **Property: Adding manga increases library size by 1**
test('library size increases by 1 when adding manga', async () => {
  await fc.assert(
    fc.property(
      mangaArbitrary,
      async (manga) => {
        const sizeBefore = await getLibrarySize(userId)
        await addToLibrary(userId, manga.id)
        const sizeAfter = await getLibrarySize(userId)
        
        expect(sizeAfter).toBe(sizeBefore + 1)
      }
    ),
    { numRuns: 100 }
  )
})
```

#### 3. Idempotence Properties

**Pattern**: `f(f(x)) === f(x)`

```typescript
// **Property: Marking chapter as read is idempotent**
test('marking chapter as read twice has same effect as once', async () => {
  await fc.assert(
    fc.property(
      fc.uuid(),
      async (chapterId) => {
        await markAsRead(userId, chapterId)
        const firstResult = await getReadStatus(userId, chapterId)
        
        await markAsRead(userId, chapterId)
        const secondResult = await getReadStatus(userId, chapterId)
        
        expect(secondResult).toEqual(firstResult)
      }
    ),
    { numRuns: 100 }
  )
})
```

#### 4. Metamorphic Properties

**Pattern**: Relationship between inputs and outputs

```typescript
// **Property: Filtering reduces or maintains size**
test('filtering never increases collection size', () => {
  fc.assert(
    fc.property(
      fc.array(mangaArbitrary),
      fc.string(),
      (manga, query) => {
        const filtered = filterManga(manga, query)
        expect(filtered.length).toBeLessThanOrEqual(manga.length)
      }
    ),
    { numRuns: 100 }
  )
})
```

#### 5. Error Condition Properties

**Pattern**: Invalid inputs produce errors

```typescript
// **Property: Empty title is rejected**
test('manga with empty title is rejected', async () => {
  await fc.assert(
    fc.property(
      fc.string().filter(s => s.trim() === ''),
      async (emptyTitle) => {
        const result = await addManga({ title: emptyTitle })
        expect(result.success).toBe(false)
      }
    ),
    { numRuns: 100 }
  )
})
```

### Custom Generators

Create reusable generators for domain objects:

```typescript
// arbitraries.ts
import * as fc from 'fast-check'

export const mangaArbitrary = fc.record({
  id: fc.uuid(),
  title: fc.string({ minLength: 1, maxLength: 200 }),
  slug: fc.string({ minLength: 1, maxLength: 200 })
    .map(s => s.toLowerCase().replace(/\s+/g, '-')),
  author: fc.string({ minLength: 1, maxLength: 100 }),
  year: fc.integer({ min: 1900, max: 2100 }),
  status: fc.constantFrom('ongoing', 'completed', 'hiatus'),
})

export const chapterArbitrary = fc.record({
  id: fc.uuid(),
  mangaId: fc.uuid(),
  number: fc.double({ min: 1, max: 1000, noNaN: true }),
  title: fc.option(fc.string({ maxLength: 200 })),
})

// Usage
test('some property', () => {
  fc.assert(
    fc.property(mangaArbitrary, (manga) => {
      // Test with generated manga
    }),
    { numRuns: 100 }
  )
})
```

### Handling Async Properties

```typescript
test('async property', async () => {
  await fc.assert(
    fc.asyncProperty(
      fc.uuid(),
      async (id) => {
        const result = await fetchManga(id)
        expect(result).toBeDefined()
      }
    ),
    { numRuns: 100 }
  )
})
```

### Debugging Failed Properties

When a property fails, fast-check provides the failing case:

```typescript
// Property failed after 42 runs with input: { title: "", author: "a" }
```

To reproduce:
```typescript
fc.assert(
  fc.property(/* ... */),
  { 
    numRuns: 100,
    seed: 1234567890, // Use seed from failure
    path: "42:0",     // Use path from failure
  }
)
```

## Integration Tests

### Database Tests

Use a test database (Docker or in-memory):

```typescript
import { beforeAll, afterAll, beforeEach } from 'vitest'
import { migrate } from 'drizzle-orm/postgres-js/migrator'
import { db } from './db'

beforeAll(async () => {
  // Run migrations
  await migrate(db, { migrationsFolder: './migrations' })
})

afterAll(async () => {
  // Cleanup
  await db.close()
})

beforeEach(async () => {
  // Clear tables
  await db.delete(manga)
  await db.delete(users)
})

test('database integration', async () => {
  const inserted = await db.insert(manga).values({
    title: 'Test Manga',
  })
  
  const retrieved = await db.query.manga.findFirst({
    where: eq(manga.id, inserted.id),
  })
  
  expect(retrieved).toEqual(inserted)
})
```

### API Tests

Mock external APIs:

```typescript
import { rest } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer(
  rest.get('https://api.mangapark.com/manga/:id', (req, res, ctx) => {
    return res(ctx.json({ id: req.params.id, title: 'Test' }))
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

test('fetches manga from API', async () => {
  const manga = await fetchMangaFromSource('123')
  expect(manga.title).toBe('Test')
})
```

## E2E Tests

### Framework: Playwright

**Scope**: Complete user flows across the application

**Location**: `tests/e2e/` in web and mobile repos

### Test Structure

```typescript
import { test, expect } from '@playwright/test'

test.describe('Manga Library', () => {
  test.beforeEach(async ({ page }) => {
    // Setup: login, navigate
    await page.goto('/library')
  })

  test('should add manga to library', async ({ page }) => {
    // Search for manga
    await page.fill('[data-testid="search-input"]', 'One Piece')
    await page.click('[data-testid="search-button"]')
    
    // Add to library
    await page.click('[data-testid="add-to-library"]')
    
    // Verify in library
    await page.goto('/library')
    await expect(page.locator('text=One Piece')).toBeVisible()
  })
})
```

### Authentication in E2E

Use mock OAuth for E2E tests (not real Google OAuth):

```typescript
// Setup mock session
test.beforeEach(async ({ page }) => {
  await page.request.post('/api/test/auth/mock', {
    data: { email: 'e2e-test@example.com' },
  })
})

// Cleanup
test.afterEach(async ({ page }) => {
  await page.request.delete('/api/test/auth/mock')
})
```

## Coverage Requirements

### Minimum Coverage

- **Critical paths**: 80% coverage
- **Authentication**: 100% coverage
- **Data persistence**: 100% coverage
- **Business logic**: 80% coverage
- **UI components**: 60% coverage (focus on behavior)

### Measuring Coverage

```bash
# Run tests with coverage
npm test -- --coverage

# View coverage report
open coverage/index.html
```

### Coverage Configuration

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'dist/',
        '**/*.test.ts',
        '**/*.config.ts',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
})
```

## CI/CD Integration

### Pre-commit

Run unit tests before commit:

```bash
# .husky/pre-commit
npm test -- --run
```

### CI Pipeline

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test -- --coverage
      - run: npm run test:e2e
```

### Pre-deployment

All tests MUST pass before deployment:

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    needs: test # Wait for tests to pass
    steps:
      - run: npm run deploy
```

## Test Organization

### File Naming

```
src/
├── manga.ts
├── manga.test.ts           # Unit tests
├── manga.property.test.ts  # Property tests
└── manga.integration.test.ts # Integration tests
```

### Test Suites

Group related tests:

```typescript
describe('Manga Management', () => {
  describe('getManga', () => {
    // Tests for getManga
  })

  describe('addManga', () => {
    // Tests for addManga
  })

  describe('updateManga', () => {
    // Tests for updateManga
  })
})
```

## Best Practices

1. **Test behavior, not implementation** - Don't test internal details
2. **One assertion per test** - Keep tests focused (guideline, not rule)
3. **Clear test names** - Describe what is being tested
4. **Arrange-Act-Assert** - Structure tests consistently
5. **Independent tests** - Tests should not depend on each other
6. **Fast tests** - Unit tests should run in milliseconds
7. **Deterministic** - Tests should always produce same result
8. **Readable** - Tests are documentation

## Common Pitfalls

❌ **Testing implementation details**
```typescript
// Bad
expect(component.state.count).toBe(1)

// Good
expect(screen.getByText('Count: 1')).toBeInTheDocument()
```

❌ **Brittle selectors**
```typescript
// Bad
await page.click('.css-abc123')

// Good
await page.click('[data-testid="add-button"]')
```

❌ **Not cleaning up**
```typescript
// Bad
test('test 1', () => {
  globalState.value = 1
})

// Good
afterEach(() => {
  globalState.value = 0
})
```

❌ **Slow tests**
```typescript
// Bad
await page.waitForTimeout(5000)

// Good
await page.waitForSelector('[data-testid="loaded"]')
```

## Testing Checklist

- [ ] Unit tests for all business logic
- [ ] Property tests for universal properties
- [ ] Integration tests for external dependencies
- [ ] E2E tests for critical user flows
- [ ] All tests pass locally
- [ ] Coverage meets minimum thresholds
- [ ] Tests are fast and deterministic
- [ ] Tests are well-named and readable
- [ ] Mocks are used appropriately
- [ ] Tests clean up after themselves
