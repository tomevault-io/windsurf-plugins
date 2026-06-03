---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an n8n community node package that integrates with **Firefly III**, a self-hosted personal finance manager. The node provides workflow automation capabilities for managing transactions, accounts, budgets, categories, tags, rules, and other Firefly III resources through n8n workflows.

**Key Technologies:**
- TypeScript (strict mode)
- n8n-workflow SDK
- OAuth2 authentication (PKCE grant)
- Firefly III REST API v1 (6.4.0)

## Development Commands

```bash
# Install dependencies (pnpm required)
pnpm install

# Build TypeScript and copy icons
pnpm build

# Watch mode for development
pnpm dev

# Start: build + restart n8n-dev Docker container + tail logs
pnpm start

# Code quality
pnpm lint
pnpm lintfix
pnpm format

# Pre-publish validation
pnpm prepublishOnly
```

## Important Folders

### `nodes/FireFlyIII/`
Main node implementation containing:
- **`Fireflyiii.node.ts`**: Core node class with `description` (defines UI/parameters) and `execute()` method (handles operations)
- **`actions/`**: Resource-specific operation definitions organized by endpoint:
  - `transactions/` - Create, list, update, delete transactions
  - `accounts/` - Account management
  - `bills/` - Bill management (full CRUD + attachments, rules, transactions)
  - `budgets/` - Budget management (full CRUD + limits, transactions)
  - `categories/` - Category operations
  - `tags/` - Tag management
  - `rules/` - Rule and rule group operations
  - `piggyBanks/` - Piggy bank management (full CRUD + events, attachments)
  - `objectGroups/` - Object group management (list, get, update, delete, related objects)
  - `general/` - Export, insights, search operations
  - `about/` - System information
- **`utils/`**: Shared utilities
  - `ApiRequest.ts` - Main Firefly III API request handler with OAuth2
  - `ApiRequestV2.ts` - API v2 request handler (partial implementation)

### `credentials/`
- **`FireflyiiiOAuth2Api.credentials.ts`**: OAuth2 credential configuration with PKCE grant type

### `dist/`
Build output directory (compiled JavaScript + icons)

### `.claude/docs/`
Reference documentation for development:
- **`firefly-iii-6.4.0-v1.yaml`**: Complete OpenAPI specification for Firefly III API v1 (6.4.0). Use this as the authoritative reference for endpoint specifications, request/response formats, and API behavior.
- **`API_REQUEST_COMPARISON.md`**: Technical comparison between ApiRequest implementations
- Additional n8n node development guides and code explanations

## Firefly III API Integration

### API Structure
The Firefly III API follows REST conventions with versioned endpoints. This node uses **Firefly III API v1 (6.4.0)** exclusively.

**Implemented Resources:**
- **General Operations** (`/api/v1/search/*`, `/api/v1/data/export`, `/api/v1/insight/*`): Search, export, insights (3 operations)
- **About** (`/api/v1/about`, `/api/v1/cron/*`): System info, user info, cron jobs (3 operations)
- **Accounts** (`/api/v1/accounts/*`): Full CRUD + related transactions, attachments, piggy banks (6 operations)
- **Available Budgets** (`/api/v1/available-budgets/*`): total available amount that the user has made available to themselves. (2 operations)
- **Bills** (`/api/v1/bills/*`): Full CRUD + attachments, rules, transactions (8 operations)
- **Budgets** (`/api/v1/budgets/*`): Full CRUD + limits, spent amounts, transactions (15 operations)
- **Transactions** (`/api/v1/transactions/*`): Full CRUD + attachments, piggy bank events, splits (6 operations)
- **Categories** (`/api/v1/categories/*`): Full CRUD + transactions (6 operations)
- **Tags** (`/api/v1/tags/*`): Full CRUD + transactions, attachments (7 operations)
- **Rules & Rule Groups** (`/api/v1/rules/*`, `/api/v1/rule-groups/*`): Full CRUD + testing, triggering (14 operations)
- **Piggy Banks** (`/api/v1/piggy-banks/*`): Full CRUD + events, attachments (7 operations)
- **Object Groups** (`/api/v1/object-groups/*`): List, get, update, delete + related bills/piggy banks (6 operations)
  - **Note**: Object groups cannot be created directly; they are auto-created when bills or piggy banks use `object_group_title` parameter
- **Recurrences** (`/api/v1/recurrences/*`): Full CRUD + trigger operations for recurring transactions (6 operations)
  - **Note**: Complex nested structures for repetitions (schedule patterns) and transactions (transaction details)

**API Endpoints Not Yet Implemented:**
- Attachments (as standalone resource - `/api/v1/attachments/*`)
- Autocomplete (`/api/v1/autocomplete/*`)
- Charts (`/api/v1/chart/*`)
- Configuration (`/api/v1/configuration/*`)
- Currencies (`/api/v1/currencies/*`)
- Currency Exchange Rates (`/api/v1/cer/*`)
- Links (`/api/v1/transaction-links/*`, `/api/v1/link-types/*`)
- Preferences (`/api/v1/preferences/*`)
- Summary (`/api/v1/summary/*`)
- Webhooks (`/api/v1/webhooks/*`)

**API Reference:**
- [Firefly III API Documentation](https://api-docs.firefly-iii.org/) - Official online documentation
- `.claude/docs/firefly-iii-6.4.0-v1.yaml` - Complete OpenAPI specification (authoritative reference)

### Authentication Pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jihad/n8n-fireflyiii-node](https://github.com/Jihad/n8n-fireflyiii-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
