# Requirements Document - Phase 2: Automation

## Introduction

This document defines the requirements for **Phase 2: Automation** of Kaze no Manga. This phase builds on the MVP by adding automated background jobs, notifications, and intelligent progress tracking to eliminate manual work and keep users engaged.

**Prerequisites:** MVP (Phase 1) must be complete before starting this phase.

**Goal:** Transform Kaze no Manga from a manual tracking tool into an intelligent, proactive system that automatically monitors manga updates and tracks reading progress.

## Glossary

- **Background Job**: Automated task that runs on a schedule (e.g., daily chapter checks)
- **Step Function**: AWS service for orchestrating workflows
- **EventBridge**: AWS service for scheduling events
- **SQS**: AWS Simple Queue Service for job queuing
- **SES**: AWS Simple Email Service for sending emails
- **SNS**: AWS Simple Notification Service for push notifications
- **FCM**: Firebase Cloud Messaging for web and Android push notifications
- **APNS**: Apple Push Notification Service for iOS push notifications
- **Scroll Tracking**: Detecting when a user has scrolled to the end of a chapter
- **Chapter Check**: Automated process to check if new chapters are available for a manga

## Requirements

### Requirement 1: Daily Chapter Checks

**User Story:** As a manga reader, I want the system to automatically check for new chapters daily, so that I don't have to manually check each manga source.

#### Acceptance Criteria

1. WHEN the system runs daily chapter checks THEN the system SHALL check all manga in users' libraries for new chapters
2. WHEN a manga has multiple sources THEN the system SHALL check all sources and aggregate results
3. WHEN new chapters are found THEN the system SHALL store them in the database and mark them as unread
4. WHEN a manga is marked as "completed" THEN the system SHALL check it less frequently (weekly instead of daily)
5. WHEN a manga is marked as "ongoing" THEN the system SHALL check it daily
6. WHEN a source fails to respond THEN the system SHALL retry with exponential backoff and log the failure
7. WHEN chapter checks complete THEN the system SHALL update the last_checked timestamp for each manga source

### Requirement 2: Email Notifications

**User Story:** As a manga reader, I want to receive email notifications when new chapters are available, so that I can read them immediately.

#### Acceptance Criteria

1. WHEN new chapters are found for manga in a user's library THEN the system SHALL send an email notification to the user
2. WHEN multiple chapters are available for the same manga THEN the system SHALL aggregate them into a single email
3. WHEN multiple manga have new chapters THEN the system SHALL send a single digest email (not one per manga)
4. WHEN a user has notification preferences THEN the system SHALL respect per-manga notification settings
5. WHEN a user sets a daily notification limit THEN the system SHALL not exceed that limit
6. WHEN an email fails to send THEN the system SHALL retry up to 3 times and log the failure
7. WHEN a user clicks "unsubscribe" in an email THEN the system SHALL disable email notifications for that user

### Requirement 3: Push Notifications (Web + Mobile)

**User Story:** As a manga reader, I want to receive push notifications on my devices when new chapters are available, so that I'm notified even when not checking email.

#### Acceptance Criteria

1. WHEN new chapters are found for manga in a user's library THEN the system SHALL send push notifications to all registered devices
2. WHEN a user grants notification permission THEN the system SHALL register the device token with SNS
3. WHEN a user revokes notification permission THEN the system SHALL unregister the device token
4. WHEN multiple chapters are available THEN the system SHALL send a single aggregated notification
5. WHEN a notification is clicked THEN the system SHALL open the manga detail page or reader
6. WHEN a push notification fails THEN the system SHALL retry up to 3 times and log the failure
7. WHEN a device token is invalid THEN the system SHALL remove it from the database

### Requirement 4: Notification Preferences

**User Story:** As a manga reader, I want granular control over notifications, so that I only receive notifications for manga I care about.

#### Acceptance Criteria

1. WHEN a user adds manga to their library THEN the system SHALL default to notifications enabled
2. WHEN a user views notification settings THEN the system SHALL display per-manga notification toggles
3. WHEN a user disables notifications for a manga THEN the system SHALL not send notifications for that manga
4. WHEN a user sets a daily notification limit THEN the system SHALL enforce that limit across all manga
5. WHEN a user sets notification frequency (immediate, daily digest, weekly digest) THEN the system SHALL respect that preference
6. WHEN a user disables all notifications THEN the system SHALL not send any notifications
7. WHEN notification preferences are updated THEN the system SHALL apply changes immediately

