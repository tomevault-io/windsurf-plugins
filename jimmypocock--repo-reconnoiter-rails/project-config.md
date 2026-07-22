---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RepoReconnoiter is an **API-only backend service** that analyzes GitHub trending repositories using AI to provide developers with context-aware recommendations. The system fetches trending repos every 20 minutes, uses AI to categorize and analyze them, and exposes a RESTful API for frontend clients to consume. The only UI is Mission Control (`/admin/jobs`) for admin job monitoring.

## Core Principles

1. **Cost Control**: Keep AI API costs under $10/month through automatic tracking, caching, and smart model selection
2. **Code Consistency**: All code follows strict organization standards with alphabetized methods and clear section headers
3. **Service Pattern**: Use "Doer" naming (no "Service" suffix) for all service classes
4. **Automatic Tracking**: The `OpenAi` service automatically tracks all API costs - never call OpenAI directly
5. **Prompt as Code**: AI prompts are versioned ERB templates in `app/prompts/`, not hardcoded strings
6. **Multi-Query Strategy**: Use 2-3 GitHub queries for comprehensive results when needed
7. **Observability**: Use strategic logging in background jobs and services for debugging - production logs stay clean and focused

## Logging Philosophy

**Development Environment:**

- Log level: `:debug` (verbose SQL queries, full stack traces, detailed debugging info)
- Broadcast logger: Writes to both STDOUT and `log/development.log` for `bin/dev` visibility
- Framework logging: Enabled (SQL, ActiveRecord, ActionCable, etc.)
- Custom logger statements: Encouraged in background jobs to track async operations

**Production Environment:**

- Log level: `:info` (request/response, errors, warnings only)
- Framework logging: Minimal (no SQL queries, concise output)
- Custom logger statements: Use sparingly - focus on business-critical events and errors
- Exception tracking: Errors automatically logged by Rails + Sentry

**Philosophy**: Strategic logging in background jobs and async operations aids debugging without cluttering logs. Development broadcasts to console for real-time monitoring via `bin/dev`. Production logs focus on business events, errors, and warnings only.

## Tech Stack

- **Framework**: Rails 8.1 (API-only mode, no frontend)
- **Database**: PostgreSQL 17 (production: Render managed)
- **Background Jobs**: Solid Queue (database-backed, no Redis needed)
- **Job Scheduling**: Solid Queue recurring tasks (see `config/recurring.yml`)
- **AI Provider**: OpenAI (gpt-5-mini for categorization, gpt-5 for comparisons and deep analysis)
- **Authentication**:
  - **Two-Layer API Auth**:
    - Layer 1 (App-to-App): API Keys via `Authorization: Bearer <API_KEY>` header - required for ALL endpoints
    - Layer 2 (User-Specific): JWT tokens via `X-User-Token` header - for user-specific actions
    - Generate API key: `bin/rails api_keys:generate NAME="Insomnia Testing"` (see `lib/tasks/api_keys.rake`)
    - Exchange GitHub OAuth token for JWT: `POST /api/v1/auth/exchange`
  - **Mission Control**: Devise + OmniAuth GitHub (invite-only whitelist)
  - **Session Exchange**: JWT → Rails session for Mission Control access via `/session_exchange?token=JWT`
- **Rate Limiting**: Rack::Attack (25/day per user, 5/day per IP)
- **Deployment**: Render.com (Starter plan - $14/month)
- **Hosting**:
  - Production API: <https://api.reporeconnoiter.com/v1/> (subdomain routing)
  - Development API: <http://localhost:3001/api/v1/> (path-based routing)
  - Mission Control: `/admin/jobs` (admin-only UI for job monitoring)
  - PostgreSQL 17 database (1GB storage, 97 connections)
  - Web Service (512MB RAM, always-on, shell access)
- **Ruby Version**: 3.4.7

## Development Commands

### Setup

```bash
bin/setup                 # Initial setup: install dependencies, setup database
bin/dev                   # Start development server (runs Puma + Solid Queue + Tailwind watcher)
```

### Database

```bash
bin/rails db:create       # Create database
bin/rails db:migrate      # Run migrations
bin/rails db:seed         # Seed database (populates initial categories)
bin/rails db:reset        # Drop, create, migrate, and seed database
```

### Testing

```bash
bin/rails test            # Run all tests (290 tests, API + backend only)
bin/rails test test/models/repository_test.rb  # Run specific test file
```

### Linting & Security

```bash
bin/rails ci:all          # Run all CI checks (security, lint, tests) - mirrors GitHub Actions
bin/rails ci:security     # Run security scans only (Brakeman, Bundler Audit, Importmap)
bin/rails ci:lint         # Run RuboCop linter only
bin/rails ci:test         # Run all tests only (unit + system)

# Individual commands (if needed)
bin/rubocop               # Run RuboCop linter
bin/brakeman              # Run security vulnerability scanner
bin/bundler-audit         # Check for vulnerable gem versions
```

### Background Jobs

```bash
bin/rails solid_queue:start    # Start Solid Queue worker
bin/rails solid_queue:stop     # Stop Solid Queue worker
```

### API Keys

```bash
bin/rails api_keys:generate NAME="Insomnia Testing"              # Generate new API key

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jimmypocock/repo-reconnoiter-rails](https://github.com/jimmypocock/repo-reconnoiter-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
