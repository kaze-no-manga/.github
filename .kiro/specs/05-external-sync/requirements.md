# Requirements Document - Phase 5: External Sync

## Introduction

This document defines the requirements for **Phase 5: External Sync** of Kaze no Manga. This phase integrates with external manga tracking services (AniList and MyAnimeList) to provide bidirectional synchronization of library and reading progress.

**Prerequisites:** MVP (Phase 1) must be complete.

**Goal:** Allow users to sync their manga library and reading progress with AniList and MyAnimeList, enabling seamless integration with existing tracking workflows.

## Glossary

- **AniList**: Popular anime and manga tracking service with GraphQL API
- **MyAnimeList (MAL)**: Popular anime and manga tracking service with REST API
- **Bidirectional Sync**: Two-way synchronization (Kaze ↔ External service)
- **Batch Import**: Importing entire library from external service at once
- **Conflict Resolution**: Handling cases where data differs between services
- **Manga Mapping**: Matching Kaze manga with external service manga
- **OAuth**: Authentication protocol used by external services
- **Rate Limiting**: API request limits imposed by external services
- **Sync Strategy**: Rules for how and when to sync data

## Requirements

### Requirement 1: AniList Integration

**User Story:** As a manga reader, I want to connect my AniList account, so that I can sync my library and progress.

#### Acceptance Criteria

1. WHEN a user connects AniList THEN the system SHALL use OAuth 2.0 for authentication
2. WHEN a user connects AniList THEN the system SHALL request necessary permissions (read library, write library)
3. WHEN a user connects AniList THEN the system SHALL store the access token securely
4. WHEN a user connects AniList THEN the system SHALL display connection status in settings
5. WHEN a user disconnects AniList THEN the system SHALL revoke the access token
6. WHEN AniList token expires THEN the system SHALL automatically refresh it
7. WHEN AniList authentication fails THEN the system SHALL prompt user to reconnect

### Requirement 2: MyAnimeList Integration

**User Story:** As a manga reader, I want to connect my MyAnimeList account, so that I can sync my library and progress.

#### Acceptance Criteria

1. WHEN a user connects MAL THEN the system SHALL use OAuth 2.0 for authentication
2. WHEN a user connects MAL THEN the system SHALL request necessary permissions (read library, write library)
3. WHEN a user connects MAL THEN the system SHALL store the access token securely
4. WHEN a user connects MAL THEN the system SHALL display connection status in settings
5. WHEN a user disconnects MAL THEN the system SHALL revoke the access token
6. WHEN MAL token expires THEN the system SHALL automatically refresh it
7. WHEN MAL authentication fails THEN the system SHALL prompt user to reconnect

### Requirement 3: Batch Import from AniList

**User Story:** As a manga reader, I want to import my entire AniList library, so that I can quickly populate my Kaze library.

#### Acceptance Criteria

1. WHEN a user initiates batch import THEN the system SHALL fetch all manga from AniList library
2. WHEN fetching manga THEN the system SHALL handle pagination (AniList returns 50 per page)
3. WHEN manga are fetched THEN the system SHALL map AniList manga to Kaze manga (by title, MAL ID, or AniList ID)
4. WHEN mapping succeeds THEN the system SHALL add manga to Kaze library with status and progress
5. WHEN mapping fails THEN the system SHALL prompt user to manually map or skip
6. WHEN import completes THEN the system SHALL show summary (X imported, Y skipped, Z failed)
7. WHEN import is in progress THEN the system SHALL show progress indicator

### Requirement 4: Batch Import from MyAnimeList

**User Story:** As a manga reader, I want to import my entire MyAnimeList library, so that I can quickly populate my Kaze library.

#### Acceptance Criteria

1. WHEN a user initiates batch import THEN the system SHALL fetch all manga from MAL library
2. WHEN fetching manga THEN the system SHALL handle pagination (MAL returns 100 per page)
3. WHEN manga are fetched THEN the system SHALL map MAL manga to Kaze manga (by title or MAL ID)
4. WHEN mapping succeeds THEN the system SHALL add manga to Kaze library with status and progress
5. WHEN mapping fails THEN the system SHALL prompt user to manually map or skip
6. WHEN import completes THEN the system SHALL show summary (X imported, Y skipped, Z failed)
7. WHEN import is in progress THEN the system SHALL show progress indicator

