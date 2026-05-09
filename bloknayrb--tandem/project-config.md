---
trigger: always_on
description: - `npm run dev:server` -- Backend: Hocuspocus on :3478 + MCP HTTP on :3479
---

# Tandem -- Collaborative AI-Human Document Editor

## Quick Reference
- `npm run dev:server` -- Backend: Hocuspocus on :3478 + MCP HTTP on :3479
- `npm run dev:client` -- Frontend: Vite on :5173
- `npm run dev:standalone` -- Both frontend + backend (via concurrently)
- `npm run dev` -- Alias for `vite` (frontend only)
- `npm run build` -- Production build: typecheck + vite build + tsup -> `dist/server/`, `dist/channel/`, `dist/cli/`, `dist/client/`
- `npm run build:server` -- Bundle server + channel + CLI via tsup only
- `npm run start:server` -- Run bundled server (`node dist/server/index.js`)
- `npm run typecheck` -- Type-check server + client without emitting
- `npm run doctor` -- Diagnose setup issues (Node version, .mcp.json, server health, ports)
- `npm test` -- Run vitest (unit tests)
- `npm run test:e2e` -- Run Playwright E2E tests (auto-starts servers via webServer config)
- `npm run test:e2e:ui` -- Playwright UI mode for interactive E2E debugging
- **Start the server before connecting Claude Code.** `npm run dev:standalone` runs both. Vite hot-reloads client code; restart `dev:server` then `/mcp` in Claude Code for server changes.

## Documentation
- [MCP Tool Reference](docs/mcp-tools.md) -- All 31 MCP tools + channel API endpoints
- [Architecture](docs/architecture.md) -- Diagrams, data flows, coordinate systems, file map
- [Workflows](docs/workflows.md) -- Real-world usage patterns
- [Agent Workflow](docs/agent-workflow.md) -- 10-step agent-driven issue pipeline (`/issue-pipeline`)
- [Roadmap](docs/roadmap.md) -- Phase 2+ roadmap, future extensions
- [Design Decisions](docs/decisions.md) -- ADRs (001-024)
- [Lessons Learned](docs/lessons-learned.md) -- 46 lessons including E2E testing gotchas

## Critical Rules

These WILL break things if violated:

1. **Y.Map key strings from constants only.** Use `Y_MAP_ANNOTATIONS`, `Y_MAP_AWARENESS`, etc. from `shared/constants.ts` -- never raw string literals for Y.Map keys.
2. **Origin-tag server writes.** All server-side Y.Map writes must tag the transaction with an origin constant exported from `src/server/events/queue.ts`. Use `MCP_ORIGIN` (`"mcp"`) for user-intent writes from MCP tools — `doc.transact(() => { ... }, MCP_ORIGIN)`. Use `FILE_SYNC_ORIGIN` (`"file-sync"`) for echoes from the durable-annotation file-writer / file-watcher reload path. The durable-annotation sync observer skips `FILE_SYNC_ORIGIN` transactions (prevents re-persisting state just loaded from disk), and the channel event queue skips BOTH origins — `MCP_ORIGIN` because external consumers already saw the MCP call, `FILE_SYNC_ORIGIN` because file reloads are not user events.
3. **stdout is reserved.** `console.log/warn/info` all redirect to stderr in `index.ts` (defense-in-depth for stdio fallback). If you add a dependency that logs to stdout, it will corrupt the MCP wire in stdio mode.
4. **Ranges use `validateRange()` + `anchoredRange()`**, not raw offsets. `anchoredRange()` creates both flat + Yjs RelativePosition in one call.
5. **`tandem_getTextContent` uses `extractText()`, never `extractMarkdown()`.** Even for .md files. `extractMarkdown()` shifts character offsets relative to the annotation coordinate system. If you need actual markdown, use `tandem_save` and read the file.
6. **`tandem_edit` rejects heading markup ranges.** Ranges that overlap heading prefixes (e.g., `## `) return INVALID_RANGE -- target text content only.
7. **E2E tests use `data-testid` attributes** (kebab-case). Key selectors: `accept-btn`, `dismiss-btn`, `edit-btn`, `review-mode-btn`, `annotation-card-{id}`, `tab-{id}`, `file-open-dialog`, `file-path-input`, `open-file-btn`, `toast-container`.

## Architecture

Three layers: Editor (Tiptap in Tauri desktop or browser) <-> Tandem Server (Hocuspocus on :3478 + MCP HTTP on :3479) <-> Claude Code. The desktop app is the primary distribution; npm global install opens the same editor in a browser. Channel shim (`src/channel/`) pushes real-time events to Claude Code via SSE, replacing polling.

Key files for navigation:
- `src/cli/index.ts` -- CLI entrypoint (`tandem` command), arg parsing, dispatches to start/setup
- `src/cli/setup.ts` -- `tandem setup`: auto-detect Claude installs, write MCP config atomically
- `src/cli/start.ts` -- `tandem start`: spawn server (opens editor via `TANDEM_OPEN_BROWSER=1`)
- `src/server/index.ts` -- Entry point, port binding, console redirect
- `src/server/open-browser.ts` -- Cross-platform browser launcher for npm-install path (execFile-based)
- `src/server/mcp/` -- Tool definitions, `api-routes.ts`, `channel-routes.ts`, `file-opener.ts`, `document-service.ts`
- `src/server/positions.ts` -- Server coordinate conversions (`validateRange`, `anchoredRange`, `resolveToElement`, `refreshRange`)
- `src/server/events/` -- Channel event infrastructure (Y.Map observers, SSE)
- `src/client/` -- Tiptap editor, React components, hooks (`useYjsSync`, `useTabOrder`, `useTutorial`, `useNotifications`)
- `src/shared/` -- Types (`types.ts`), constants (`constants.ts`), offsets (`offsets.ts`), position types (`positions/`)

Full file-level detail: [docs/architecture.md](docs/architecture.md#file-map)

## Key Patterns
- All document mutations go through the server's Y.Doc -> changes sync to editor via Hocuspocus

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloknayrb/tandem](https://github.com/bloknayrb/tandem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
