---
trigger: always_on
description: Web-based UI for the Claude Code CLI: a desktop/mobile interface for managing
---

# Bottega

Web-based UI for the Claude Code CLI: a desktop/mobile interface for managing
projects, tasks, conversations, and agentic coding workflows.

Start with @docs/project.md — a short index pointing to topical docs. Pull only
the sections you need rather than loading everything: `claude-sdk-integration.md`,
`conversation-management.md`, `agentic-loop.md`, `task-management.md`,
`authentication.md`, `api-reference.md`. Each lists its key files at the bottom.

> **Machine/operator-specific context** (your dev-box URLs, deploy layout,
> manual-test fixtures, auth tokens) does not belong in this file — it goes in
> `CLAUDE.local.md`, which is gitignored and loads automatically alongside this
> file. Copy `CLAUDE.local.md.example` to `CLAUDE.local.md` and fill it in for
> your environment.

## Tech Stack

- **Frontend**: React 18, Vite, Tailwind CSS, CodeMirror
- **Backend**: Node.js, Express, WebSocket (ws)
- **Database**: SQLite (better-sqlite3) at `server/database/bottega.db`

**TypeScript-only.** Every source file is `.ts`/`.tsx` — `tsconfig.json` sets
`allowJs: false` and a `pnpm guard-no-js` prelint hook fails CI on any new
`.js`/`.jsx` outside `node_modules`/`dist`/`coverage`. Don't add JavaScript
files; if you genuinely need to (e.g. a third-party script that ships as
`.js`), update the allowlist in `scripts/guard-no-js.ts`.

## Data Architecture

### What SQLite stores (server/database/bottega.db)

The database stores **metadata only** — projects, tasks, conversations, users:

- `projects` — id, name, repo_folder_path
- `tasks` — id, project_id, title, status, workflow flags
- `conversations` — id, task_id, claude_conversation_id, session_path
- `task_agent_runs` — id, task_id, agent_type, status, conversation_id

**Schema:** `server/database/init.sql`

### Where messages are stored

**Messages live in SQLite**, in two tables:
- `messages` — one row per SDK transcript entry, PK `(project_key, session_id, subpath, uuid)`
- `session_summaries` — incrementally-folded summaries per session

We register a custom `SqliteSessionStore` (`server/services/sqliteSessionStore.ts`)
with the Claude Agent SDK via the `sessionStore` option. The SDK calls our
`append/load/...` methods for every conversation; SQLite is the single source of
truth. The SDK still writes its own `.jsonl` files under `CLAUDE_CONFIG_DIR` for
its private resume path, but **runtime code never reads them** — the only file
that knows JSONL exists is `scripts/data-migrations/import-jsonl-to-sqlite.ts`.

**Query conversation metadata:**
```bash
sqlite3 server/database/bottega.db "SELECT id, task_id, claude_conversation_id, session_path FROM conversations WHERE id = <ID>;"
```

**Inspect messages for a conversation:**
```bash
# Via API
curl http://localhost:3001/api/conversations/<ID>/messages

# Or directly in SQLite — project_key = repo_folder_path (or session_path) with /. → -
sqlite3 server/database/bottega.db "SELECT seq, json_extract(entry_json,'$.type') AS type FROM messages WHERE session_id = (SELECT claude_conversation_id FROM conversations WHERE id = <ID>) ORDER BY seq"
```

### URL routing

URLs follow the pattern `/projects/:projectId/tasks/:taskId/chat/:conversationId`.
All IDs correspond to SQLite row IDs, so a URL like
`/projects/178/tasks/562/chat/2683` maps directly to row lookups:
```bash
sqlite3 server/database/bottega.db "SELECT * FROM conversations WHERE id = 2683;"
sqlite3 server/database/bottega.db "SELECT * FROM tasks WHERE id = 562;"
sqlite3 server/database/bottega.db "SELECT * FROM projects WHERE id = 178;"
```

## Third-Party APIs & Libraries

Before using any external API or library:
1. **Verify with Context7 MCP** — `resolve-library-id` → `get-library-docs`
2. **If insufficient**, use `WebFetch` on official docs
3. **Never assume** method names, parameters, or response formats

## API request validation

Every Express route handler that reads `req.body`, `req.params`, or `req.query`
must validate that input through a zod schema before touching it. Schemas live
next to their HTTP contracts in `shared/schemas/` (`auth.ts`, `admin.ts`,
`projects.ts`, `tasks.ts`, plus a `_common.ts` for shared shapes like
`IdParamsSchema`). Each schema also exports its inferred type via
`z.infer<typeof X>`, so backend handlers and frontend callers share a single
source of truth.

The boundary itself is three middleware factories in
`server/middleware/validate.ts`: `validateBody`, `validateParams`,
`validateQuery`. They run `schema.safeParse()` on the corresponding slice of
`req`, attach the parsed value to `req.validated.body/.params/.query`, and on
failure short-circuit with HTTP 400 and
`{ error: 'Validation failed', issues: ZodIssue[] }`. Handlers read fields off
`req.validated!.body as <BodyType>` (the field is `unknown` at the type level —
each route casts to the schema it asked for). When adding a new route, add a
schema to `shared/schemas/`, plug the matching `validate*` middleware in front
of the handler, and delete any ad-hoc shape checks that the schema now enforces.

## pnpm scripts

This project uses **pnpm** (pinned via `packageManager` in `package.json` and
provisioned by Corepack — run `corepack enable` once after cloning).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vdaubry/bottega](https://github.com/vdaubry/bottega) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