### Requirement 5: Manual Sync (Kaze → AniList)

**User Story:** As a manga reader, I want to manually sync my Kaze library to AniList, so that I can update my AniList with my Kaze progress.

#### Acceptance Criteria

1. WHEN a user initiates sync to AniList THEN the system SHALL compare Kaze library with AniList library
2. WHEN differences are found THEN the system SHALL show a preview of changes (X to add, Y to update)
3. WHEN user confirms sync THEN the system SHALL push changes to AniList
4. WHEN pushing changes THEN the system SHALL respect AniList rate limits (90 requests per minute)
5. WHEN sync completes THEN the system SHALL show summary (X added, Y updated, Z failed)
6. WHEN sync fails THEN the system SHALL show error details and allow retry
7. WHEN sync is in progress THEN the system SHALL show progress indicator

### Requirement 6: Manual Sync (Kaze → MyAnimeList)

**User Story:** As a manga reader, I want to manually sync my Kaze library to MyAnimeList, so that I can update my MAL with my Kaze progress.

#### Acceptance Criteria

1. WHEN a user initiates sync to MAL THEN the system SHALL compare Kaze library with MAL library
2. WHEN differences are found THEN the system SHALL show a preview of changes (X to add, Y to update)
3. WHEN user confirms sync THEN the system SHALL push changes to MAL
4. WHEN pushing changes THEN the system SHALL respect MAL rate limits (varies by endpoint)
5. WHEN sync completes THEN the system SHALL show summary (X added, Y updated, Z failed)
6. WHEN sync fails THEN the system SHALL show error details and allow retry
7. WHEN sync is in progress THEN the system SHALL show progress indicator

### Requirement 7: Manual Sync (AniList → Kaze)

**User Story:** As a manga reader, I want to manually sync my AniList library to Kaze, so that I can update my Kaze with my AniList progress.

#### Acceptance Criteria

1. WHEN a user initiates sync from AniList THEN the system SHALL compare AniList library with Kaze library
2. WHEN differences are found THEN the system SHALL show a preview of changes (X to add, Y to update)
3. WHEN user confirms sync THEN the system SHALL pull changes from AniList
4. WHEN pulling changes THEN the system SHALL map AniList manga to Kaze manga
5. WHEN sync completes THEN the system SHALL show summary (X added, Y updated, Z failed)
6. WHEN sync fails THEN the system SHALL show error details and allow retry
7. WHEN sync is in progress THEN the system SHALL show progress indicator

### Requirement 8: Manual Sync (MyAnimeList → Kaze)

**User Story:** As a manga reader, I want to manually sync my MyAnimeList library to Kaze, so that I can update my Kaze with my MAL progress.

#### Acceptance Criteria

1. WHEN a user initiates sync from MAL THEN the system SHALL compare MAL library with Kaze library
2. WHEN differences are found THEN the system SHALL show a preview of changes (X to add, Y to update)
3. WHEN user confirms sync THEN the system SHALL pull changes from MAL
4. WHEN pulling changes THEN the system SHALL map MAL manga to Kaze manga
5. WHEN sync completes THEN the system SHALL show summary (X added, Y updated, Z failed)
6. WHEN sync fails THEN the system SHALL show error details and allow retry
7. WHEN sync is in progress THEN the system SHALL show progress indicator

### Requirement 9: Conflict Resolution

**User Story:** As a manga reader, I want to resolve conflicts when data differs between services, so that I can choose which data to keep.

#### Acceptance Criteria

1. WHEN sync detects conflicts THEN the system SHALL show a conflict resolution screen
2. WHEN conflicts are shown THEN the system SHALL display both versions (Kaze vs External)
3. WHEN conflicts are shown THEN the system SHALL highlight differences (status, progress, rating)
4. WHEN user resolves conflict THEN the system SHALL allow choosing: Keep Kaze, Keep External, or Merge
5. WHEN user chooses "Merge" THEN the system SHALL use most recent data for each field
6. WHEN user resolves all conflicts THEN the system SHALL continue sync
7. WHEN user skips conflict THEN the system SHALL leave data unchanged and continue

### Requirement 10: Manga Mapping

**User Story:** As a manga reader, I want accurate manga mapping between services, so that sync works correctly.

#### Acceptance Criteria

