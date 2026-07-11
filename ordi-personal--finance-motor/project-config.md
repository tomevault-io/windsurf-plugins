---
trigger: always_on
description: Purpose: provide short, actionable guidance so Copilot suggestions match project conventions.
---


# Copilot instructions (English — concise)

Purpose: provide short, actionable guidance so Copilot suggestions match project conventions.

## Common Development Commands

### Development Server
- `bin/dev` - Start development server (Rails, Sidekiq, Tailwind CSS watcher)
- `bin/rails server` - Start Rails server only
- `bin/rails console` - Open Rails console

### Testing
- `bin/rails test` - Run all tests
- `bin/rails test:db` - Run tests with database reset
- `DISABLE_PARALLELIZATION=true bin/rails test:system` - Run system tests only (use sparingly - they take longer)
- `bin/rails test test/models/account_test.rb` - Run specific test file
- `bin/rails test test/models/account_test.rb:42` - Run specific test at line

### Linting & Formatting
- `bin/rubocop` - Run Ruby linter
- `npm run lint` - Check JavaScript/TypeScript code
- `npm run lint:fix` - Fix JavaScript/TypeScript issues
- `npm run format` - Format JavaScript/TypeScript code
- `bin/brakeman` - Run security analysis

### Database
- `bin/rails db:prepare` - Create and migrate database
- `bin/rails db:migrate` - Run pending migrations
- `bin/rails db:rollback` - Rollback last migration
- `bin/rails db:seed` - Load seed data

### Setup
- `bin/setup` - Initial project setup (installs dependencies, prepares database)

## Pre-PR workflow (run locally before opening PR)
- Tests: bin/rails test (all), DISABLE_PARALLELIZATION=true bin/rails test:system (when applicable)
- Linters: bin/rubocop -f github -a; bundle exec erb_lint ./app/**/*.erb -a
- Security: bin/brakeman --no-pager

## High-Level Architecture

### Application Modes
The app runs in two modes:
- **Managed** (Rails.application.config.app_mode = "managed")
- **Self-hosted** (Rails.application.config.app_mode = "self_hosted")

### Core Domain Model
The application is built around financial data management with these key relationships:
- **User** → has many **Accounts** → has many **Transactions**
- **Account** types: checking, savings, credit cards, investments, crypto, loans, properties
- **Transaction** → belongs to **Category**, can have **Tags** and **Rules**
- **Investment accounts** → have **Holdings** → track **Securities** via **Trades**

### API Architecture
The application provides both internal and external APIs:
- Internal API: Controllers serve JSON via Turbo for SPA-like interactions
- External API: `/api/v1/` namespace with Doorkeeper OAuth and API key authentication
- API responses use Jbuilder templates for JSON rendering.
- Rate limiting via Rack::Attack with configurable limits per API key

### Sync & Import System
Two primary data ingestion methods:
1. **Plaid Integration**: Real-time bank account syncing
   - `PlaidItem` manages connections
   - `Sync` tracks sync operations
   - Background jobs handle data updates
2. **CSV Import**: Manual data import with mapping
   - `Import` manages import sessions
   - Supports transaction and balance imports
   - Custom field mapping with transformation rules

### Background Processing
Sidekiq handles asynchronous tasks:
- Account syncing (`SyncJob`)
- Import processing (`ImportJob`)
- AI chat responses (`AssistantResponseJob`)
- Scheduled maintenance via sidekiq-cron

### Frontend Architecture
- **Hotwire Stack**: Turbo + Stimulus for reactive UI without heavy JavaScript
- **ViewComponents**: Reusable UI components in `app/components/`
- **Stimulus Controllers**: Handle interactivity, organized alongside components
- **Charts**: D3.js for financial visualizations (time series, donut, sankey)
- **Styling**: Tailwind CSS v4.x with custom design system
  - Design system defined in `app/assets/tailwind/sure-design-system.css`
  - Always use functional tokens (e.g., `text-primary` not `text-white`)
  - Prefer semantic HTML elements over JS components
  - Use `icon` helper for icons, never `lucide_icon` directly

### Multi-Currency Support
- All monetary values stored in base currency (user's primary currency)
- `Money` objects handle currency conversion and formatting
- Historical exchange rates for accurate reporting

### Security & Authentication
- Session-based auth for web users
- API authentication via:
  - OAuth2 (Doorkeeper) for third-party apps
  - API keys with JWT tokens for direct API access
- Scoped permissions system for API access
- Strong parameters and CSRF protection throughout

## Key rules
- Project modes: "managed" or "self_hosted".
- Domain: User → Accounts → Transactions. Keep business logic in models, controllers thin.

Authentication & context
- Use Current.user and Current.family (never current_user / current_family).

Testing conventions
- Use Minitest + fixtures (no RSpec, no FactoryBot).
- Use mocha for mocks where needed; VCR for external API tests.

Frontend conventions
- Hotwire-first: Turbo + Stimulus.
- Prefer semantic HTML, Turbo Frames, server-side formatting.
- Use the helper icon for icons (do not use lucide_icon directly).
- Use Tailwind design tokens (text-primary, bg-container, etc.).

Backend & architecture
- Skinny controllers, fat models.
- Prefer built-in Rails patterns; add dependencies only with strong justification.
- Sidekiq for background jobs (e.g., SyncJob, ImportJob, AssistantResponseJob).

API & security

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ordi-personal/finance-motor](https://github.com/Ordi-personal/finance-motor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
