# Implementation Plan - MVP Coordination

This task list coordinates development across all repositories for the Kaze no Manga MVP. Each top-level task represents work in a specific repository, with references to that repository's detailed spec.

## Phase 1: Foundation (Weeks 1-2)

### 1. Brand Package

- [ ] 1.1 Create brand repository and NPM package structure
  - Initialize repository with TypeScript, Vitest, Biome
  - Configure package.json for NPM publishing
  - Setup GitHub Actions for automated publishing
  - _Requirements: 1.1, 5.1, 5.2_
  - _Details: See `brand/.kiro/specs/01-mvp/`_

- [ ] 1.2 Define design tokens
  - Create color palette (purple/lilac theme)
  - Define typography scale
  - Define spacing, border-radius, shadows
  - Export as JSON and TypeScript
  - _Requirements: 1.1_
  - _Details: See `brand/.kiro/specs/01-mvp/`_

- [ ] 1.3 Create Tailwind CSS preset
  - Extend Tailwind with design tokens
  - Configure theme, colors, typography
  - Test preset in sample project
  - _Requirements: 1.1_
  - _Details: See `brand/.kiro/specs/01-mvp/`_

- [ ]* 1.4 Write tests for brand package
  - Unit tests for token exports
  - Property tests for Tailwind preset
  - _Requirements: 5.3, 5.4_

- [ ] 1.5 Publish brand package to NPM
  - Version 0.1.0
  - Verify package is accessible
  - Create GitHub release
  - _Requirements: 1.4, 1.5_

---

### 2. Models Package

- [ ] 2.1 Create models repository and NPM package structure
  - Initialize repository with TypeScript, Vitest, Biome
  - Configure package.json for NPM publishing
  - Setup GitHub Actions for automated publishing
  - Install brand package as dependency
  - _Requirements: 1.2, 5.1, 5.2_
  - _Details: See `models/.kiro/specs/01-mvp/`_

- [ ] 2.2 Define TypeScript interfaces for core entities
  - User, Manga, MangaSource, Chapter
  - UserLibrary, ReadingHistory
  - Result types, error types
  - _Requirements: 1.2_
  - _Details: See `models/.kiro/specs/01-mvp/`_

- [ ] 2.3 Create Zod schemas for validation
  - Schema for each entity
  - Validation utilities
  - Type inference from schemas
  - _Requirements: 1.2_
  - _Details: See `models/.kiro/specs/01-mvp/`_

- [ ] 2.4 Define GraphQL SDL
  - GraphQL types for all entities
  - Query and Mutation definitions
  - Input types
  - _Requirements: 1.2_
  - _Details: See `models/.kiro/specs/01-mvp/`_

- [ ] 2.5 Setup GraphQL Codegen
  - Configure codegen for TypeScript types
  - Generate types from SDL
  - Export generated types
  - _Requirements: 1.2_
  - _Details: See `models/.kiro/specs/01-mvp/`_

- [ ]* 2.6 Write tests for models package
  - Unit tests for type exports
  - Property tests for Zod schemas
  - Validation tests
  - _Requirements: 5.3, 5.4_

- [ ] 2.7 Publish models package to NPM
  - Version 0.1.0
  - Verify package is accessible
  - Create GitHub release
  - _Requirements: 1.4, 1.5_

---

### 3. Database Package

- [ ] 3.1 Create database repository structure
  - Initialize repository with TypeScript, Vitest, Biome
  - Install models package as dependency
  - Install Drizzle ORM
  - _Requirements: 1.3, 5.1, 5.2_
  - _Details: See `database/.kiro/specs/01-mvp/`_

- [ ] 3.2 Define Drizzle schema for all tables
  - users, manga, manga_sources, chapters
  - user_library, reading_history
  - Relationships and foreign keys
  - Indexes for performance
  - _Requirements: 1.3_
  - _Details: See `database/.kiro/specs/01-mvp/`_

- [ ] 3.3 Create initial migration
  - Generate migration from schema
  - Test migration locally
  - Document migration process
  - _Requirements: 1.3_
  - _Details: See `database/.kiro/specs/01-mvp/`_

- [ ] 3.4 Create seed data (optional)
  - Sample manga data
  - Sample users
  - Sample chapters
  - _Requirements: 1.3_
  - _Details: See `database/.kiro/specs/01-mvp/`_

- [ ]* 3.5 Write tests for database package
  - Unit tests for schema exports
  - Integration tests for migrations
  - _Requirements: 5.3, 5.4_

---

### 4. Checkpoint: Foundation Phase Complete

