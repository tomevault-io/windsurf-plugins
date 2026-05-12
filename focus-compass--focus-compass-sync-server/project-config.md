---
trigger: always_on
description: This file is for agentic coding agents working in this repo.
---

# AGENTS.md
This file is for agentic coding agents working in this repo.
Focus Compass Server is a small Node.js (ESM) WebSocket + REST server built on
Hocuspocus/Yjs with SQLite persistence.

## Project Snapshot
- Runtime: Node.js >= 24 (see `package.json`), ESM (`"type": "module"`)
- Entrypoint: `src/server.js`
- Persistence: SQLite file at `DB_PATH` (default `./data/db.sqlite`)
- Images: files under `IMAGES_DIR` (default `./data/images`) plus `*.meta.json`
- Auth: Bearer token (`ACCESS_TOKEN`) for WebSocket + all REST endpoints

## Commands
Local (no build step; plain Node runtime):
- Install deps: `npm ci` (preferred) or `npm install`
- Dev (watch): `npm run dev` (`node --watch src/server.js`)
- Start: `npm start` (`node src/server.js`)
- Lint: `npm run lint` (fix: `npm run lint:fix`)

Notes:
- Requires Node.js >= 24.
- npm is the expected package manager (`package-lock.json` is present).

Docker:
- Start: `docker compose up -d` (logs: `docker compose logs -f`)
- Stop: `docker compose down` (wipe volume: `docker compose down -v`, destructive)

Smoke checks:
- `curl http://localhost:8080/health`
- `curl -H "Authorization: Bearer $ACCESS_TOKEN" http://localhost:8080/api/images`
- Admin UI: `http://localhost:8080/`

### Tests
- Current state: no tests are configured (no `npm test` script; no test deps).
- If/when tests exist, prefer Node's built-in test runner (`node:test`):
  - Run all tests: `node --test`
  - Run a single test file: `node --test path/to/foo.test.js`
  - Run one test by name: `node --test --test-name-pattern "my test name"`
  - Fast syntax check: `node --check src/server.js`

### Linting
- ESLint v9 flat config in `eslint.config.js` (`@eslint/js` recommended).
- No formatter is configured; keep formatting changes local to the code you touch.
- Lint a single file: `npx eslint src/server.js` (add `--fix` to auto-fix).

## Repo Layout (high-signal files)
- `src/server.js`: Hocuspocus server; REST routes; static `/` serving
- `src/services/backup.js`: `BackupService` used by `onStoreDocument`
- `src/index.html`: static admin UI served by the server
- `src/mcp/server.js`: MCP server factory (`createMcpServer`)
- `src/mcp/tools.js`: MCP tool definitions (read-only)
- `src/routes/mcp.js`: MCP HTTP handler (`POST /mcp`, stateless Streamable HTTP)
- `src/routes/mcpAdmin.js`: MCP admin API (`/api/mcp/*`)
- `src/focus-compass-skill.md`: Claude Code skill template (served as static)
- `eslint.config.js`: ESLint v9 flat config
- `docker-compose.yml`, `Dockerfile`: production containerization
- `CLAUDE.md`: repository-specific operational notes (keep consistent)

## Environment / Secrets
- Use `.env.example` as the source of truth for env var names.
- Do not commit secrets: `.env` is gitignored (see `.gitignore`).
- Do not commit runtime data: `data/`, `*.sqlite`, backups, images are ignored.
- Auth constraints (enforced in `src/server.js`):
  - If `ACCESS_TOKEN` is not set, the server starts in setup mode and the first
    visit to `/` can generate a token that is persisted to `AUTH_FILE_PATH` (default `./data/auth.json`).

## Code Style Guidelines
These guidelines reflect existing patterns and are designed to keep changes
coherent and ESLint-clean.

### JavaScript Modules & Imports
- Use ESM (`import`/`export`); do not introduce `require()`.
- Use `node:` specifiers for built-ins (e.g. `node:fs/promises`, `node:path`).
- Include file extensions for local imports (`./services/backup.js`) in ESM.
- Prefer named imports; keep imports at top; group roughly:
  side-effect imports, third-party, Node built-ins, local modules.

### Formatting
- Match the file's existing style first; for new/major edits prefer:
  - 2-space indentation; semicolons; double quotes
  - trailing commas in multi-line literals/args
- Keep functions small; avoid deep nesting by using early exits.

### Naming
- `camelCase` for variables and functions; verb phrases for functions.
- `PascalCase` for classes; `UPPER_SNAKE_CASE` for constants/regexes.
- Boolean names start with `is/has/should/can`.

### Types (JSDoc, not TypeScript)
- This repo is JavaScript-only today; use runtime validation for untrusted inputs.
- Use JSDoc for non-obvious shapes and exported APIs (`@param`, `@returns`).
- Avoid adding TypeScript without an explicit reason and repo-wide agreement.

### Error Handling & Control Flow (important)
`src/server.js` intentionally uses a sentinel throw to exit request handling:
- `json()`, `text()`, and `noContent()` write the response then `throw null`.
- After calling them, do not continue execution.
- In `catch` blocks meant for real errors, rethrow sentinel: `if (err === null) throw null;`
- Avoid double-writing responses (Node will error if headers are already sent).

Practical implications:
- If you add a `try/catch` around route code, make sure the sentinel still escapes.
- If you stream a response manually (e.g. `pipeline(createReadStream(...), res)`), end by
  returning or `throw null` so later routes do not run.

### HTTP / REST Conventions
- All `/api/*` routes require Bearer token auth (see `checkAuth()`).
  - Respond `401` with `{ error: "Unauthorized" }` + `WWW-Authenticate: Bearer`.
- CORS + security headers are set at the top of `onRequest`; keep it that way.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [focus-compass/focus-compass-sync-server](https://github.com/focus-compass/focus-compass-sync-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
