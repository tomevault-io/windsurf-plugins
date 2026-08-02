---
trigger: always_on
description: This is a comprehensive guide for AI coding agents working on the Condo project - an Open Source property management SaaS platform.
---

# AGENTS.md

This is a comprehensive guide for AI coding agents working on the Condo project - an Open Source property management SaaS platform.

> **Important Reminder:** As an AI coding agent, you are a tool that **complements** the developer, not a replacement. The developer remains responsible for reviewing, understanding, and validating all code changes you generate. Always provide clear explanations and context for your suggestions.

## Project Overview

**Condo** is a property management platform that allows users to manage tickets, resident contacts, properties, payment tracking, invoices, and a service marketplace. It offers an extension system for mini-apps and is built as a monorepo with multiple apps and shared packages.

- **Repository:** https://github.com/open-condo-software/condo
- **License:** MIT
- **Architecture:** Monorepo with yarn workspaces
- **Primary Stack:** KeystoneJS, NextJS, ApolloGraphQL, PostgreSQL, Redis

## Monorepo Structure

### Apps (`/apps/*`)
Independent applications that cannot use code from each other. Each app is a standalone service:
- `condo` - Main Keystone + Next web application (property management)
- `pos-integration` - POS integration service
- `billing-connector` - Billing integration
- `callcenter` - Call center service
- `resident-app` - Resident mobile/web app
- `dev-portal-web` - Developer portal
- And 20+ other specialized apps

### Packages (`/packages/*`)
Internal libraries shared across apps:
- `@open-condo/ui` - **UI Kit** (recommended for all GUI elements)
- `@open-condo/icons` - Icon library
- `@open-condo/*` packages - Core utilities and shared functionality
- `@open-condo/webhooks` - Webhook functionality
- `apollo` - Apollo client utilities
- `billing` - Billing utilities
- And more specialized packages

## Environment Requirements

### Required Software
- **Node.js:** 24.x (LTS) - Use nvm for local development
- **Yarn:** 4.17.0 (Berry) — pinned via `packageManager` in `package.json` and `yarnPath` in `.yarnrc.yml`
- **Python:** 3.x (for database migrations)
- **PostgreSQL:** 16.8 (as defined in docker-compose.yml)
- **Redis:** 6.2 (as defined in docker-compose.yml)
- **Docker & Docker Compose** (optional, for databases)

### Python Packages
```bash
pip install Django==5.2 psycopg2-binary==2.9.10
```

## Setup Commands

### Initial Setup
```bash
# 1. Start databases (optional, using Docker)
docker compose --profile dbs up -d

# 2. Install Node.js dependencies
yarn install

# 3. Build @open-condo packages (required before running apps)
yarn workspace @app/condo build:deps

# 4. Prepare local environment (creates .env, databases, test users)
node bin/prepare -f condo
```

### Development Commands
```bash
# Start main condo app in dev mode
yarn workspace @app/condo dev

# Start worker (handles async tasks)
yarn workspace @app/condo worker

# Run tests
yarn workspace @app/condo test
yarn workspace @app/condo test User.test.js  # Specific test file

# Build for production
yarn workspace @app/condo build
yarn workspace @app/condo start
```

### Database Migrations
```bash
# Run migrations
yarn workspace @app/condo migrate

# Create new migrations after schema changes
yarn workspace @app/condo makemigrations

# Revert last migration
yarn workspace @app/condo migrate:down

# Unlock migrations table
yarn workspace @app/condo migrate:unlock
```

### Build Commands
```bash
# Build @open-condo dependencies (required before running apps)
yarn workspace @app/condo build:deps

# Build all packages
yarn build:packages

# Build all apps
yarn build:apps

# Build everything
yarn build
```

### Package Management

**Important:** The project maintains the same version of each package across all apps due to the shared `yarn.lock` file. This ensures consistency and prevents version conflicts.

```bash
# Add package to all apps (recommended for external dependencies)
yarn add <package> -W

# Add package to specific app (use only for app-specific dependencies)
yarn workspace @app/<name> add <package>

# Run command in specific workspace
yarn workspace @app/<name> <command>

# Upgrade packages interactively (updates across all apps)
yarn upgrade-interactive --latest
```

## Code Style Guidelines

### JavaScript/TypeScript

**ESLint Configuration** (`.eslintrc.js`):
- **Indentation:** 4 spaces
- **Quotes:** Single quotes for strings, single quotes for JSX attributes
- **Semicolons:** Never use semicolons
- **Object spacing:** Always use spaces inside curly braces: `{ foo }`
- **Trailing commas:** Always for multiline arrays/objects/imports
- **Space before function paren:** Always required
- **Import order:** Enforced with groups (builtin → external → @open-condo → internal → sibling → parent)
- **Newlines between imports:** Always required between groups
- **Alphabetize imports:** Case-insensitive ascending order

**Key Rules:**
```javascript
// ✅ Correct
const foo = { bar: 'baz' }
function example () {
    return 'hello'
}

// ❌ Wrong
const foo = {bar: 'baz'};
function example() {
    return "hello";
}
```

**Import Patterns:**
```javascript
// ✅ Use specific lodash imports
import get from 'lodash/get'

// ❌ Don't use named imports from lodash
import { get } from 'lodash'


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-condo-software/condo](https://github.com/open-condo-software/condo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
