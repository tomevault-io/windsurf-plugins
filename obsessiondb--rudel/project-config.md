---
trigger: always_on
description: A platform for ingesting, storing, and analyzing Claude Code / Codex session transcripts. Users authenticate via the web app, use the CLI to upload `.jsonl` session transcripts, which are stored in ClickHouse for analytics.
---

# Rudel

A platform for ingesting, storing, and analyzing Claude Code / Codex session transcripts. Users authenticate via the web app, use the CLI to upload `.jsonl` session transcripts, which are stored in ClickHouse for analytics.

**Stack**: Bun, Turborepo, Biome, TypeScript
**Deployment**: Bun server + Postgres (Neon) + ClickHouse
**Domain**: `app.rudel.ai`

## Monorepo Structure

### Apps

| App | Package | Description |
|-----|---------|-------------|
| `apps/api` | `@rudel/api` | HTTP API server (Bun). Auth via `better-auth`, RPC via `@orpc/server`, session ingestion into ClickHouse. |
| `apps/cli` | `rudel` (npm) | CLI tool for authenticating and uploading Claude Code / Codex session transcripts. Commands: `login`, `logout`, `whoami`, `upload`. |
| `apps/web` | `@rudel/web` | React SPA (Vite + Tailwind + shadcn). Auth UI and CLI login portal. |

### Packages

| Package | Description |
|---------|-------------|
| `packages/api-routes` (`@rudel/api-routes`) | Shared RPC contract (`@orpc/contract` + Zod schemas). Single source of truth for the API interface. |
| `packages/ch-schema` (`@rudel/ch-schema`) | ClickHouse table schema (`rudel.claude_sessions`, `rudel.session_analytics`), generated TypeScript types, and ingest functions via `chkit`. Has two entry points: `@rudel/ch-schema` (full — includes schema definitions that pull in `@chkit/core` and `@clickhouse/client`) and `@rudel/ch-schema/generated` (lightweight — only generated types and ingest functions). **Use `@rudel/ch-schema/generated`** for runtime imports to avoid loading heavy ClickHouse dependencies. |
| `packages/sql-schema` (`@rudel/sql-schema`) | Drizzle ORM schema for Postgres auth tables (`user`, `session`, `account`, `verification`). |
| `packages/typescript-config` (`@rudel/typescript-config`) | Shared `tsconfig` base files (`base.json`, `node.json`, `react-library.json`). |

### Dependency Graph

```
@rudel/typescript-config ← (devDep) all packages and apps
@rudel/api-routes        ← @rudel/api, @rudel/web, rudel CLI
@rudel/sql-schema        ← @rudel/api
@rudel/ch-schema         ← @rudel/api
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `PG_CONNECTION_STRING` | Yes | Postgres connection string (e.g. `postgres://postgres:postgres@localhost:5432/rudel`) |
| `BETTER_AUTH_SECRET` | Yes | Auth secret (generate with `openssl rand -base64 32`) |
| `CLICKHOUSE_URL` | Yes | ClickHouse HTTP endpoint |
| `CLICKHOUSE_USERNAME` | No | ClickHouse username |
| `CLICKHOUSE_PASSWORD` | No | ClickHouse password |
| `CLICKHOUSE_DB` | No | ClickHouse database name |
| `APP_URL` | No | API base URL (default: `http://localhost:4010`) |
| `ALLOWED_ORIGIN` | No | CORS origin (default: `http://localhost:4011`) |
| `RESEND_API_KEY` | No | Resend API key for email integration |
| `RESEND_FROM_EMAIL` | No | Sender address for organization invitation emails |
| `RESEND_AUDIENCE_ID` | No | Resend audience ID for signup contact sync |
| `VITE_ADMIN_ORGANIZATION_ID` | No | Organization ID whose members have admin panel access |

## Local chkit development

To test local chkit changes without publishing:

```bash
# Link to local chkit (at /Users/marc/Workspace/chkit or CHKIT_PATH)
bun run chkit:link

# Restore to published versions
bun run chkit:unlink
```

`chkit:link` rewrites the root `package.json` overrides to `file:` paths pointing at your local chkit checkout and runs `bun install`. It also installs a pre-commit hook that blocks commits with `file:` overrides active.

`chkit:unlink` restores overrides to the version declared in `packages/ch-schema/package.json` and runs `bun install`.

CI runs `scripts/check-overrides.ts` before `bun install` to catch any accidentally committed `file:` overrides.

## Local Development

There are two ways to run the app locally:

### 1. Standalone (local infra)

Uses Docker containers for Postgres and ClickHouse. No Doppler or external accounts required. Good for working on auth, UI, or API logic without needing real data.

**Prerequisites**: Docker (or [OrbStack](https://orbstack.dev)) must be installed and running.

```bash
bun install
bun run dev:local
```

This runs `scripts/dev-local.sh`, which:
1. Starts local Postgres + ClickHouse via Docker Compose (`docker compose up -d --wait`)
2. Runs Postgres migrations
3. Launches the API (`:4010`) and web app (`:4011`) in parallel

Open **http://localhost:4011** in your browser. Sign up with email/password to create a local account. Social login (Google/GitHub) is not available in this mode since there are no OAuth client credentials.

Environment is hardcoded in the script: local Postgres (`postgres://postgres:postgres@localhost:5432/rudel`), local ClickHouse (`http://localhost:8123`, password: `clickhouse`), and a static auth secret.

Manage containers separately: `bun run infra:up` / `bun run infra:down`. To wipe all data and start fresh: `docker compose down -v`.

### 2. Dev (production databases)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obsessiondb/rudel](https://github.com/obsessiondb/rudel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