1. WHEN mapping manga THEN the system SHALL first try matching by external ID (MAL ID, AniList ID)
2. WHEN external ID is not available THEN the system SHALL try matching by title (exact match)
3. WHEN exact match fails THEN the system SHALL try fuzzy matching (Levenshtein distance)
4. WHEN fuzzy matching finds multiple candidates THEN the system SHALL prompt user to choose
5. WHEN no match is found THEN the system SHALL prompt user to manually search and map
6. WHEN mapping is confirmed THEN the system SHALL store the mapping for future syncs
7. WHEN mapping is incorrect THEN the system SHALL allow user to remap

### Requirement 11: Automatic Sync (Future)

**User Story:** As a manga reader, I want automatic background sync, so that my libraries stay in sync without manual intervention.

#### Acceptance Criteria

1. WHEN automatic sync is enabled THEN the system SHALL sync daily at user-preferred time
2. WHEN automatic sync runs THEN the system SHALL use last-write-wins strategy for conflicts
3. WHEN automatic sync runs THEN the system SHALL sync bidirectionally (Kaze ↔ External)
4. WHEN automatic sync fails THEN the system SHALL retry with exponential backoff
5. WHEN automatic sync detects conflicts THEN the system SHALL notify user for manual resolution
6. WHEN automatic sync completes THEN the system SHALL log results and update last sync timestamp
7. WHEN automatic sync is disabled THEN the system SHALL only sync manually

### Requirement 12: Sync History

**User Story:** As a manga reader, I want to see my sync history, so that I can track what was synced and when.

#### Acceptance Criteria

1. WHEN a user views sync settings THEN the system SHALL display sync history (last 30 syncs)
2. WHEN displaying sync history THEN the system SHALL show: timestamp, direction, service, status, summary
3. WHEN displaying sync history THEN the system SHALL allow filtering by service (AniList, MAL)
4. WHEN displaying sync history THEN the system SHALL allow filtering by status (success, failed, partial)
5. WHEN a user clicks a sync entry THEN the system SHALL show detailed log (what was synced)
6. WHEN sync fails THEN the system SHALL show error details in history
7. WHEN sync history is old THEN the system SHALL archive entries older than 90 days

### Requirement 13: Rate Limiting & API Quotas

**User Story:** As a developer, I want to respect external API rate limits, so that we don't get blocked.

#### Acceptance Criteria

1. WHEN making AniList requests THEN the system SHALL respect 90 requests per minute limit
2. WHEN making MAL requests THEN the system SHALL respect endpoint-specific rate limits
3. WHEN rate limit is reached THEN the system SHALL queue requests and retry after cooldown
4. WHEN rate limit is exceeded THEN the system SHALL show user-friendly message and estimated wait time
5. WHEN syncing large libraries THEN the system SHALL batch requests to stay within limits
6. WHEN API returns rate limit error THEN the system SHALL parse retry-after header and wait
7. WHEN monitoring API usage THEN the system SHALL track requests per minute and alert if approaching limits

### Requirement 14: Data Mapping

**User Story:** As a developer, I want accurate data mapping between services, so that sync preserves user data correctly.

#### Acceptance Criteria

1. WHEN mapping status THEN the system SHALL map: Reading ↔ Current, Completed ↔ Completed, Plan to Read ↔ Planning, Dropped ↔ Dropped
2. WHEN mapping progress THEN the system SHALL map chapter numbers (Kaze chapters ↔ External chapters)
3. WHEN mapping rating THEN the system SHALL convert between rating scales (Kaze 1-10 ↔ AniList 1-10 ↔ MAL 1-10)
4. WHEN mapping dates THEN the system SHALL preserve timestamps (started, completed)
5. WHEN mapping notes THEN the system SHALL sync notes/comments (if supported by external service)
6. WHEN external service has fields not in Kaze THEN the system SHALL ignore them
7. WHEN Kaze has fields not in external service THEN the system SHALL not sync them

### Requirement 15: Error Handling

**User Story:** As a manga reader, I want clear error messages when sync fails, so that I know what went wrong and how to fix it.

#### Acceptance Criteria

