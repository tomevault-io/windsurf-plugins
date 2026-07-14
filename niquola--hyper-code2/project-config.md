---
trigger: always_on
description: hyper-code2 — procedural Bun codebase with a self-extending agent at `/`.
---


# hyper-code2

Procedural TypeScript on Bun. Functions + data + REPL. Inspired by [proc-ts](../proc-ts). One tiny HTTP server, one agent at `/` driven by `evalCode` only, all code hot-reloadable.

## Runtime environment

- **Bun, not Node.js.** Use `bun`, `bun test`, `bun install`, `bunx`, `bun build`. `.env` loads automatically.
- Prefer Bun built-ins over npm packages:
  - `Bun.serve()`, `Bun.file()`, `Bun.write()`, `Bun.$` shell, `Bun.Glob`, `Bun.spawn`, `Bun.hash`, `Bun.CryptoHasher`, `Bun.password`, `Bun.TOML.parse`, `Bun.gzipSync`, `Bun.randomUUIDv7`, `Bun.inspect`, `Bun.markdown.html`.
  - `bun:sqlite`, `Bun.sql` (Postgres), `Bun.redis`, `Bun.s3()`.
- Tests: `bun:test` — Jest-compatible. Filename must match `*.test.ts` for auto-discovery. `.env.test` loads automatically when `NODE_ENV=test` (bun test sets this).

## Architecture: procedural ctx.fns

**One function per file. Folder = namespace.** Files are loaded into `ctx.fns.<module>.<fn>`.

```
src/
  $main.ts                 entry: loadFns → genTypes → loadRoutes → server.start
  $type_Context.ts         global `Context` type
  ctx_ns.d.ts              AUTO-GEN — FnsRegistry, RootFns, types.*
  genTypes.ts              ctx.genTypes — rescans src/ + .hyper/, writes ctx_ns.d.ts
  $route_GET.ts            GET /  (single-page chat UI)

  llm/                     ctx.fns.llm.* — LLM layer
    resolveEndpoint.ts     (ctx, "provider:modelId") → {url, apiKey, modelId, provider}
    stream.ts              stateless /v1/chat/completions with tool_calls + streaming (OpenAI-compat)

  agent/                   ctx.fns.agent.* — the agent runtime
    SYSTEM_PROMPT.md       editable prompt (authoritative)
    $type_Agent.ts
    start / stream / run / compact / clear / stop / systemPrompt
    renderMarkdown / highlight
    $route_*.ts            HTTP: POST/GET/DELETE /agent, POST /agent/stop

  repl/                    ctx.fns.repl.*
    eval.ts                new Function("ctx", ...) + extra bindings
    load.ts                hot-reload a fn or folder from src/ or .hyper/
    $route__POST.ts        POST /repl — executes arbitrary JS

  server/                  ctx.fns.server.*
    $start.ts              Bun.serve with dynamic dispatch via server.match
    match.ts               path matcher (supports :params)

  http/                    ctx.fns.http.*
    loadRoutes.ts          scans $route_*.ts files into ctx.routes

  db/                      ctx.fns.db.* — shared SQLite infrastructure
    connect.ts             opens db (WAL), stores on ctx.state.db
    migrate.ts             scans **/$migrate_<ts>_<name>.up.sql, applies pending in ts-order, tracks in _migrations
    exec.ts                exec(ctx, {sql, params?}) → {changes, lastInsertRowid}
    select.ts              select<T>(ctx, {sql, params?}) → T[]
    insert.ts              insert(ctx, {table, row}) → {changes, lastInsertRowid}

  session/                 ctx.fns.session.* — per-agent persistence (uses db/)
    $migrate_20260418000000_init.up.sql  — baseline schema (agents, messages, events)
    save / load / loadAll / list / search / delete
```

## Conventions

- `export default async function (ctx: Context, opts: {...})` — **anonymous**, no function name. Every fn takes `ctx` first, then a single options-object.
- **Universal calling convention**: `ctx.fns.<ns>.<fn>(ctx, { ...opts })`. Single rule, no per-fn argument-order recall. The agent reads the destructuring pattern via `fn.toString()` rather than guessing positional arguments.
  - Single-arg fns still take an opts wrapper: `files.read(ctx, { path })`, not `files.read(ctx, path)`.
  - Zero-arg fns are `(ctx)` only (no opts wrapper): `session.list(ctx)`, `agent.workerLoop(ctx)`, `llm.listModels(ctx)`.
  - Optional fields go inside opts as `opts.x ?? default`.
- Cross-file calls go through `ctx.fns.<ns>.<fn>(ctx, { ... })`. **No cross-imports between project files.** Only `import` from `bun`, `node:*`, or third-party.
- Types are global via auto-generated `ctx_ns.d.ts`:
  - `src/<mod>/$type_<Name>.ts` → `types.<mod>.<Name>` globally.
  - `src/$type_Context.ts` → global `Context` (composed with `FnsRegistry` + `RootFns`).
  - Never `import type { Agent }` — use `types.agent.Agent` directly.
- Special filenames (`$` prefix stripped when registering in `ctx.fns`):
  - `$main.ts` — entry point, NOT loaded into ctx.fns.
  - `$test.ts` — deprecated; use `*.test.ts` for bun test discovery.
  - `$route_<path>_<METHOD>.ts` — HTTP route. `_` in path = `/`, `$foo` = `:foo` param. See `src/http/loadRoutes.ts`.
  - `$type_<Name>.ts` — type declaration, compile-time only.
  - Other `$<name>.ts` (e.g. `$start.ts`) — regular function, loaded as `ctx.fns.<mod>.<name>`.
- Test files named `*.test.ts`. `bun test` picks them up automatically.

## Routes

Dynamic dispatch — mutations to `ctx.routes` are effective on the next request without `server.reload()`. See `src/server/$start.ts` and `src/server/match.ts`.

Current live routes:
- `GET /` — chat UI (Tailwind via CDN)
- `POST /agent` — send a message (non-blocking, queues run in background)
- `GET /agent?offset=N` — poll for new events (`isStreaming`, `nextOffset`)
- `DELETE /agent` — reset agent
- `POST /agent/stop` — abort current run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niquola/hyper-code2](https://github.com/niquola/hyper-code2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
