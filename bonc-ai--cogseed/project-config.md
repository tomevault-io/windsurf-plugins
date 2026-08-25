---
trigger: always_on
description: Prompt context only: keep hard constraints, short rationale, and traps already hit. Implementation details belong in source headers and tests.
---

# AGENTS.md

Prompt context only: keep hard constraints, short rationale, and traps already hit. Implementation details belong in source headers and tests.

## Repo

CogSeed desktop companion agent (Electron). Main is TypeScript under `src/main`, renderer is vanilla HTML/CSS/JS under `src/renderer`.

- Quick gates: `npm run typecheck` (tsc --noEmit), `npm test` (js + resources), `./run.sh` to start.

## Boundary

Single-process Electron app. Main is a Node backend, renderer is vanilla HTML/CSS/JS, and IPC is the only app communication path.

- No HTTP server, no occupied port, and no local auth layer in main.
- Renderer access goes through the canonical `contextBridge` allow-list API `window.cogseed.{invoke, stream}`.
- No TypeScript/JSX/bundler in the renderer; classic scripts only.
- `src/main/preload.js` must remain `.js`; preload does not run the tsx hook.
- LLM calls use the in-process `core-agent` loaded dynamically through `import('#core-agent')`.
- Local CLI agents are the explicit child-process exception. `features/local_agents/runner.ts` is the only CLI dispatch spawn path.
- The isolated worker process is spawned only through its dedicated `worker-process.ts` entry and speaks the Runtime JSONL protocol; no IPC handler/renderer code may spawn it directly. Inside that isolated worker, Runtime tool execution is limited to the dedicated kernel tool choke points: shell commands through `shell-tools.ts`, skill scripts through `skill-tools.ts` → `bin/run-skill.cjs`.
- MCP stdio connectors spawn only through `features/connectors/mcp-client.ts`.
- User data is mostly JSON/JSONL for readability and sync friendliness; sqlite is reserved for the KB vector store.
- macOS and Windows are primary. Platform branches need platform-specific verification.
- New npm dependencies require prior discussion; renderer third-party JS/CSS goes under `src/renderer/vendor/`, not npm.

## Layering

- `ipc/`: validate args and call features; no business logic.
- `features/`: business workflows; may use storage, paths, prompts, model, util, and sibling features.
- `model/` and `model/core-agent/`: model-call adapters and tool plumbing; do not read/write business data under `data/`.
- `util/`: pure/foundational helpers; never reverse-import features/model.
- `storage.ts`, `paths.ts`, and path sandbox helpers are the storage/path choke points.
- `i18n.ts` may read locales but must not import features/model.

Additional rules:

- Feature functions handling user-private data take `userId` as the first argument.
- Boot-time async work registers through `util/boot_init.ts`, not raw startup timers or async IIFEs.
- `#core-agent` is dynamic-import only. Static import loads dependencies before the SDK timeout patch and can break ESM resolution.
- New core-agent tools must be registered in `tool-catalog.ts::TOOL_CATALOG` and runner wiring. Tool descriptions live in SDK `tools[]`, not in a duplicated prompt tool list.
- File-class tools must check `util/path-sandbox.isPathAllowed` at entry.
- Tool results go through `util/tool-result-cap.ts`.

## Prompt Files

`src/main/prompts/*.md` is LLM-facing source. It must not contain:

- Product/brand names.
- Real OS paths.
- Project source/data directory literals.
- Hard-coded tool catalogs.

Runtime-volatile prompt fields go in one trailing `## Runtime injection` section. Static rules stay first so cache prefixes remain stable.

## Data Domains

All user-scoped data lives under `<container>/data/<uid>/{cloud,local}/`.

- `uid` is an opaque single path segment. Do not parse it or embed it into session ids.
- `cloud/` is syncable user-private state: chats, resumable sessions, attachments, artifacts, saved apps, contexts source files, memory, custom agents/skills, projects, marketplace install manifest, auto tasks, and user config.
- `local/` is machine-private state: account/session cache, marketplace installed content, caches, indexes, vector DB, workspace selection, tool-result spills, local-agent archives, and dev archives.
- Never cache uid-derived paths as module-level constants. Get the active uid at use time.
- Project membership is an index field on a conversation. Do not encode `project_id` into paths, cids, or session ids.
- Project lists are directory scans; do not restore an aggregate `projects/_index.json`.

## Conversations And Group Chat

Session ids are `<kind>-<tail>`; user scoping comes from the path root, not the filename. Add new kinds by updating the session-store allowlist and all session-kind gates.

Main rules:

- Commander, agent-worker, skill-edit, agent-edit, and one-shot sessions are separate session files from UI message lists.
- Group chat dispatch goes through `features/group_chat/bus.ts::enqueue`. Do not create parallel enqueue/scheduling paths.
- Agent workers read only their visibility slice; never the full conversation jsonl.
- LLM dispatch is structured (`dispatch_to`, `plan_set`), not `@name` in prose.
- User abort is never a transient retry. Network retry patterns must stay network-specific.
- Group abort is the single stop path for all actors.
- Infinite-loop protection is turn-count based, paired with idle timeout; do not replace it with total wall-clock timeout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bonc-ai/cogseed](https://github.com/bonc-ai/cogseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
