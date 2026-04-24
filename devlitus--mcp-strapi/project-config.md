---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Strapi Server is a Model Context Protocol (MCP) server that provides 16 tools for CRUD operations, batch operations, and content management in Strapi CMS. It uses stdio transport to integrate with Claude Desktop and other MCP clients.

**Key Technologies:**

- TypeScript with ESNext modules
- Node.js 22+ (requires native fetch support)
- MCP SDK (@modelcontextprotocol/sdk)
- Zod for schema validation

## Development Commands

```bash
# Build TypeScript to dist/
npm run build

# Development with hot-reload (uses tsx)
npm run dev

# Production (requires build first)
npm start

# Watch mode for TypeScript compilation
npm run watch

# Clean compiled files
npm run clean
```

## Git Commits

When creating commits:
- Write clear, concise commit messages that describe the changes
- Do NOT add "Generated with Claude Code" footers or "Co-Authored-By" signatures
- Use conventional commit format when appropriate (fix:, feat:, docs:, etc.)
- Keep the commit message focused and professional

## Architecture

### Entry Point Flow

1. `src/index.ts` - Runs health check on startup, then creates stdio transport
2. **Health Check** - Validates Strapi connection and credentials before starting
3. `src/server.ts` - Registers all 16 tools and 1 prompt with the MCP server
4. Tools are loaded from `src/tools/` and registered individually

### Health Check (`src/services/strapi-client.ts`)

Runs automatically on server startup to validate Strapi is accessible:

- **URL Validation**: Checks that `STRAPI_URL` has valid format (http/https)
- **Connection Test**: Attempts to connect to Strapi instance via `GET /health` endpoint
- **Clear Error Messages**: Fails early with helpful diagnostics

**Endpoint Used:** `GET /health` (no authentication required)

**Error Cases Handled:**
- Invalid URL format → `Invalid STRAPI_URL format`
- Server unreachable → `Connection refused. Is Strapi running?`
- Any other HTTP error → `HTTP {status}: {statusText}`

If health check fails, the server exits with status code 1 and won't accept connections.

### Critical Components

**StrapiClient (`src/services/strapi-client.ts`)**

- Singleton HTTP client using native fetch (Node.js 22+)
- Handles authentication via `STRAPI_API_TOKEN` env var
- Includes `healthCheck()` method for startup validation
- Builds Strapi-compliant query strings for filters, pagination, populate, etc.
- All errors logged to stderr (stdout reserved for MCP protocol)

**i18n Validator (`src/services/i18n-validator.ts`)**

- Validates content language consistency in multilingual entries
- Detects inherited vs native translations
- Used by read, update, and list tools when locale parameter is specified

### Tool Categories

**CRUD Operations (6 tools):**

- `strapi-create` - Create entry (supports locale)
- `strapi-create-with-locales` - Create entry with multiple languages simultaneously
- `strapi-read` - Read by documentId (supports locale, validateLanguage, strictMode)
- `strapi-list` - List with filters (supports locale, showLocalizationSummary)
- `strapi-update` - Update by documentId (supports locale, validateBeforeUpdate, strictMode)
- `strapi-delete` - Delete by documentId

**Batch Operations (3 tools):**

- `strapi-batch-create` - Create multiple entries in parallel (ideal for migrations and bulk data loading)
- `strapi-batch-update` - Update multiple entries in parallel (supports locale for multilingual updates)
- `strapi-batch-delete` - Delete multiple entries in parallel

**Content Type Management (3 tools):**

- `strapi-list-content-types` - List all available content types
- `strapi-get-schema` - Get detailed schema of a content type
- `strapi-add-field` - Add field to content type (requires content-type-builder permissions)

**Internationalization (1 tool):**

- `strapi-get-i18n-locales` - Get all available locales

**Media Management (3 tools):**

- `strapi-search-media` - Search files in upload library
- `strapi-get-media` - Get specific media file by ID or documentId
- `strapi-upload-media` - Upload file to media library

## Critical Implementation Details

### Content Type Naming Convention

**MOST IMPORTANT:** Different tools use different content type identifiers:

- **CRUD tools** (create, read, list, update, delete) use **PLURAL name**:

  - ✅ `"products"`, `"subcategories"`, `"categories"`
  - ❌ NOT `"api::product.product"`, NOT `"product"`

- **Schema tools** (get-schema, add-field) use **UID format**:
  - ✅ `"api::product.product"`
  - ❌ NOT `"products"`

Use `strapi-list-content-types` to get both formats - check the `pluralName` field for CRUD operations.

### Strapi v5 IDs

- Strapi v5 uses **documentId** (string) as the stable identifier
- All read/update/delete operations use documentId, not numeric id
- Responses include both `id` (number) and `documentId` (string)

### i18n Features

- **locale parameter**: Available in create, read, list, update
- **validateLanguage**: Auto-validates content language (default: true for read)
- **strictMode**: Rejects inherited/fallback content (default: false)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devlitus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