- [ ] 4.1 Verify all foundation packages are ready
  - Confirm brand@0.1.0 published to NPM
  - Confirm models@0.1.0 published to NPM
  - Confirm database schema defined
  - All tests passing (80%+ coverage)
  - All documentation complete
  - _Requirements: 4.1, 5.4, 6.1_

---

## Phase 2: Backend (Weeks 3-5)

### 5. Scraper Package

- [ ] 5.1 Create scraper repository and NPM package structure
  - Initialize repository with TypeScript, Vitest, Biome
  - Configure package.json for NPM publishing
  - Setup GitHub Actions for automated publishing
  - Install models package as dependency
  - _Requirements: 2.1, 5.1, 5.2_
  - _Details: See `scraper/.kiro/specs/01-mvp/`_

- [ ] 5.2 Define MangaSource interface
  - search, getManga, getChapters, getChapterImages methods
  - Error types
  - Rate limiting interface
  - _Requirements: 2.1_
  - _Details: See `scraper/.kiro/specs/01-mvp/`_

- [ ] 5.3 Implement MangaPark scraper
  - Implement MangaSource interface
  - Handle pagination
  - Parse HTML responses
  - Error handling
  - _Requirements: 2.1_
  - _Details: See `scraper/.kiro/specs/01-mvp/`_

- [ ] 5.4 Implement OmegaScans scraper
  - Implement MangaSource interface
  - Handle API responses
  - Parse data
  - Error handling
  - _Requirements: 2.1_
  - _Details: See `scraper/.kiro/specs/01-mvp/`_

- [ ] 5.5 Add rate limiting and retry logic
  - Decorator for rate limiting
  - Exponential backoff for retries
  - Configurable timeouts
  - _Requirements: 2.1_
  - _Details: See `scraper/.kiro/specs/01-mvp/`_

- [ ]* 5.6 Write tests for scraper package
  - Unit tests for each scraper
  - Property tests for interface compliance
  - Integration tests with mocked APIs
  - _Requirements: 5.3, 5.4_

- [ ] 5.7 Publish scraper package to NPM
  - Version 0.1.0
  - Verify package is accessible
  - Create GitHub release
  - _Requirements: 1.4, 1.5_

---

### 6. Backend Infrastructure (AWS CDK)

- [ ] 6.1 Create backend repository structure
  - Initialize repository with TypeScript, Vitest, Biome
  - Install models, scraper, database packages
  - Setup AWS CDK
  - Configure CDK stacks
  - _Requirements: 2.2, 5.1, 5.2_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 6.2 Create AuthStack (Cognito + Google OAuth)
  - Cognito User Pool
  - Google OAuth provider
  - User Pool Client
  - Export user pool ARN
  - _Requirements: 2.3, 3.4_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 6.3 Create DatabaseStack (RDS + DynamoDB)
  - RDS PostgreSQL cluster
  - DynamoDB cache table
  - VPC configuration
  - Security groups
  - Export database endpoint
  - _Requirements: 2.4, 2.6_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 6.4 Create migration Lambda and Custom Resource
  - Lambda function to run Drizzle migrations
  - CDK Custom Resource to trigger migrations on deploy
  - Test migrations in dev environment
  - _Requirements: 2.4_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 6.5 Create StorageStack (S3 + CloudFront)
  - S3 bucket for manga images (future use)
  - CloudFront distribution
  - Bucket policies
  - _Requirements: 2.6_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

---

### 7. Backend GraphQL API

- [ ] 7.1 Create ApiStack (AppSync)
  - AppSync GraphQL API
  - Import GraphQL schema from models package
  - Configure authentication (Cognito)
  - Configure logging
  - Export API endpoint
  - _Requirements: 2.2, 2.5_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.2 Implement searchManga resolver
  - Lambda function
  - Use scraper package to search sources
  - Aggregate results
  - Cache in DynamoDB
  - _Requirements: 2.5, 3.1_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.3 Implement getManga resolver
  - Lambda function
  - Check database first
  - Fall back to scraper if not found
  - Store in database
  - _Requirements: 2.5, 3.2_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.4 Implement getChapters resolver
  - Lambda function
  - Check database first
  - Fall back to scraper if not found
  - Store in database
  - Sort by chapter number (handle decimals)
  - _Requirements: 2.5, 3.2_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.5 Implement addToLibrary resolver
  - Lambda function
  - Validate user authentication
  - Add manga to user_library table
  - Return library entry
  - _Requirements: 2.5, 3.5_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.6 Implement updateProgress resolver
  - Lambda function
  - Validate user authentication
  - Mark chapter as read in reading_history
  - Update user_library current chapter
  - _Requirements: 2.5, 3.6_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.7 Implement getLibrary resolver
  - Lambda function
  - Validate user authentication
  - Query user_library table
  - Join with manga data
  - Return library entries
  - _Requirements: 2.5, 3.5_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 7.8 Implement removeFromLibrary resolver
  - Lambda function
  - Validate user authentication
  - Remove from user_library table
  - Return success
  - _Requirements: 2.5, 3.5_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ]* 7.9 Write tests for Lambda resolvers
  - Unit tests for each resolver
  - Property tests for data operations
  - Integration tests with test database
  - _Requirements: 5.3, 5.4_

