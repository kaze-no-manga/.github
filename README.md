# Kaze no Manga (風の漫画) - Project Documentation

> **Complete project knowledge base for humans and AI agents**

## Table of Contents

- [Project Overview](#project-overview)
- [Problem & Solution](#problem--solution)
- [Architecture](#architecture)
- [Functional Requirements](#functional-requirements)
- [Technical Stack](#technical-stack)
- [Repository Structure](#repository-structure)
- [Development Workflow](#development-workflow)
- [Roadmap](#roadmap)

---

## Project Overview

**Kaze no Manga** (風の漫画, "Wind Manga") is a cross-device manga reading tracker that eliminates the frustration of manually tracking reading progress across multiple devices and sources.

### Core Value Proposition

- **Cross-device synchronization**: Never lose your place again
- **Multi-source aggregation**: Read from MangaPark, OmegaScans, and more
- **Automatic progress tracking**: Seamless chapter tracking
- **Smart notifications**: Get notified when new chapters are available
- **External tracker sync**: Integrate with AniList and MyAnimeList
- **Community features**: Shared lists, recommendations, and social interactions

### Target Users

- Manga readers using multiple devices (phone, tablet, desktop)
- Users reading from multiple sources
- People who want automatic progress tracking and notifications
- Readers using external trackers (AniList, MyAnimeList)

---

## Problem & Solution

### The Problem

Every manga reader faces the same frustration:
- Manually tracking which chapter you've read across different devices
- Losing your place when switching between sources
- Missing new chapter releases
- Re-reading chapters by mistake
- No reliable cross-device synchronization

### The Solution

A centralized platform that:
1. **Tracks progress automatically** across all devices
2. **Aggregates manga** from multiple sources into a unified entity
3. **Notifies users** when new chapters are available (email + push)
4. **Syncs bidirectionally** with AniList and MyAnimeList
5. **Provides discovery** through recommendations and community lists

---

## Architecture

### High-Level Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    CloudFront CDN                         │
│              (web assets + S3 images)                     │
└──────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌──────────────────┐            ┌──────────────────┐
│  Lambda@Edge     │            │    AppSync       │
│ (React Router    │            │   (GraphQL)      │
│      SSR)        │            └──────────────────┘
└──────────────────┘                     │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                         ┌────────┐ ┌────────┐ ┌────────┐
                         │Lambda  │ │Lambda  │ │Lambda  │
                         │Resolver│ │Resolver│ │Resolver│
                         └────────┘ └────────┘ └────────┘
                              │          │          │
                    ┌─────────┴──────────┴──────────┴─────────┐
                    ▼                                          ▼
              ┌──────────┐                              ┌──────────┐
              │   RDS    │                              │ DynamoDB │
              │(Postgres)│                              │ (Cache)  │
              └──────────┘                              └──────────┘
                    │
                    ▼
              ┌──────────┐         ┌──────────┐
              │   SQS    │────────▶│  Lambda  │
              │ (Jobs)   │         │(Scraper) │
              └──────────┘         └──────────┘
                                         │
                                         ▼
                                   ┌──────────┐
                                   │    S3    │
                                   │ (Images) │
                                   └──────────┘
```

### Multi-Repository Structure

The project is organized as a **multi-repo architecture** with NPM private packages:

```
kaze-no-manga/ (GitHub Organization)
├── .github/              # Organization-wide documentation
├── brand/                # NPM: Design tokens + Tailwind preset
├── models/               # NPM: TypeScript interfaces + Zod schemas + GraphQL types
├── scraper/              # NPM: Stateless scrapers with common interface
├── database/             # Drizzle schema + migrations + utilities
├── backend/              # AppSync GraphQL + Lambda resolvers + Step Functions
├── web/                  # React Router v7 SSR (Lambda@Edge)
├── mobile/               # Flutter app (iOS + Android)
├── mcp-server/           # MCP server for LLM integration (experiment)
└── telegram-bot/         # Telegram bot (experiment)
```

---

## Functional Requirements

### 1. Authentication & Users

**Providers:**
- Google OAuth (initial)
- Discord OAuth (future)
- Passkey (future)

**User Features:**
- Private library (public libraries in future)
- Personal reading statistics
- Notification preferences (granular per manga)
- Profile settings

### 2. Manga & Sources

**Manga Entity:**
- Centralized manga entity (not tied to users)
- Multiple sources per manga (MangaPark, OmegaScans, etc.)
- Alternative titles support
- Metadata: genres, authors, year, status
- Automatic source selection (best availability + updates)

**Chapter Management:**
- Decimal chapter numbers (5.5, 5.1, etc.)
- Correct ordering (5 → 5.1 → 5.5 → 6)
- Multiple sub-chapters support
- Images downloaded to S3 (not just URLs)

**Source Aggregation:**
- Unified manga across sources
- Automatic migration when source becomes unavailable (with approval)
- Source transparency (initially visible, then transparent)

### 3. Library & Progress Tracking

**Library Features:**
- Add/remove manga
- Optional metadata: status (Reading, Completed, Plan to Read, Dropped), rating, notes
- Custom lists (future)
- Public/shared lists (future)

**Progress Tracking:**
- Initial: Manual click to mark as read
- Future: Automatic on scroll to end of chapter
- Mark chapters as read without opening
- Progress doesn't auto-update if reading out of order (e.g., chapter 5 without 4)
- Prompt user when marking chapter 10 as read (auto-mark 1-9?)
- Reading history with timestamps

### 4. Reader

**Features:**
- Vertical scroll reader
- Infinite scrolling between chapters (1 → 2 → 3 seamlessly)
- Visual indicator when transitioning between chapters
- Lazy loading images
- Preload next chapter automatically
- Configurable settings: image width, spacing, theme

**Mobile:**
- Swipe gestures for chapter navigation
- Optimized for touch

### 5. Search & Discovery

**Search:**
- Initially: Title only
- Future: Author, genre, year, filters

**Discovery:**
- Homepage sections: Popular (real-time), Recently Added (to database)
- Popular = number of users currently reading
- Infinite scroll on homepage
- Algorithmic recommendations (users who read X also read Y)
- Community lists (curated + collaborative)

### 6. Notifications

**Types:**
- Email (HTML templates via SES)
- Push browser (FCM for web + mobile, APNS for iOS)

**Configuration:**
- Granular per manga
- Configurable limit (e.g., max 10/day)
- Aggregated notifications when multiple chapters available

**Frequency:**
- Daily check for new chapters (Step Functions)
- Priority-based: ongoing manga checked daily, completed manga checked weekly/monthly

### 7. External Tracker Sync

**Supported:**
- AniList
- MyAnimeList

**Sync Strategy:**
- Bidirectional sync (Kaze ↔ AniList/MAL)
- Initially manual, future automatic
- Batch import from AniList/MAL
- Conflict resolution: prompt user
- Mapping: prompt user for confirmation

### 8. Community Features (Future)

- Follow/follower system
- Shared reading lists (public + collaborative)
- User recommendations
- Social interactions

### 9. Moderation & Quality

**Reporting:**
- Users can report: duplicates, broken links, missing images
- Reports go to moderation queue (future system)
- Feedback to users when resolved

**Content Management:**
- Only devs can add new sources
- No approval needed for manga added to database
- Automatic scraping robot populates database

---

## Technical Stack

### Frontend

- **Web**: React Router v7 (SSR on Lambda@Edge)
- **Mobile**: Flutter (iOS + Android)
- **Styling**: Tailwind CSS (via brand package)
- **State Management**: TanStack Query (for GraphQL)
- **Offline**: SQLite (mobile) for offline-first + chapter downloads

### Backend

- **API**: AWS AppSync (GraphQL)
- **Resolvers**: Lambda (Node.js) + VTL for simple queries
- **Direct Resolvers**: AppSync → DynamoDB for cache
- **Authentication**: AWS Cognito (Google, Discord, Passkey)
- **Jobs**: Step Functions + EventBridge (daily checks)
- **Queue**: SQS (scraper jobs)

### Database

- **Primary**: RDS Postgres (manga, users, library, progress)
- **Cache**: DynamoDB (GraphQL queries, sessions, preferences)
- **TTL**: Automatic on DynamoDB items
- **Migrations**: Drizzle ORM + CDK Custom Resource

### Storage & CDN

- **Images**: S3 (manga chapter images)
- **CDN**: CloudFront (web assets + S3 images)
- **Caching**: CloudFront + DynamoDB

### Notifications

- **Email**: AWS SES (HTML templates)
- **Push**: AWS SNS → FCM (web + Android) + APNS (iOS)

### Infrastructure

- **IaC**: AWS CDK (TypeScript)
- **Stacks**: Separate stacks (Auth, API, Storage, Jobs, Frontend)
- **Cross-stack**: SSM Parameters
- **Secrets**: AWS Secrets Manager
- **Environments**: Production only (initially)

### CI/CD

- **Pipeline**: GitHub Actions (on push)
- **Deploy**: AWS CDK via GitHub Actions
- **Strategy**: Blue/Green deployment
- **Testing**: Unit tests mandatory before deploy

### Packages

- **Brand**: NPM private package (tokens + Tailwind preset)
- **Models**: NPM private package (TS interfaces + Zod + GraphQL SDL)
- **Scraper**: NPM private package (common interface + decorators)

---

## Repository Structure

### `.github` (This Repo)

Organization-wide documentation, guidelines, and public profile.

### `brand`

Design system package:
- Color tokens (purple/lilac theme)
- Typography tokens
- Spacing, border-radius, shadows
- Tailwind preset (extends default config)
- Exported as NPM package

**Formats**: JSON, TypeScript, CSS variables

### `models`

Shared types package:
- TypeScript interfaces
- Zod schemas for validation
- GraphQL SDL files
- Generated GraphQL types (codegen)
- Exported as NPM package

### `scraper`

Stateless scraper package:
- Common interface: `MangaSource`
  - `search(query: string): Promise<Manga[]>`
  - `getManga(id: string): Promise<MangaDetail>`
  - `getChapters(mangaId: string): Promise<Chapter[]>`
  - `getChapterImages(chapterId: string): Promise<string[]>`
- Separate file per source (mangapark.ts, omegascans.ts)
- Decorator/wrapper for rate limiting + retry logic
- Configurable timeout per source
- No database access (stateless)
- Exported as NPM package

### `database`

Database schema and migrations:
- Drizzle ORM schema
- Migration files
- Seed data
- Utility functions
- Applied via CDK Custom Resource (Lambda)

**Main Entities:**
- `User` (Cognito ID, email, preferences)
- `Manga` (id, title, alt_titles, metadata, sources[])
- `Source` (id, name, base_url, status)
- `Chapter` (id, manga_id, number, title, source_id)
- `UserLibrary` (user_id, manga_id, status, rating, notes, progress)
- `ReadingHistory` (user_id, chapter_id, timestamp, completed)
- `Notification` (user_id, manga_id, chapter_id, sent_at, type)

### `backend`

GraphQL API and business logic:
- AppSync GraphQL schema (deployed via CDK)
- Lambda resolvers (per resolver)
- VTL templates for simple queries
- Direct resolvers (AppSync → DynamoDB)
- Step Functions (daily chapter checks)
- SQS handlers (scraper jobs)
- Notification logic (SES + SNS)

**GraphQL Schema (main operations):**
```graphql
# Queries
searchManga(query: String!): [Manga!]!
getManga(id: ID!): Manga
getChapters(mangaId: ID!): [Chapter!]!
getLibrary: [LibraryEntry!]!

# Mutations
addToLibrary(mangaId: ID!): LibraryEntry!
updateProgress(mangaId: ID!, chapterId: ID!): LibraryEntry!
updateLibraryEntry(mangaId: ID!, input: LibraryInput!): LibraryEntry!
removeFromLibrary(mangaId: ID!): Boolean!
```

### `web`

React Router v7 web application:
- SSR on Lambda@Edge
- Static pages: homepage, about, manga detail
- CSR pages: library, settings, reader
- GraphQL client (TanStack Query)
- Auth via Cognito (OAuth flow)

**Routes:**
- `/` - Homepage (SSR)
- `/about` - About page (SSR)
- `/manga/:slug` - Manga detail (SSR)
- `/manga/:slug/:chapter` - Reader (CSR)
- `/library` - User library (CSR, auth required)
- `/settings` - User settings (CSR, auth required)
- `/auth/callback` - OAuth callback

### `mobile`

Flutter mobile application:
- iOS + Android
- GraphQL client (graphql_flutter)
- SQLite for offline-first
- Download chapters for offline reading (future)
- OAuth flow (Google, Discord)
- Push notifications (FCM + APNS)

**Features:**
- Auth, library, reader, progress sync
- Swipe gestures
- Offline support

### `mcp-server`

MCP (Model Context Protocol) server for LLM integration:
- Lambda-based
- Tools: search manga, read chapters, add to library, mark as read
- Resources: manga data, user library
- Auth: user-specific (future)
- Experimental feature

### `telegram-bot`

Telegram bot for manga tracking:
- Lambda-based
- Commands: `/search`, `/library`, `/read`, `/track`
- GraphQL backend integration
- Auth: user-specific (future)
- Experimental feature

---

## Development Workflow

### Package Management

- **Multi-repo** approach (not monorepo)
- **NPM private packages** via GitHub Packages
- **Conventional commits** + semantic versioning
- **Manual dependency management** (no Turborepo/Nx)

### Development Order

**Phase 0: Setup**
1. Create GitHub Organization
2. Setup NPM organization (GitHub Packages)
3. Define naming conventions

**Phase 1: Foundation**
1. `brand` - Design tokens + Tailwind preset
2. `models` - Base interfaces + Zod schemas
3. `database` - Drizzle schema + migrations

**Phase 2: Backend Core**
1. `scraper` - Interface + 2-3 sources (MangaPark, OmegaScans)
2. `backend` - AppSync schema + Lambda resolvers
3. Infrastructure - CDK stacks (Auth, API, Database)

**Phase 3: Frontend**
1. `web` - React Router v7 + pages + GraphQL client
2. Infrastructure - CDK stack (Frontend)

**Phase 4: Jobs & Notifications**
1. `backend` - Step Functions + SQS + notifications
2. Infrastructure - CDK stack (Jobs)

**Phase 5: Mobile**
1. `mobile` - Flutter app + GraphQL + SQLite

**Phase 6: Experiments**
1. `mcp-server` - MCP server
2. `telegram-bot` - Telegram bot

### CI/CD

- **Trigger**: On push to main
- **Pipeline**: GitHub Actions
- **Deploy**: AWS CDK
- **Strategy**: Blue/Green
- **Tests**: Unit tests mandatory

### Testing

- Unit tests (Vitest)
- E2E tests (Playwright for web, Flutter integration tests for mobile)
- Property-based tests (fast-check) for critical logic

---

## Roadmap

### MVP (Current Focus)

- [x] Architecture design
- [ ] Brand package (tokens + Tailwind)
- [ ] Models package (interfaces + Zod + GraphQL)
- [ ] Database schema (Drizzle)
- [ ] Scraper package (MangaPark + OmegaScans)
- [ ] Backend (AppSync + Lambda resolvers)
- [ ] Web (React Router v7 + basic pages)
- [ ] Authentication (Cognito + Google OAuth)
- [ ] Library management
- [ ] Reader (vertical scroll)
- [ ] Manual progress tracking

### Phase 2: Automation

- [ ] Daily chapter checks (Step Functions)
- [ ] Email notifications (SES)
- [ ] Push notifications (SNS + FCM)
- [ ] Automatic progress tracking (scroll-based)
- [ ] Image download to S3

### Phase 3: Mobile

- [ ] Flutter app (iOS + Android)
- [ ] Offline support (SQLite)
- [ ] Push notifications

### Phase 4: Discovery

- [ ] Search filters (genre, author, year)
- [ ] Algorithmic recommendations
- [ ] Popular manga (real-time)
- [ ] Recently added manga

### Phase 5: External Sync

- [ ] AniList integration (manual sync)
- [ ] MyAnimeList integration (manual sync)
- [ ] Batch import
- [ ] Automatic sync (future)

### Phase 6: Community

- [ ] User profiles (public)
- [ ] Follow/follower system
- [ ] Shared lists (public + collaborative)
- [ ] Social recommendations

### Phase 7: Advanced

- [ ] Custom lists
- [ ] Advanced filters
- [ ] Moderation system
- [ ] Native mobile apps (if needed)
- [ ] Additional auth providers (Discord, Passkey)

### Experiments (Ongoing)

- [ ] MCP server for LLM integration
- [ ] Telegram bot

---

## Key Decisions & Rationale

### Why Multi-Repo?

- Clear separation of concerns
- Independent versioning
- Easier to manage permissions
- Scalable for large teams (future)

### Why AppSync?

- Managed GraphQL service
- Built-in auth integration (Cognito)
- VTL for simple queries (no Lambda overhead)
- Direct resolvers to DynamoDB
- Real-time subscriptions (future)

### Why React Router v7?

- Modern SSR on Lambda@Edge
- No vendor lock-in (vs Next.js on Vercel)
- Full control over AWS infrastructure
- Better performance on CloudFront

### Why Flutter?

- Single codebase for iOS + Android
- Native performance
- Rich UI components
- Strong offline support

### Why RDS + DynamoDB?

- RDS for relational data (manga, users, library)
- DynamoDB for cache + sessions (fast reads)
- Best of both worlds

### Why S3 for Images?

- Permanent storage (not just URLs)
- Protection against source downtime
- CloudFront CDN integration
- Image optimization (WebP, responsive)

---

## Contributing

This project is currently in active development. Contributions will be welcome once the MVP is complete.

### Development Principles

1. **Incremental**: Small, achievable milestones
2. **User-focused**: Every feature solves a real problem
3. **Scalable**: Architecture ready for growth
4. **Secure**: Privacy and data protection first
5. **Mobile-first**: Responsive design from the start

---

## License

MIT License - see individual repositories for details.

---

## Contact

- **Organization**: [kaze-no-manga](https://github.com/kaze-no-manga)
- **Project Status**: In Development (MVP Phase)

---

**Last Updated**: December 2025