### Requirement 5: Automatic Progress Tracking

**User Story:** As a manga reader, I want my progress to be tracked automatically when I finish reading a chapter, so that I don't have to manually mark chapters as read.

#### Acceptance Criteria

1. WHEN a user scrolls to the end of a chapter THEN the system SHALL automatically mark that chapter as read
2. WHEN a user scrolls to the end of a chapter THEN the system SHALL wait 2 seconds before marking as read (to avoid accidental marks)
3. WHEN a chapter is automatically marked as read THEN the system SHALL update the reading history with a timestamp
4. WHEN a chapter is automatically marked as read THEN the system SHALL update the user's library progress
5. WHEN a user manually marks a chapter as read THEN the system SHALL override automatic tracking
6. WHEN a user disables automatic tracking THEN the system SHALL only use manual marking
7. WHEN automatic tracking is enabled THEN the system SHALL show a visual indicator when a chapter is about to be marked as read

### Requirement 6: Image Download to S3

**User Story:** As a system administrator, I want manga chapter images stored in S3, so that we're not dependent on external sources and can optimize image delivery.

#### Acceptance Criteria

1. WHEN a chapter is first accessed THEN the system SHALL download all images from the source and store them in S3
2. WHEN images are downloaded THEN the system SHALL store them in the format: `s3://kaze-images/{mangaId}/{chapterId}/{pageNumber}.jpg`
3. WHEN images are stored in S3 THEN the system SHALL serve them via CloudFront CDN
4. WHEN an image download fails THEN the system SHALL retry up to 3 times with exponential backoff
5. WHEN all retries fail THEN the system SHALL fall back to serving the original source URL
6. WHEN images are stored THEN the system SHALL optimize them (WebP conversion for modern browsers)
7. WHEN images are older than 1 year THEN the system SHALL transition them to S3 Infrequent Access storage class

### Requirement 7: Background Job Infrastructure

**User Story:** As a developer, I want a robust background job infrastructure, so that automated tasks run reliably and can be monitored.

#### Acceptance Criteria

1. WHEN a background job is scheduled THEN the system SHALL use AWS Step Functions for orchestration
2. WHEN a job needs to be queued THEN the system SHALL use SQS for reliable message delivery
3. WHEN a job fails THEN the system SHALL retry with exponential backoff (max 3 retries)
4. WHEN all retries fail THEN the system SHALL move the job to a dead letter queue (DLQ)
5. WHEN jobs are running THEN the system SHALL log execution details to CloudWatch
6. WHEN job execution time exceeds threshold THEN the system SHALL trigger an alarm
7. WHEN jobs complete THEN the system SHALL record metrics (duration, success rate, error rate)

### Requirement 8: Job Scheduling

**User Story:** As a system administrator, I want flexible job scheduling, so that background tasks run at optimal times.

#### Acceptance Criteria

1. WHEN the system schedules daily chapter checks THEN the system SHALL run them at 00:00 UTC
2. WHEN the system schedules email digests THEN the system SHALL run them at user-preferred times
3. WHEN the system schedules image downloads THEN the system SHALL run them during off-peak hours
4. WHEN a job is scheduled THEN the system SHALL use EventBridge for reliable scheduling
5. WHEN a scheduled job fails THEN the system SHALL retry on the next schedule
6. WHEN job schedules need to change THEN the system SHALL allow dynamic schedule updates
7. WHEN multiple jobs are scheduled THEN the system SHALL prevent overlapping executions

### Requirement 9: Notification Templates

**User Story:** As a user, I want well-designed notification emails, so that they're easy to read and actionable.

#### Acceptance Criteria

1. WHEN an email is sent THEN the system SHALL use HTML templates with responsive design
2. WHEN an email contains new chapters THEN the system SHALL include manga cover, title, and chapter list
3. WHEN an email is sent THEN the system SHALL include direct links to read each chapter
4. WHEN an email is sent THEN the system SHALL include an unsubscribe link
5. WHEN an email is sent THEN the system SHALL include the Kaze no Manga branding (logo, colors)
6. WHEN an email is sent THEN the system SHALL include a plain text fallback for email clients that don't support HTML
7. WHEN email templates are updated THEN the system SHALL version them for rollback capability

### Requirement 10: Performance & Scalability

**User Story:** As a system administrator, I want the automation system to scale efficiently, so that it can handle growth without performance degradation.

