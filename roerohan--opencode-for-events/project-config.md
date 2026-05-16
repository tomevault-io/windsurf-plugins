---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md

Guidelines for AI agents working in this repository.

## Project Overview

Cloudflare Worker that provides secure, budget-controlled OpenCode access for event participants through Cloudflare Access authentication and team-based credit limits.

**What it does:**
- Authenticates users via Cloudflare Access (JWT with JWKS verification)
- Maps authenticated user emails to teams from KV storage
- Enforces per-team credit limits with real-time usage tracking
- Proxies requests to Cloudflare AI Gateway (Anthropic & OpenAI providers)
- Tracks costs via `cf-aig-cost-usd` header and updates team usage in KV
- Blocks access with 429 when team exceeds credit limit
- Serves dynamic OpenCode configuration at `/.well-known/opencode`
- Caches OpenAI models list from models.dev (refreshed hourly via cron)
- Applies `store: false` to all OpenAI models for ZDR compatibility

**Use Case:** Hackathons, workshops, and coding events where participants are organized into teams with fixed credit allocations (e.g., $20 per team). Access is automatically denied when credits are exhausted.

**Tech Stack:** Cloudflare Workers, Hono, TypeScript, Wrangler, KV Storage, Cloudflare Access, AI Gateway

## Commands

```bash
npm install              # Install dependencies
npm run build:config     # Build OpenCode config (runs automatically)
npm run setup-teams      # Upload team configurations
npm run view-teams       # View team usage
npm run dev              # Development server (wrangler dev)
npm run deploy           # Production deployment
npm run cf-typegen       # Generate Cloudflare types
```

## Project Structure

```
config/
  base.json               # Core config (providers and settings)
  opencode.json           # Generated - DO NOT EDIT (gitignored)
public/
  index.html              # Landing page
scripts/
  build-config.ts         # Compiles config into opencode.json
  setup-teams.ts          # Bulk upload team configurations
  view-teams.ts           # View team usage and status
src/
  index.ts                # Main Hono app entry point
wrangler.jsonc            # Wrangler config
teams.example.json        # Example team configuration
```

## TypeScript Configuration

- Target: ESNext
- Module: ESNext with Bundler resolution
- **Strict mode enabled**
- JSX: react-jsx with hono/jsx import source

## Code Style Guidelines

### Imports

```typescript
// Named imports from libraries
import { Hono } from "hono";

// Type-only imports (use 'import type' for types)
import type { JwtVariables } from "hono/jwt";
import type { HonoJsonWebKey } from "hono/utils/jwt/jws";

// Local imports
import configTemplate from "../config/opencode.json";

// Default exports for app/tool modules
export default app;
```

### Naming Conventions

| Type       | Convention          | Example                    |
|------------|---------------------|----------------------------|
| Constants  | SCREAMING_SNAKE     | `CF_ACCESS_TEAM_NAME`      |
| Functions  | camelCase           | `createErrorResponse`      |
| Interfaces | PascalCase          | `ErrorResponse`            |
| Variables  | camelCase           | `cachedKeys`               |

### Type Definitions

```typescript
// Define interfaces for data structures
interface Env {
  GATEWAY_API_KEY: string;
  GATEWAY_URL: string;
}

interface ErrorResponse {
  error: string;
  message: string;
  status?: number;
  timestamp: string;
}

// Type Hono apps with bindings and variables
type Variables = JwtVariables<{ email?: string }>;
const app = new Hono<{ Bindings: Env; Variables: Variables }>();

// Type JSON responses explicitly
const data = await response.json() as { keys: HonoJsonWebKey[] };
```

### Error Handling

Use structured error responses with consistent format:

```typescript
interface ErrorResponse {
  error: string;      // Error type (e.g., "Unauthorized", "Gateway Error")
  message: string;    // Human-readable message
  status?: number;    // HTTP status code
  timestamp: string;  // ISO timestamp
}

function createErrorResponse(message: string, status = 500, error = 'Internal Server Error'): Response {
  const errorResponse: ErrorResponse = {
    error,
    message,
    status,
    timestamp: new Date().toISOString()
  };
  console.log(errorResponse);  // Always log errors
  return new Response(JSON.stringify(errorResponse), {
    status,
    headers: { 'Content-Type': 'application/json' }
  });
}
```

For tools, return error strings (don't throw):
```typescript
return `Error: ${message}`;
```

### Comments

- Focus on "why", not "what"
- Don't comment single variables or short functions
- Comment logic with I/O, validation, or edge cases

```typescript
// Good: explains why
// 1 hour TTL for JWKS cache
cacheExpiration = now + 3600 * 1000;

// Good: documents configurable behavior
// JIRA fields to extract from API response
// Add/remove field paths here to control what's included
const JIRA_INCLUDED_FIELDS = [...]
```

### Formatting

- Tabs for indentation
- Double quotes for strings
- Semicolons at end of statements

## Environment Variables

Worker bindings (defined in wrangler.jsonc):
- `GATEWAY_API_KEY` - AI Gateway authorization key (secret)
- `GATEWAY_URL` - AI Gateway base URL
- `GATEWAY_ACCOUNT_ID` - Cloudflare account ID
- `GATEWAY_ID` - Gateway identifier

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roerohan/opencode-for-events](https://github.com/roerohan/opencode-for-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