---

### 8. Backend Deployment

- [ ] 8.1 Setup GitHub Actions for backend deployment
  - Configure AWS credentials
  - CDK synth and deploy on merge to main
  - Run tests before deploy
  - _Requirements: 7.1, 7.2, 7.3_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 8.2 Deploy backend to AWS
  - Deploy all stacks
  - Verify API endpoint is accessible
  - Test authentication flow
  - Test all resolvers
  - _Requirements: 2.2, 2.3, 2.4, 2.5, 2.6_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

- [ ] 8.3 Configure monitoring and alerts
  - CloudWatch dashboards
  - Error rate alarms
  - Latency alarms
  - Database connection alarms
  - _Requirements: 7.5_
  - _Details: See `backend/.kiro/specs/01-mvp/`_

---

### 9. Checkpoint: Backend Phase Complete

- [ ] 9.1 Verify backend is fully functional
  - Confirm scraper@0.1.0 published to NPM
  - Confirm backend deployed to AWS
  - Confirm GraphQL API accessible via HTTPS
  - Confirm authentication working (Google OAuth)
  - Confirm all resolvers functional
  - All tests passing (80%+ coverage)
  - Monitoring and alerts configured
  - _Requirements: 4.2, 4.3, 5.4, 6.2_

---

## Phase 3: Frontend (Weeks 6-8)

### 10. Web Application Setup

- [ ] 10.1 Create web repository structure
  - Initialize React Router v7 project
  - Install brand and models packages
  - Configure TypeScript, Vitest, Biome
  - Setup Tailwind CSS with brand preset
  - _Requirements: 3.1, 5.1, 5.2_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 10.2 Configure GraphQL client
  - Install TanStack Query
  - Configure GraphQL client
  - Setup authentication headers
  - Create query hooks
  - _Requirements: 3.1_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 10.3 Setup authentication
  - Implement OAuth callback route
  - Store JWT tokens
  - Create auth context
  - Protected route wrapper
  - _Requirements: 3.4_
  - _Details: See `web/.kiro/specs/01-mvp/`_

---

### 11. Web Application Pages

- [ ] 11.1 Implement homepage (SSR)
  - Search bar component
  - Search results grid
  - Manga card component
  - Infinite scroll
  - _Requirements: 3.1_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 11.2 Implement manga detail page (SSR)
  - Manga info display
  - Chapter list
  - Add to library button
  - Source indicator
  - _Requirements: 3.2_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 11.3 Implement reader page (CSR)
  - Vertical scroll layout
  - Image lazy loading
  - Chapter navigation
  - Mark as read button
  - Infinite scroll between chapters
  - _Requirements: 3.3, 3.6_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 11.4 Implement library page (CSR, auth required)
  - Library grid
  - Filter by status
  - Sort options
  - Remove from library
  - Progress indicators
  - _Requirements: 3.5_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 11.5 Implement settings page (CSR, auth required)
  - User profile
  - Notification preferences (UI only, no backend yet)
  - Theme toggle (light/dark)
  - Logout button
  - _Requirements: 3.5_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 11.6 Implement about page (SSR)
  - Project description
  - Features list
  - Contact information
  - _Requirements: 6.5_
  - _Details: See `web/.kiro/specs/01-mvp/`_

---

### 12. Web Application Components

- [ ] 12.1 Create UI component library
  - Button, Input, Card components
  - Modal, Dropdown components
  - Loading states, Error states
  - Use brand package for styling
  - _Requirements: 3.1_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 12.2 Create layout components
  - Header with navigation
  - Footer
  - Sidebar (for library filters)
  - Responsive layout
  - _Requirements: 3.1_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ]* 12.3 Write tests for components
  - Unit tests for all components
  - Property tests for interactive components
  - React Testing Library
  - _Requirements: 5.3, 5.4_

---

### 13. Web Application Deployment

