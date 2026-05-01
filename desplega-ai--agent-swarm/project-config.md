---
trigger: always_on
description: Multi-agent orchestration for Claude Code, Codex, Gemini CLI. Runtime: Bun + TypeScript. DB: bun:sqlite (WAL mode). Linter: Biome. CLI: Ink.
---

# Agent Swarm

Multi-agent orchestration for Claude Code, Codex, Gemini CLI. Runtime: Bun + TypeScript. DB: bun:sqlite (WAL mode). Linter: Biome. CLI: Ink.

**Getting Started**: See [CONTRIBUTING.md](./CONTRIBUTING.md) for setup. Run `bun run start:http` to start the server.

## Project map

```
src/
  http.ts       # Main HTTP server + MCP endpoints
  stdio.ts      # Stdio MCP transport
  cli.tsx       # CLI entry point (Ink)
  commands/
    codex-login.ts  # Codex ChatGPT OAuth login command
  tools/        # MCP tool definitions
  providers/
    codex-oauth/   # Codex OAuth PKCE flow + storage
    codex-adapter.ts # Codex provider adapter
  be/           # Backend (DB, storage)
    db.ts       # DB init + query functions
    migrations/ # SQL migration files + runner
  github/       # GitHub webhook handlers
  slack/        # Slack integration
new-ui/          # Dashboard (Next.js app)
templates/       # Template data (official + community)
  official/      # 9 official templates
  community/     # Community-contributed templates
  schema.ts      # Shared TypeScript types
templates-ui/    # Templates registry (Next.js app)
```

## Architecture invariants

The API server (`src/http.ts`, `src/server.ts`, `src/tools/`, `src/http/`) is the **sole owner** of the SQLite database. Worker-side code (`src/commands/`, `src/hooks/`, `src/providers/`, `src/prompts/`, `src/cli.tsx`, `src/claude.ts`) must **never** import from `src/be/db` or `bun:sqlite`. Workers communicate with the API exclusively via HTTP using `API_KEY` and `X-Agent-ID` headers. Enforced by `scripts/check-db-boundary.sh` (pre-push hook + CI).

If worker-side code needs data from the DB (template resolution, config lookup), it must fetch via HTTP, not query SQLite directly. Shared pure logic goes in `src/prompts/` or `src/utils/`.

<important if="you need to run commands to build, test, lint, start the server, or generate code">

## Commands

| Command | What it does |
|---|---|
| `bun install` | Install dependencies |
| `bun run start:http` | Run MCP HTTP server (port 3013) |
| `bun run dev:http` | Dev with hot reload (portless) |
| `bun run lint:fix` | Lint & format with Biome |
| `bun run tsc:check` | Type check |
| `bun test` | Run all unit tests |
| `bun test src/tests/<file>.test.ts` | Run specific test |
| `bun run pm2-start` | Start all (API :3013, UI :5274, lead :3201, worker :3202) |
| `bun run pm2-start` | Start all (API :3013, UI :5274, lead :3201, worker :3202) |
| `bun run pm2-stop` | Stop all services |
| `bun run pm2-restart` | Restart all services |
| `bun run pm2-logs` | View logs |
| `bun run pm2-status` | Check status |
| `bun run docker:build:worker` | Build Docker worker image |
| `bun run docs:openapi` | Regenerate `openapi.json` |
| `bun run docs:business-use` | Regenerate `BUSINESS_USE.md` (requires BU backend) |
| `bun run build:pi-skills` | Regenerate `plugin/pi-skills/` from `plugin/commands/*.md` |
| `docker compose -f docker-compose.local.yml up --build` | Local Docker Compose (API + lead + worker) |
| `docker compose -f docker-compose.local.yml down` | Tear down local Docker Compose |
| `uvx business-use-core@latest server dev` | Start BU backend on :13370 |
| `uvx business-use-core@latest flow eval <runId> <flow> -g -v` | Evaluate a BU flow run |
| `uvx business-use-core@latest flow graph <flow>` | Show BU flow graph |

PM2 note: lead/worker run in Docker. On code changes: `bun run docker:build:worker && bun run pm2-restart`.

</important>

<important if="you are choosing between Bun and Node.js APIs, selecting packages, or writing shell/file/HTTP/SQLite code">

## Bun rules

Use Bun instead of Node.js, npm, pnpm, or vite everywhere.

- `Bun.serve()` for HTTP/WebSocket (not express/ws)
- `bun:sqlite` for SQLite (not better-sqlite3)
- `Bun.file()` over `node:fs` for file I/O
- `Bun.$` for shell commands (not execa)
- Bun auto-loads `.env` — don't use dotenv

</important>

<important if="you are referencing Gemini models in tests, workflows, or examples">

Use `google/gemini-3-flash-preview` as the default Gemini model (not `gemini-2.0-flash-001`).

</important>

<important if="you are adding or modifying database schema or migrations">

## Database migrations

Schema changes use file-based migrations in `src/be/migrations/`. Runner auto-applies on startup.

**Adding a migration:**
1. Create `src/be/migrations/NNN_descriptive_name.sql` (next number after highest existing)
2. Write forward-only SQL (CREATE TABLE, ALTER TABLE, CREATE INDEX, etc.)
3. Test with both fresh DB (`rm agent-swarm-db.sqlite && bun run start:http`) and existing DB

**Rules:**
- Never modify an already-applied migration — create a new one instead
- No `down` migrations (SQLite limitations make rollbacks unreliable)
- Use `IF NOT EXISTS` for CREATE TABLE/INDEX for safety
- Keep `AgentTaskSourceSchema` in `src/types.ts` in sync with CHECK constraints in SQL

**How it works:** `_migrations` table tracks applied versions with checksums. Bootstrap is schema-aware — if baseline tables already exist, `001_initial` is marked applied without re-executing. `initDb()` runs compatibility guards after migrations for legacy DBs.

</important>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [desplega-ai/agent-swarm](https://github.com/desplega-ai/agent-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