#### Acceptance Criteria

1. WHEN the system checks chapters for 10,000 manga THEN the system SHALL complete within 1 hour
2. WHEN the system sends notifications to 10,000 users THEN the system SHALL complete within 30 minutes
3. WHEN the system downloads images THEN the system SHALL process them in parallel (max 10 concurrent downloads per manga)
4. WHEN the system experiences high load THEN the system SHALL use SQS to queue jobs and process them gradually
5. WHEN Lambda functions execute THEN the system SHALL complete within timeout limits (max 15 minutes)
6. WHEN database queries are slow THEN the system SHALL use indexes and query optimization
7. WHEN costs increase THEN the system SHALL implement cost optimization (batch processing, caching)

### Requirement 11: Monitoring & Observability

**User Story:** As a developer, I want comprehensive monitoring of automation jobs, so that I can quickly identify and fix issues.

#### Acceptance Criteria

1. WHEN jobs run THEN the system SHALL log execution details (start time, end time, duration, status)
2. WHEN jobs fail THEN the system SHALL log error details (error message, stack trace, context)
3. WHEN job metrics are collected THEN the system SHALL track success rate, error rate, and duration
4. WHEN job performance degrades THEN the system SHALL trigger CloudWatch alarms
5. WHEN alarms trigger THEN the system SHALL send notifications to the development team
6. WHEN investigating issues THEN the system SHALL provide correlation IDs for request tracing
7. WHEN viewing dashboards THEN the system SHALL display real-time job status and metrics

### Requirement 12: Error Handling & Recovery

**User Story:** As a system administrator, I want robust error handling, so that temporary failures don't cause permanent data loss.

#### Acceptance Criteria

1. WHEN a source API is down THEN the system SHALL skip that source and continue with others
2. WHEN a database connection fails THEN the system SHALL retry with exponential backoff
3. WHEN an email fails to send THEN the system SHALL queue it for retry
4. WHEN a push notification fails THEN the system SHALL queue it for retry
5. WHEN image download fails THEN the system SHALL fall back to serving original URLs
6. WHEN a job fails after all retries THEN the system SHALL move it to DLQ for manual investigation
7. WHEN errors are logged THEN the system SHALL include sufficient context for debugging

### Requirement 13: Testing & Quality

**User Story:** As a developer, I want comprehensive tests for automation features, so that I can deploy with confidence.

#### Acceptance Criteria

1. WHEN automation code is written THEN the system SHALL include unit tests for all functions
2. WHEN automation code is written THEN the system SHALL include property-based tests for critical logic
3. WHEN automation code is written THEN the system SHALL include integration tests for external services (mocked)
4. WHEN automation code is written THEN the system SHALL achieve 80% test coverage
5. WHEN Step Functions are defined THEN the system SHALL include tests for workflow logic
6. WHEN notification templates are created THEN the system SHALL include visual regression tests
7. WHEN deploying to production THEN the system SHALL run all tests in CI pipeline

### Requirement 14: User Experience

**User Story:** As a manga reader, I want automation features to enhance my experience without being intrusive.

#### Acceptance Criteria

1. WHEN automatic progress tracking is enabled THEN the system SHALL show a subtle visual indicator
2. WHEN a chapter is about to be marked as read THEN the system SHALL show a countdown (2 seconds)
3. WHEN notifications are sent THEN the system SHALL not spam users (respect daily limits)
4. WHEN notification preferences are changed THEN the system SHALL apply changes immediately
5. WHEN viewing notification history THEN the system SHALL display past notifications and their status
6. WHEN a notification is clicked THEN the system SHALL deep link to the relevant content
7. WHEN automation features fail THEN the system SHALL gracefully degrade (manual tracking still works)

### Requirement 15: Security & Privacy

**User Story:** As a user, I want my notification preferences and reading data to be secure and private.

#### Acceptance Criteria

1. WHEN notification tokens are stored THEN the system SHALL encrypt them at rest
2. WHEN emails are sent THEN the system SHALL not include sensitive user data in URLs
3. WHEN push notifications are sent THEN the system SHALL not include sensitive data in the payload
4. WHEN users unsubscribe THEN the system SHALL immediately stop sending notifications
5. WHEN device tokens are invalid THEN the system SHALL remove them from the database
6. WHEN accessing notification preferences THEN the system SHALL require authentication
7. WHEN logging notification data THEN the system SHALL not log personally identifiable information (PII)
