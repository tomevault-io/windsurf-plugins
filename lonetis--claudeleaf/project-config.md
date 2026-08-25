---
trigger: always_on
description: Internal documentation for **Claudeleaf** — the single source of truth for how the project
---

# CLAUDE.md

Internal documentation for **Claudeleaf** — the single source of truth for how the project
works. (User-facing docs live in `README.md`.)

## What this is

Claudeleaf bridges an Overleaf *account* to an API so Claude — or any agent/tool — can edit
any of the account's projects programmatically through Overleaf's *live* web-editing channel
(not git). Edits appear to collaborators in real time, attributed to the signed-in user. One
instance is account-scoped: it lists every accessible project and edits any of them (by id or
name); each project is backed by its own realtime connection.

TypeScript / Node, delivered as three layers over one core:

- a **TypeScript SDK** (`OverleafClient`, account-scoped; `ProjectSession`, per-project),
- a **CLI** (`claudeleaf`) — thin command wrapper,
- an **MCP server** — thin tool wrapper, for Claude Code.

## Tech stack

- Node.js 20+, TypeScript (ESM, `NodeNext`), built with `tsc` to `dist/`. npm for tooling.
- `ws` — the realtime WebSocket transport.
- global `fetch` (Node) — the Socket.IO handshake and the REST tree/projects endpoints.
- `playwright-core` — drives the user's *installed* Chrome/Edge for the one-time manual login
  (no browser download). Loaded lazily (dynamic `import` in `auth.browserLogin`) so the
  SDK/CLI/MCP stay lightweight — important for `npx`.
- `@modelcontextprotocol/sdk` + `zod` — the MCP server.
- `vitest` — unit + live integration tests. `tsx` — run TS directly (dev / MCP launch).

## Project structure (`src/`, flat)

```
src/
  index.ts        public API: OverleafClient, ProjectSession, Config, Document, errors
  config.ts       account-level Config (no project/credentials); env parsing
  errors.ts       error hierarchy (ClaudeleafError + subclasses)
  util.ts         async helpers: Mutex, Deferred, sleep, withTimeout
  protocol.ts     Socket.IO 0.9 frame encode/decode (pure)
  ot.ts           ShareJS ops, offset<->rowcol, diff, wire encoding, git-blob hash (pure)
  logParser.ts    parse pdfTeX/LaTeX output.log into errors/warnings/bad-boxes (pure)
  document.ts     Document: local mirror of one open doc (text + version + generation)
  types.ts        shared types (Entity, ProjectSummary, ProjectInfo, CompileResult, ...)
  realtime.ts     RealtimeConnection(projectId): WS transport, acks, heartbeat, reconnect
  auth.ts         SessionManager: manual browser login + session cookie cache
  rest.ts         listProjects() + RestClient(projectId) for tree CRUD + compile/output
  session.ts      ProjectSession: one project's realtime connection, tree, edits, compile
  client.ts       OverleafClient: account-scoped; listProjects + per-project delegation
  cli.ts          CLI (login, doctor, projects, doc ops, compile, mcp); the bin
  mcpServer.ts    MCP server exposing the SDK as tools (each takes a project)
test/             vitest unit (no network) + integration/mcp e2e (live, auto-skip)
```

## Architecture & data flow

```
Config (env, all optional) ──> OverleafClient (account)
                                  ├─ SessionManager ──(manual browser login, cached cookie)
                                  ├─ rest.listProjects()  ─ POST /api/project
                                  └─ ProjectSession(projectId)   one per project, cached
                                       ├─ RealtimeConnection(projectId)   Socket.IO 0.9 (ws)
                                       └─ RestClient(projectId)           tree CRUD (fetch)
OverleafClient ──> CLI  and  MCP server   (each op takes a project id/name)
```

- **OverleafClient** is account-scoped. It resolves a project (24-hex id, or unique name via
  `listProjects`), lazily creates and caches a `ProjectSession` per project, and delegates
  document operations to it. All methods are `async`. A session whose connection has
  permanently failed is rebuilt on next access.
- **ProjectSession** owns one project's realtime connection + tree + open documents and all
  editing logic. This is where the reliability/concurrency model lives.
- **SessionManager** returns the cached auth cookie (sync), or throws directing the user to
  `claudeleaf login` — it never automates credentials.

## Authentication (manual)

Overleaf's login is protected by reCAPTCHA (and may use SSO/2FA), so Claudeleaf does not
automate credentials. `claudeleaf login` (→ `SessionManager.login`) drives the user's
*installed* browser via `playwright-core` — `launchPersistentContext` with `channel: "chrome"`,
falling back to `"msedge"`, then any Playwright-managed Chromium, else a clear error. A real
headed browser is what passes Overleaf's invisible reCAPTCHA reliably; using the system browser
also means **no Playwright browser download** (so `npx claudeleaf` stays light). The user signs
in by hand; once `ol-user_id` appears it extracts the `overleaf_session2` cookie and caches it
(`~/.claudeleaf/session.json`, mode 0600). The lightweight fetch/WebSocket layers reuse the
cookie; the browser is only needed again when it expires.

## Listing projects

`rest.listProjects` scrapes the dashboard CSRF token then calls the dashboard JSON API
`POST /api/project`, returning `{id, name, accessLevel, lastUpdated, owner, archived,
trashed}` per project. Account-level — no project id needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lonetis/claudeleaf](https://github.com/lonetis/claudeleaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
