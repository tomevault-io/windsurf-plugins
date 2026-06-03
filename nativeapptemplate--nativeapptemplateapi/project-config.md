---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rails 8.1 API application that serves as the backend for NativeAppTemplate iOS/Android mobile applications. It's a multi-tenant SaaS application with token-based authentication, role-based authorization, and RESTful API endpoints. Ruby 4.0.2, PostgreSQL, Solid Queue/Cable/Cache.

## Development Commands

### Initial Setup
```bash
bin/setup  # Installs all dependencies, prepares database, builds assets
```

### Running the Application
```bash
bin/dev  # Starts Rails server, CSS watcher, JS bundler
```

### Testing
```bash
bin/rails test                  # Run all tests
bin/rails test test/path/to/test.rb  # Run specific test file
bin/rails test test/path/to/test.rb:42  # Run specific test line
```

### Linting & Security
```bash
bin/rubocop                     # Ruby code linting
bundle exec erb_lint --lint-all # ERB template linting
bin/brakeman                    # Security vulnerability scanning
bin/bundler-audit               # Audit gems for known security defects
```

### Local CI
```bash
bin/ci  # Runs setup, rubocop, bundler-audit, brakeman, tests, and seeds
```

### Database Operations
```bash
bin/rails db:create db:migrate  # Create and migrate database
bin/rails db:seed_fu           # Load seed data (uses seed-fu gem)
bin/rails db:reset             # Drop, create, migrate, and seed
```

### Console & Debugging
```bash
bin/rails console              # Rails console
bin/rails dbconsole           # Database console
```

## Architecture & Key Concepts

### API Structure
- All API endpoints are under `/api/v1/` namespace
- Token-based authentication using `devise_token_auth`
- Separate namespaces for different user types (e.g., `/api/v1/shopkeeper/`)
- JSON API specification for responses using `jsonapi-serializer`
- CORS enabled for cross-origin requests

### Authentication & Authorization
- **Authentication**: Devise Token Auth with headers-based token management
- **Authorization**: Pundit policies for resource-level permissions
- **Multi-tenancy**: acts_as_tenant for complete data isolation between accounts
- **RBAC**: Role and Permission models for fine-grained access control

### Key Models & Relationships
- `Account` - Top-level tenant/organization
- `Shopkeeper` - Main user type (belongs to Account)
- `Shop` - Core business entity (belongs to Account)
- `ItemTag` - Belongs to Shop; has name/description/position and a binary state (idled/completed)
- `Role` & `Permission` - Authorization system
- State machines implemented with AASM gem

### Background Processing
- Solid Queue for background jobs (database-backed, no Redis needed)
- Solid Cable for Action Cable (database-backed)
- Solid Cache for caching in production/staging
- Monitor jobs at `/madmin/jobs` (Mission Control)

### Push Notifications
Cross-platform push via the `noticed` (v3) and `action_push_native` gems. APNs for iOS, FCM for Android.

- **Provider names, not OS names**: the device `platform` is `apple` (APNs) or `google` (FCM) — never `ios`/`android`. The enum (`validate: true`) rejects anything else with 422 "Platform is not included in the list".
- **Models**: `ApplicationPushDevice < ActionPushNative::Device` (`action_push_native_devices` table; polymorphic `owner`, unique on `[platform, token]`). `ApplicationPushNotification < ActionPushNative::Notification`. Delivery records live in `noticed_events` / `noticed_notifications`. All visible in `/madmin`.
- **Device registration**: `POST /api/v1/shopkeeper/devices` is idempotent on `(platform, token)` — re-POST updates `last_active_at`; a token bound to another shopkeeper is reassigned.
- **Notifiers**: subclass `ApplicationNotifier` (`Noticed::Event`). `ItemTagNotifier` fires from the AASM `complete` event. In `deliver_by :action_push_native`, the `with_apple`, `with_google`, and `with_data` options must each return a **Hash** (use `{}` when empty) — passing `nil` raises `TypeError: no implicit conversion of nil into Hash`.
- **Config & secrets**: `config/push.yml` reads everything from credentials (`bin/rails credentials:edit --environment <env>`) under `action_push_native:apns:{key_id, team_id, topic, encryption_key}` and `action_push_native:fcm:{project_id, encryption_key}`. APNs `encryption_key` is the full `.p8` PEM; FCM `encryption_key` is the entire service-account JSON. `topic` stays in credentials (not source) because the agent's rename pipeline would otherwise desync the bundle id.
- **APNs sandbox vs production**: `connect_to_development_server: <%= Rails.env.development? %>`. The token's environment is fixed at iOS build time — Xcode debug builds get **sandbox** tokens, TestFlight/App Store get **production**. They must match the server: test Xcode debug builds against a **local development** server; use TestFlight to test against Render. A mismatch returns APNs `400 BadDeviceToken`, which the gem treats as `TokenError` and **destroys the device row** (the default `rescue_from`). FCM has no such split.

### Testing Strategy
- Minitest for all tests (models, controllers, integration, policies)
- WebMock for stubbing external HTTP requests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nativeapptemplate/nativeapptemplateapi](https://github.com/nativeapptemplate/nativeapptemplateapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
