---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: YorYor Dating Application

A comprehensive Muslim dating and matchmaking platform built with Laravel 12, emphasizing cultural values, family involvement, and serious relationships. The platform modernizes traditional matchmaking while respecting Islamic cultural and religious values.

**Tech Stack:** Laravel 12, PHP 8.2+, Tailwind CSS 4.0, Laravel Reverb (WebSocket), Laravel Sanctum (API), PostgreSQL

**Core Purpose:** Islamic/Cultural Dating & Matchmaking with family involvement and serious commitment to marriage.

## Essential Commands

### Development Server
```bash
# Start all services (Laravel server + queue worker + logs + Vite)
composer dev

# Individual services
php artisan serve                    # Laravel server (port 8000)
php artisan reverb:start            # WebSocket server (port 8080)
php artisan queue:listen --tries=1   # Queue worker
php artisan pail --timeout=0        # Real-time logs
npm run dev                          # Vite dev server
```

### Database
```bash
# Run migrations
php artisan migrate

# Fresh database with seeders
php artisan migrate:fresh --seed

# Create new migration
php artisan make:migration create_table_name

# Rollback last migration
php artisan migrate:rollback
```

### Testing
```bash
# Run all tests (Pest)
php artisan test

# Run specific test file
php artisan test tests/Feature/AuthTest.php

# Run tests with coverage
php artisan test --coverage

# Code formatting (Laravel Pint)
./vendor/bin/pint
```

### Cache & Optimization
```bash
# Clear all caches
php artisan optimize:clear

# Production optimization
php artisan optimize
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Monitoring Tools
```bash
# Access Laravel Telescope (development debugging)
# http://localhost:8000/telescope

# Access Laravel Pulse (performance monitoring)
# http://localhost:8000/pulse

# Access Laravel Horizon (queue monitoring)
php artisan horizon
# http://localhost:8000/horizon

# Access API documentation (Swagger)
# http://localhost:8000/api/documentation
```

## Architecture Overview

### Layered Architecture Pattern

**YorYor follows a strict layered architecture:**

1. **Presentation Layer**: Blade views + API Resources (JSON:API format)
2. **Application Layer**: Controllers (API + Web) orchestrate requests
3. **Service Layer**: Business logic in dedicated service classes (25+ services)
4. **Domain Layer**: Eloquent models (55+ models) with relationships
5. **Data Access Layer**: Database, Cache, Queue

### Key Architectural Principles

**Service Layer Pattern:**
- **All business logic lives in services**, not controllers
- Controllers validate input, call services, return responses
- Services are injected via dependency injection
- Example: `AuthService`, `MediaUploadService`, `VerificationService`

**Example Service Usage:**
```php
// Controller only orchestrates
public function __construct(
    private AuthService $authService,
    private NotificationService $notificationService
) {}

public function register(Request $request) {
    $validated = $request->validate([...]);
    $result = $this->authService->register($validated);
    $this->notificationService->sendWelcomeEmail($result['user']);
    return response()->json($result);
}
```

**Real-Time with Laravel Reverb:**
- WebSocket server runs on port 8080 (separate from Laravel)
- Broadcasting uses private channels: `private-chat.{chatId}`, `private-user.{userId}`
- Echo client configured in `resources/js/echo.js`
- Events must implement `ShouldBroadcast` interface

### Critical Architecture Rules

1. **Never put business logic in controllers** - always use services
2. **Never query models directly in views** - pass data from controllers
3. **Always use transactions for multi-model operations** - wrap in `DB::beginTransaction()`
4. **Route organization**:
   - `routes/api.php` - Mobile/API endpoints (Sanctum auth)
   - `routes/web.php` - Landing pages (public)
   - `routes/user.php` - Authenticated user routes (web middleware)
   - `routes/admin.php` - Admin dashboard routes (admin middleware)

## Database Architecture

### 70+ Tables Organized by Domain

**Database:** PostgreSQL (default connection configured)

**Core Pattern: User → Profile → Extended Profiles**
- `users` (authentication, basic info - password varchar(255) for future-proof hashing)
- `profiles` (core profile data with soft deletes)
- Extended profiles: `user_cultural_profiles`, `user_career_profiles`, `user_physical_profiles`, `user_family_preferences`, `user_location_preferences`

**Communication: Chat → Messages → Reads**
- `chats` (conversations)
- `chat_users` (pivot: participants with indexes)
- `messages` (chat messages)
- `message_reads` (read receipts)
- `calls` (video/voice calls linked to messages)

**Matching: Likes → Matches**
- `likes` (one-way interest)
- `dislikes` (pass/reject)
- `matches` (mutual likes - created automatically, with soft deletes)

**Reporting System:**
- `user_reports` (comprehensive reporting with severity, priority, evidence)
- `report_evidence` (file attachments for reports)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hitcklife) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