1. WHEN authentication fails THEN the system SHALL show message: "Please reconnect your [Service] account"
2. WHEN network fails THEN the system SHALL show message: "Network error. Please check your connection and try again"
3. WHEN rate limit is hit THEN the system SHALL show message: "Rate limit reached. Please wait X minutes and try again"
4. WHEN manga mapping fails THEN the system SHALL show message: "Could not find [Manga] on [Service]. Please map manually"
5. WHEN API returns error THEN the system SHALL show user-friendly message (not raw API error)
6. WHEN sync partially fails THEN the system SHALL show which items failed and allow retry
7. WHEN errors are logged THEN the system SHALL include sufficient context for debugging

### Requirement 16: Sync Settings

**User Story:** As a manga reader, I want to configure sync behavior, so that I can control how sync works.

#### Acceptance Criteria

1. WHEN a user views sync settings THEN the system SHALL show connected services (AniList, MAL)
2. WHEN a user views sync settings THEN the system SHALL show last sync timestamp for each service
3. WHEN a user views sync settings THEN the system SHALL allow enabling/disabling automatic sync
4. WHEN a user views sync settings THEN the system SHALL allow choosing sync direction (bidirectional, Kaze → External, External → Kaze)
5. WHEN a user views sync settings THEN the system SHALL allow choosing conflict resolution strategy (manual, last-write-wins, Kaze wins, External wins)
6. WHEN a user views sync settings THEN the system SHALL allow choosing what to sync (status, progress, rating, notes)
7. WHEN settings are changed THEN the system SHALL apply them immediately

### Requirement 17: Performance & Optimization

**User Story:** As a manga reader, I want sync to be fast, so that I don't have to wait long.

#### Acceptance Criteria

1. WHEN syncing small libraries (<100 manga) THEN the system SHALL complete within 30 seconds
2. WHEN syncing large libraries (>1000 manga) THEN the system SHALL complete within 5 minutes
3. WHEN syncing THEN the system SHALL use parallel requests (respecting rate limits)
4. WHEN syncing THEN the system SHALL cache external API responses for 1 hour
5. WHEN syncing THEN the system SHALL only sync changed items (not entire library every time)
6. WHEN syncing THEN the system SHALL use delta sync (compare timestamps to detect changes)
7. WHEN syncing THEN the system SHALL show progress indicator with estimated time remaining

### Requirement 18: Testing & Quality

**User Story:** As a developer, I want comprehensive tests for sync features, so that sync is reliable.

#### Acceptance Criteria

1. WHEN sync code is written THEN the system SHALL include unit tests for all functions
2. WHEN sync code is written THEN the system SHALL include property-based tests for data mapping
3. WHEN sync code is written THEN the system SHALL include integration tests with mocked APIs
4. WHEN sync code is written THEN the system SHALL achieve 80% test coverage
5. WHEN testing sync THEN the system SHALL test conflict resolution scenarios
6. WHEN testing sync THEN the system SHALL test rate limiting behavior
7. WHEN deploying to production THEN the system SHALL run all tests in CI pipeline

### Requirement 19: Security & Privacy

**User Story:** As a user, I want my external service credentials to be secure, so that my accounts are protected.

#### Acceptance Criteria

1. WHEN storing access tokens THEN the system SHALL encrypt them at rest
2. WHEN transmitting tokens THEN the system SHALL use HTTPS only
3. WHEN tokens are no longer needed THEN the system SHALL revoke them
4. WHEN user disconnects service THEN the system SHALL delete all stored tokens
5. WHEN logging sync activity THEN the system SHALL not log access tokens or sensitive data
6. WHEN displaying sync data THEN the system SHALL not expose tokens in UI
7. WHEN user deletes account THEN the system SHALL revoke all external service connections

### Requirement 20: Documentation

**User Story:** As a manga reader, I want clear documentation on how to use sync features, so that I can set them up correctly.

#### Acceptance Criteria

1. WHEN a user views sync settings THEN the system SHALL provide a "How to Sync" guide
2. WHEN a user connects a service THEN the system SHALL show step-by-step instructions
3. WHEN a user encounters errors THEN the system SHALL provide troubleshooting tips
4. WHEN a user views documentation THEN the system SHALL explain conflict resolution strategies
5. WHEN a user views documentation THEN the system SHALL explain data mapping (what syncs and what doesn't)
6. WHEN a user views documentation THEN the system SHALL provide FAQ for common issues
7. WHEN documentation is updated THEN the system SHALL version it for reference
