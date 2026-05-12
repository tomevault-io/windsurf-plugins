---
trigger: always_on
description: A GUI for Claude Code and Codex, built for multi-agent orchestration — with
---

# Crispy

A GUI for Claude Code and Codex, built for multi-agent orchestration — with
'superthink' adversarial verification, agent memory, and Discord remote
access. VS Code / Cursor extension, standalone browser app, and Windows
desktop app.

<!-- TAGLINE LOCKED — do not change without explicit user approval -->

Claude and Codex adapters are shipping. Gemini CLI and OpenCode are next.

## Architecture

Three layers: **core** (`src/core/`), **host** (`src/host/`), **webview**
(`src/webview/`). Core owns state and logic; host is a thin RPC router;
webview derives state client-side from channel events.

**Full detail:** [`architecture.md`](./architecture.md)

### Core (`src/core/`) — logic and state

Vendor-agnostic contracts, session orchestration, persistence. Free functions
with module-level state, not classes — no `this`.

- Universal types (`transcript.ts`) — vendor-agnostic; use `metadata` bag for vendor extras
- Adapter contract (`agent-adapter.ts`) — new adapters implement, never extend
- Event types (`channel-events.ts`) — idle/active/approval state transitions
- Session multiplexer (`session-channel.ts`) — dumb pub/sub, no transformation; adapters emit, UI interprets
- Session orchestration (`session-manager.ts`) — adapter registry, create/load/fork, pending→real ID re-keying
- Session list (`session-list-manager.ts`) — background rescan, push notifications
- Persistence (`activity-index.ts`) — owns all `~/.crispy/` I/O
- File service (`file-service.ts`) — git-backed file listing, @-mention indexing
- Provider config (`provider-config.ts`) — dynamic provider CRUD, hot-reload watch
- Per-vendor adapters (`adapters/claude/`, `adapters/codex/`)

**State ownership:** session lifecycle, pending approvals, adapter registry,
disk persistence. **When adding behavior, ask:** does this touch session
state, transcript data, or vendor logic? → it belongs here.

### Host (`src/host/`) — transport and wiring

Thin RPC router. Creates transport abstractions, routes client requests to
core, wires adapters at startup. No business logic.

- RPC protocol (`client-connection.ts`) — JSON-RPC wire protocol, request/response correlation
- Adapter registration (`adapter-registry.ts`) — one-shot startup wiring
- Internal dispatch (`agent-dispatch.ts`) — in-process loopback client
- VS Code host (`webview-host.ts`) — panel management, VS Code–specific methods
- Dev server (`dev-server.ts`) — HTTP + WebSocket on port 3456

**State ownership:** connection lifecycle, subscription tracking — nothing
else. **When adding behavior, ask:** is this just routing a call or managing
a transport? → it belongs here. Anything else → push down to core.

### Webview (`src/webview/`) — UI rendering

React 19, esbuild, vanilla CSS. Derives all state client-side from channel
events. No direct file I/O or process spawning. **State ownership:** UI-only
(scroll position, panel expand/collapse, render mode, playback position).

- Provider cascade (`context/`) — Transport → Environment → Session → FileIndex → Preferences
- Blocks rendering pipeline (`blocks/`) — normalize → dispatch → tool-specific views
- Tool views (`blocks/views/`) — per-tool compact/expanded renderers
- Components (`components/`) — TranscriptViewer, ControlPanel, approval flow, session browser
- Hooks (`hooks/`) — reusable React logic (transcript, status, approvals, scroll)
- Alternative renderers (`renderers/`) — Compact and YAML modes

**Extending the UI:**
- New tool → metadata in `tool-definitions.ts`, view in `blocks/views/`, register in `register-views.ts`
- New approval type → variant in `ApprovalContent.tsx`, new component
- New context → file in `context/`, add to cascade in `App.tsx`

## Do not

- **Add business logic to host.** `client-connection.ts` routes RPCs — it
  does not interpret session events, manage state transitions, or reach into
  core internals. The core-plumbing refactor (beaf021) cleaned up exactly
  this: duplicated re-keying, copy-pasted pending-channel boilerplate, and
  `getChannel()` leaking core state to host. Don't recreate it.
- **Add vendor-specific fields to `transcript.ts`.** Use the `metadata` bag.
  Rule of thumb: all vendors need it → base type; one vendor needs it →
  metadata; unsure → metadata first, promote later.
- **Extend the adapter contract.** New adapters implement `AgentAdapter` —
  they do not add fields to it.
- **Add switch statements to `BlocksEntry` or `BlocksBlockRenderer`.** Extend
  via `tool-definitions.ts` → `register-views.ts` → new view component.
- **Replace `useRef` with `useMemo` in `BlocksToolRegistryContext.tsx`.**
  useMemo's cache is discarded unpredictably — causes Task children to vanish.
- **Modify generated Codex protocol files** (`adapters/codex/protocol/**/*.ts`).
  They're auto-generated from ts-rs.
- **Write to `~/.crispy/` outside `activity-index.ts`.** It owns all reads/writes.
- **Use `console.log/warn/error` in `src/core/` or `src/mcp/`.** Use `log()` from
  `src/core/log.ts` instead. `console.*` calls bypass the structured ring buffer
  and won't appear in the Rosie Log panel or respond to `CRISPY_LOG_LEVEL`
  gating. Only exception: fatal subprocess paths where the process is dying and
  structured logging may be unavailable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSylvester/crispy](https://github.com/TheSylvester/crispy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
