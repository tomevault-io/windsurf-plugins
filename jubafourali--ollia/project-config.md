---
trigger: always_on
description: **Ollia** is a family reassurance mobile app (Expo/React Native) backed by a Kotlin/Spring Boot REST API. The app enables lightweight check-ins and real-time family connectivity through activity signals, family circles, and safety event alerts.
---

# Ollia AI Agent Guidelines

## Architecture Overview

**Ollia** is a family reassurance mobile app (Expo/React Native) backed by a Kotlin/Spring Boot REST API. The app enables lightweight check-ins and real-time family connectivity through activity signals, family circles, and safety event alerts.

- **Backend**: `apps/backend/` — Spring Boot (Kotlin), PostgreSQL, Flyway migrations
- **Mobile**: `apps/mobile/` — Expo/React Native, TypeScript
- **Authentication**: Clerk (OAuth2 resource server)
- **Database**: PostgreSQL with Flyway version control

## Key Data Flows

### 1. Activity Signal Pipeline
- **Mobile** → sends heartbeat/check-in/shortcut signals → **POST /api/activity**
- Backend records signal to `activity_signals` table
- Simultaneously updates `users.lastSeenAt` and optionally resets escalation/nudge flags
- **Pattern Analysis**: 30-day rolling window aggregates signals by hour for streak/distribution metrics
- **Query Patterns**: `findAllByUserIdAndCreatedAtAfterOrderByCreatedAtDesc()` (frequent, time-windowed)

### 2. Family Circle & Member Management
- Users create circles (`POST /api/circles`) — one circle per user owner
- Members join via invite code or direct add (`POST /api/circles/join`)
- Free plan: 3 member cap; premium: unlimited
- **Query Patterns**: 
  - `findAllByCircleId()` / `findAllByUserId()` (member enumeration)
  - `findByOwnerId()` (single circle lookup)
  - `countByCircleId()` (member limit enforcement)

### 3. Safety Events & Regional Alerts
- **Background scheduler** fetches every 15 min from USGS, NOAA, GDACS
- Events stored with region, type, severity, lat/lon for mapping
- Cleanup: removes events older than 48 hours
- **Query Patterns**:
  - `findAllByRegionIgnoreCaseAndFetchedAtAfterOrderByEventTimeDesc(region, since)` (regional filtering + time window)
  - `findAllByFetchedAtAfterOrderByEventTimeDesc(since)` (recent events, no filter)
  - `deleteBySource(source)` (bulk replace on scheduler run)
  - `deleteAllByFetchedAtBefore(cutoff)` (old data cleanup)

### 4. Push Notification Lifecycle
- Tokens registered via `POST /api/push-tokens` (upsert per user)
- **Notification Log** tracks all send attempts (success/failure, type, timestamp)
- Admin views logs filtered by type or time range
- **Query Patterns**:
  - `findAllByUserIdOrderBySentAtDesc()` (user's notification history)
  - `findAllByNotificationTypeOrderBySentAtDesc()` (admin filtering by type)
  - `findAllBySentAtAfterOrderBySentAtDesc(since)` (recent notifications, no user filter)

## Database Schema & Index Recommendations

### Current Indexes (✅ = good, ⚠️ = missing)

#### users
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (clerk_id) — lookup by auth ID
✅ idx_users_clerk_id — redundant with UNIQUE, but explicit
✅ idx_users_last_seen_at — inactivity detection queries
⚠️ MISSING: (escalation_level, last_seen_at) — compound for escalation queries
⚠️ MISSING: (plan) — premium vs free filtering not indexed
⚠️ MISSING: (founding_member, founding_expires_at) — effective plan queries
```

#### activity_signals
```sql
✅ PRIMARY KEY (id)
✅ idx_activity_signals_user_id — lookups per user
⚠️ MISSING: (user_id, created_at DESC) — compound index for time-windowed queries
   Query: findAllByUserIdAndCreatedAtAfterOrderByCreatedAtDesc() — very common in pattern analysis
```

#### family_circles
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (invite_code) — join circle lookup
⚠️ MISSING: (owner_id) — list circles by owner
⚠️ MISSING: (plan) — filter circles by subscription tier
```

#### family_members
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (circle_id, user_id) — uniqueness constraint (good)
✅ idx_family_members_circle_id — list members in circle
✅ idx_family_members_user_id — list circles user is in
```

#### family_invites
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (token) — redundant with idx_family_invites_token but good
✅ idx_family_invites_token — lookup by token
⚠️ MISSING: (circle_id) — bulk deletion by circle needs index
⚠️ MISSING: (created_by) — user's invites list
⚠️ MISSING: (expires_at) — cleanup queries for expired invites
```

#### push_tokens
```sql
✅ PRIMARY KEY (id)
✅ UNIQUE (user_id) — enforced by migration, upsert requires uniqueness
✅ idx_push_tokens_user_id — lookup token by user
⚠️ MISSING: (platform) — filter by iOS vs Android for bulk sends
```

#### safety_events
```sql
✅ PRIMARY KEY (id)
✅ idx_safety_events_region — region-filtered queries
✅ idx_safety_events_fetched_at — cleanup queries
✅ idx_safety_events_type — admin filtering
⚠️ MISSING: (fetched_at DESC, region) — regional queries with ordering
⚠️ MISSING: (source) — deleteBySource() does sequential scan
```

#### notification_log
```sql
✅ PRIMARY KEY (id)
✅ idx_notification_log_user_id — user's history
✅ idx_notification_log_sent_at DESC — recent notifications (good DESC)
✅ idx_notification_log_type — admin filtering by notification type
⚠️ MISSING: (user_id, sent_at DESC) — compound for user's recent notifications
⚠️ MISSING: (notification_type, sent_at DESC) — compound for type + ordering
⚠️ MISSING: (status) — failed notification tracking
```

## Recommended Index Migration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jubafourali/ollia](https://github.com/jubafourali/ollia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