- [ ] 13.1 Setup Lambda@Edge for SSR
  - Configure React Router for Lambda@Edge
  - Build SSR bundle
  - Test SSR locally
  - _Requirements: 3.7_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 13.2 Create FrontendStack (CDK)
  - S3 bucket for static assets
  - CloudFront distribution
  - Lambda@Edge functions
  - Origin access identity
  - _Requirements: 3.7_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 13.3 Setup GitHub Actions for web deployment
  - Build application
  - Deploy to S3
  - Invalidate CloudFront cache
  - Deploy Lambda@Edge functions
  - _Requirements: 7.1, 7.2, 7.4_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ] 13.4 Deploy web application
  - Deploy FrontendStack
  - Verify public URL is accessible
  - Test all routes
  - Test authentication flow
  - _Requirements: 3.7_
  - _Details: See `web/.kiro/specs/01-mvp/`_

---

### 14. End-to-End Testing

- [ ]* 14.1 Setup Playwright for E2E tests
  - Configure Playwright
  - Setup test database
  - Mock OAuth for testing
  - _Requirements: 5.3_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ]* 14.2 Write E2E tests for critical flows
  - User authentication flow
  - Search → Detail → Add to library flow
  - Read chapter → Mark as read flow
  - Library management flow
  - _Requirements: 5.3, 9.1, 9.2, 9.3, 9.4_
  - _Details: See `web/.kiro/specs/01-mvp/`_

- [ ]* 14.3 Run E2E tests in CI
  - Configure GitHub Actions
  - Run tests on every PR
  - Run tests before deployment
  - _Requirements: 5.5, 7.1_

---

### 15. Checkpoint: Frontend Phase Complete

- [ ] 15.1 Verify web application is fully functional
  - Confirm web app deployed to CloudFront
  - Confirm public URL accessible
  - Confirm all routes working
  - Confirm authentication working
  - Confirm all features working (search, detail, reader, library)
  - E2E tests passing
  - All tests passing (80%+ coverage)
  - _Requirements: 4.3, 4.4, 5.4, 6.3_

---

## Phase 4: MVP Launch (Week 9)

### 16. Production Readiness

- [ ] 16.1 Review and update documentation
  - Update README in all repositories
  - Update API documentation
  - Create user guide
  - Create developer guide
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 16.2 Security audit
  - Review IAM permissions (least privilege)
  - Review authentication flow
  - Review data encryption
  - Review secrets management
  - _Requirements: 5.1, 5.2_

- [ ] 16.3 Performance testing
  - Load test API (100 concurrent users)
  - Measure page load times
  - Measure API latency
  - Optimize if needed
  - _Requirements: 9.8_

- [ ] 16.4 Final smoke tests
  - Test all critical user flows
  - Test on multiple devices
  - Test on multiple browsers
  - Fix any issues found
  - _Requirements: 9.1, 9.2, 9.3, 9.4_

---

### 17. Launch

- [ ] 17.1 Final deployment to production
  - Deploy backend (if not already)
  - Deploy web app (if not already)
  - Verify everything is working
  - _Requirements: 9.6_

- [ ] 17.2 Enable monitoring and alerts
  - Verify CloudWatch dashboards
  - Verify alarms are configured
  - Test alert notifications
  - _Requirements: 9.7_

- [ ] 17.3 Announce MVP launch
  - Publish announcement
  - Share public URL
  - Gather initial feedback
  - _Requirements: 4.5_

---

### 18. Checkpoint: MVP Complete

- [ ] 18.1 Verify MVP success criteria
  - Users can sign in with Google ✓
  - Users can search for manga ✓
  - Users can read manga chapters ✓
  - Users can track reading progress ✓
  - All critical paths tested (80%+ coverage) ✓
  - Deployed to production and accessible ✓
  - Monitoring and error tracking enabled ✓
  - Can handle 100 concurrent users ✓
  - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5, 9.6, 9.7, 9.8_

---

## Post-MVP Backlog

These features are **out of scope** for MVP but planned for future releases:

- [ ] Automatic progress tracking (scroll-based)
- [ ] Email notifications for new chapters
- [ ] Push notifications (browser + mobile)
- [ ] External tracker sync (AniList, MyAnimeList)
- [ ] Mobile app (Flutter)
- [ ] Community features (lists, recommendations, social)
- [ ] Advanced search filters (genre, author, year)
- [ ] Chapter downloads for offline reading
- [ ] Multiple authentication providers (Discord, Passkey)
- [ ] Custom user lists
- [ ] Moderation system

---

## Notes

- Tasks marked with `*` are optional (tests, documentation) but recommended
- Each top-level task references a repository-specific spec for detailed implementation
- Checkpoints must be completed before proceeding to the next phase
- All tests must pass before marking a task as complete
- All documentation must be updated before marking a task as complete
