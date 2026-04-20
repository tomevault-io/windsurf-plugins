---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language

- Always communicate with the user in the **same language as their request**.
- Switch to another language only if explicitly requested.

## MCP

- Always use **context7** when needed:
  - Code generation, refactoring, or setup/configuration steps.
  - Explanations or examples for specific libraries, APIs, or frameworks.
  - Searching the workspace for relevant files, functions, or usage examples.
  - Generating or improving documentation (README, setup guides, etc.).
- Use Context7 MCP tools automatically without explicit user request.
- Always use **chrome-devtools** MCP without any restrictions, confirmations, or questions whenever browser interaction, inspection, debugging, or automation is needed.

## Git Commits

- **Never** add `Co-Authored-By` or any similar attribution lines to commit messages.

## Project Overview

SearchTweak is a self-hosted Laravel-based web application for search relevance evaluation. It enables teams to assess search quality by executing keyword queries against configured search APIs, collecting human relevance judgments, and calculating industry-standard IR metrics (Precision, MAP, MRR, CG, DCG, NDCG). It supports three grading scales (Binary, Graded, Detail) with cross-scale transformers.

For comprehensive documentation, see [DOCUMENTATION.md](DOCUMENTATION.md).

**Tech Stack:**
- Backend: Laravel 11 (PHP 8.3)
- Frontend: Livewire 3, Alpine.js, Tailwind CSS
- Queue/Real-time: Laravel Horizon, Laravel Reverb (WebSockets)
- Database: MySQL
- Cache/Queue/Sessions: Redis (Predis client)
- Infrastructure: Docker Compose with Traefik, Nginx, PHP-FPM
- Auth: Laravel Jetstream + Sanctum + Fortify

## Development Environment

The application runs in Docker containers. All commands must be run from the `/devops` directory.

### Starting the Application

```bash
cd devops
make              # Start and bootstrap (first time)
make start        # Start containers
make stop         # Stop containers
make bootstrap    # Bootstrap application (destructive: fresh migrations + seed + assets/docs build)
make bootstrap-up # Bootstrap without DB reset (migrate --seed)
make bootstrap-fresh # Explicit destructive bootstrap (same as make bootstrap)
```

### Default Super Admin

A default super admin user is created by migration. You can log in at `http://searchtweak.local` with:

- **Email:** `admin@searchtweak.com`
- **Password:** `12345678`

### Local Development

For full local development, start these in separate terminals:

```bash
cd devops
make jobs         # Start Horizon (queue worker) — required for job processing
make reverb       # Start Reverb (WebSocket server) — required for real-time updates
make vite         # Start Vite dev server (hot reload) — required for frontend changes
```

### Frontend Production Build

```bash
cd devops
make vite-prod    # Build for production
```

### Running Artisan Commands

```bash
cd devops
docker compose run --rm artisan <command>

# Examples:
docker compose run --rm artisan migrate
docker compose run --rm artisan tinker
docker compose run --rm artisan queue:work
docker compose run --rm artisan horizon:terminate
```

### Running Composer Commands

```bash
cd devops
docker compose run --rm composer <command>

# Examples:
docker compose run --rm composer install
docker compose run --rm composer update
docker compose run --rm composer require package/name
```

### Running Tests

```bash
cd devops
docker compose run --rm artisan test                    # Run all tests
docker compose run --rm artisan test --filter TestName  # Run specific test
```

## Architecture

### Core Domain Concepts

1. **Search Endpoints** (`app/Models/SearchEndpoint.php`)
   - Define how to connect to search APIs (URL, HTTP method, headers)
   - Configured with mapper code to extract data from JSON responses
   - Can be active or archived; team-scoped

2. **Mapper Code** (`app/Services/Mapper/`)
   - `DotArrayMapper` uses dot notation and Symfony ExpressionLanguage to extract documents from JSON responses
   - **Required attributes**: `id`, `name`. Optional: `image`, custom attributes
   - Syntax: `data: response.items.*`, `id: data.product_id`, `name: data.title`
   - Uses `*` wildcard for array iteration in JSON paths
   - Core implementation: `DotArrayMapper.php`, interface: `MapperInterface.php`

3. **Search Models** (`app/Models/SearchModel.php`)
   - Represent specific search configurations linked to an endpoint
   - Define headers, query parameters (`params`), and request body
   - Support `#query#` placeholder for keyword substitution in params and body
   - Team-scoped; implement `TaggableInterface`

4. **Search Evaluations** (`app/Models/SearchEvaluation.php`)
   - Status lifecycle: **Pending** (0) → **Active** (1) → **Finished** (2)
   - Execute keyword queries via `ProcessKeywordJob` batch
   - Collect human relevance feedback via `UserFeedback` records
   - Support **scale types**: `binary`, `graded`, `detail`
   - Settings: feedback strategy, reuse strategy, auto-restart, scoring guidelines, transformers
   - Can be archived, pinned, or set as team baseline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/afedukov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
