---
trigger: always_on
description: This repository implements the **Ngent** — a local-first Go service exposing HTTP/JSON APIs and SSE streaming for multi-client, multi-thread ACP-compatible agent turns.
---

# AGENTS

This repository implements the **Ngent** — a local-first Go service exposing HTTP/JSON APIs and SSE streaming for multi-client, multi-thread ACP-compatible agent turns.

ACP protocol reference: <https://agentclientprotocol.com/>

## Mandatory Rules

- MUST use Go 1.24.
- MUST use scope-appropriate validation:
  - frontend-only changes limited to `internal/webui/web/**` (plus optional docs updates) MUST pass `cd internal/webui/web && npm run build`; local `go test ./...` is not required.
  - any backend change, mixed frontend/backend change, or change outside `internal/webui/web/**` that can affect runtime behavior MUST keep `go test ./...` passing.
- MUST default to local-only bind (`127.0.0.1:8686`); require `--allow-public=true` for LAN-accessible binds.
- MUST support local-only mode when `--allow-public=false` (loopback-only binds).
- MUST validate inputs:
  - `agent` must be in server allowlist.
  - `cwd` must be an absolute path.
  - `cwd` must be inside configured allowed roots, otherwise reject.
- MUST enforce concurrency model:
  - one active turn per `(thread, session)` scope at a time.
  - thread-level destructive or shared-state operations must still fail/lock at whole-thread scope.
  - cancel must take effect quickly.
  - permission workflow is fail-closed by default.
- MUST keep stdout and HTTP output protocol-only.
- MUST send logs to stderr using the repo's human-readable logger; access logs should be easy to scan at a glance and ACP debug logs must still redact secrets.
- MUST redact sensitive information in logs and errors.

## Long-Term Memory Rules

- At the end of every completed phase, update `PROGRESS.md`.
- Write key technical/product decisions to `docs/DECISIONS.md`.
- Track known limitations and risks in `docs/KNOWN_ISSUES.md`.
- Keep acceptance checklist in `docs/ACCEPTANCE.md`.
- Keep implementation design in `docs/SPEC.md`.

---

## Frontend (Web UI)

Milestones F0–F9 are complete. The frontend is a no-framework Vite + TypeScript SPA embedded in the Go binary via `//go:embed web/dist`.

### Stack

- **Build**: Vite 6 + TypeScript 5.6, output to `internal/webui/web/dist/`.
- **Rendering**: vanilla DOM, no virtual DOM or component framework.
- **Markdown**: `marked@^9` with custom renderer; `highlight.js` core (go/ts/js/python/bash/json/yaml).
- **Go embed**: `internal/webui/webui.go` embeds `web/dist`, exposes `webui.Handler()` — an SPA fallback handler registered at lowest priority in `httpapi`.
- **Build targets** (Makefile):
  - `make build-web` — `npm ci && npm run build` inside `internal/webui/web/`.
  - `make build` — `build-web` then `go build ./...`.
  - `make run` — `build-web` then `go run ./cmd/ngent`.

### Source file map

All files live under `internal/webui/web/src/`.

| File | Purpose |
|---|---|
| `types.ts` | All TypeScript interfaces: `AgentInfo`, `Thread`, `Turn`, `TurnEvent`, `Message`, `PermissionRequest`, `StreamState`, `AppState`. |
| `utils.ts` | `generateUUID`, `formatTimestamp`, `formatRelativeTime`, `isAbsolutePath`, `escHtml`, `debounce`. |
| `store.ts` | Singleton `AppStore`: `get()` / `set(patch)` / `subscribe(fn) → unsub`. Persists `authToken`, `serverUrl`, `theme` to `localStorage` (keys `ngent:*`). Never persists runtime data. |
| `api.ts` | `ApiClient` singleton (`api`): reads `serverUrl`/`authToken` from store on every call and sends a fixed compatibility `X-Client-ID` header internally. Methods: `getAgents`, `getThreads`, `getHistory`, `createThread`, `startTurn`, `cancelTurn`, `resolvePermission`. |
| `sse.ts` | `TurnStream` class: POST SSE via `fetch` + `ReadableStream` (not `EventSource` — lacks POST/custom-header support). Parses `event:\ndata:\n\n` blocks. Callbacks: `onTurnStarted`, `onDelta`, `onCompleted`, `onError`, `onPermissionRequired`, `onDisconnect`. `abort()` sets `terminated=true` and aborts fetch. |
| `markdown.ts` | Configures `marked` renderer: `html()` → `escHtml()` (XSS guard); `code()` → `.code-block` with hljs highlight, copy button, optional "Show all N lines" fold (>20 lines). Exports `renderMarkdown(text)` and `bindMarkdownControls(container)` (idempotent, `data-bound` guard). |
| `main.ts` | App entry: `renderShell()`, `init()`, all DOM wiring. See patterns below. |
| `components/settings-panel.ts` | Slide-in drawer: Bearer Token, Server URL, Light/Dark/System theme toggle. |
| `components/new-thread-modal.ts` | Modal: agent card grid (radio, disabled for unavailable), absolute-path CWD validation, optional title, collapsible JSON agent-options textarea. |
| `components/permission-card.ts` | `mountPermissionCard(listEl, event)`: appends ephemeral card with a 2 hour countdown, renders all agent-advertised permission options (falls back to Allow/Deny when none are provided), and shows resolved states. Calls `api.resolvePermission()`; ignores 409. |
| `style.css` | All styles. CSS custom properties (`--bg`, `--accent`, etc.) with `[data-theme="dark"]` override block. hljs tokens use CSS variables (`--hljs-fg`, `--hljs-kw`, …). |

### Key patterns in `main.ts`

**Streaming sentinel** — prevents `updateMessageList()` from wiping the live bubble:
```
let activeStreamMsgId: string | null = null
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beyond5959/ngent](https://github.com/beyond5959/ngent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
