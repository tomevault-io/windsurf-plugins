---
trigger: always_on
description: - `background/aid/api/` — one file per GraphQL query or mutation (never inline queries elsewhere)
---

# MCA Architecture

## Folder Conventions

- `background/aid/api/` — one file per GraphQL query or mutation (never inline queries elsewhere)
- `background/openrouter/api/` — one file per OpenRouter API call
- `background/protocol.js` — single source of truth for all message types between popup, content, and background
- `background/types.js` — all JSDoc typedefs live here, import via `@typedef {import('./types.js').TypeName}`
- `background/storage.js` — all chrome.storage access goes through these helpers
- `background/constants.js` — shared constants (API URLs, defaults). Background modules only — content scripts inline their own

## Service Worker Rules

- **Never store persistent state in module-scope variables.** Service workers go dormant after ~30s. Use `chrome.storage` via `storage.js`
- **Register all listeners synchronously** at the top level of `background/index.js` (Chrome requirement)
- **Message routing** lives in `background/index.js` as a switch statement. Import protocol constants, not raw strings
- **Handler implementations** live in `background/handlers.js` — one exported function per message type

## Content Script Rules

- Content scripts **cannot use ES module imports**. Inline any needed constants
- Reference `background/protocol.js` in comments when using message type strings
- Token extraction uses `window.postMessage` between `inject.js` (page context) and `index.js` (content script context)

## Error Handling

- Async operations return `{ ok: boolean, value?, error? }` (the `Result` type from `types.js`)
- GraphQL errors are caught in `gql.js` and thrown as standard Errors
- Never let errors silently disappear — log with `[MCA]` prefix

## Agent Workspace

- `dev/agents/` contains markdown files for agents to track work across sessions
- **Read before starting**: `session-notes.md` (what was done last), `decisions.md` (why things are the way they are)
- **Update when done**: `session-notes.md` with what you did and what's next
- **Record findings**: `investigations.md`, `api-testing.md`, `api-traffic.md`, `token-notes.md`
- **Track issues**: `bugs.md`
- **Create new files** for any topic that doesn't fit existing files, and update `README.md`
- **Undocumented work is lost work** — the next agent starts from zero without notes

## Popup Module Pattern

- `popup.html` loads `popup.js` as `type="module"` — all popup files use ES module imports
- `popup/helpers.js` — shared utilities (`$`, `send`, `setDot`, `filterBranches`), `ctx` (transient state), agent a11y helpers
- `popup/render.js` — pure rendering functions (`makeField`, `renderPlotFields`, `buildCardItem`, `updateCardCount`, `setupCardsToggle`)
- `popup/events.js` — optimistic UI event handlers (`mca:branch-added`, `mca:branch-deleted`, `mca:branch-renamed`, `mca:navigate`)
- `popup/clone.js` — clone panel logic (self-contained, registers own event listeners at module load)
- `popup/delete.js` — delete confirmation dialog + API call, dispatches `mca:branch-deleted` event
- `popup/tree.js` — tree view navigation: lazy-loading hierarchy, expand/collapse, direct navigation via `mca:navigate`
- `popup/popup.js` — entry point: init, navigation, branch list, delegation, broadcasts
- **Cross-module state**: `ctx` from `helpers.js` holds `{ scenario, branches, state }` — all popup modules read/write it
- **Cross-module signals**: Use `CustomEvent("mca:...")` to avoid circular imports. clone.js → `mca:branch-added`, delete.js → `mca:branch-deleted`, tree.js → `mca:navigate`, events.js listens and coordinates
- **Test harness access**: popup.js exposes `ctx`, `openClonePanel`, `closeClonePanel`, `rebuildBranchList`, `deleteBranch` on `window`
- **New features** (settings, generation, diff) each get one new popup module file — never bolt onto popup.js
- Popup modules can import from `../background/constants.js` (both are ES modules in the same extension directory)

## Chat Module Pattern

- `popup/chat/` — agent chat panel (conversation loop, rendering, state, tools, system prompt)
- `chat.js` — conversation loop (`sendMessage`), event wiring, `initChat()`. Imports from `state.js` and `render.js`
- `state.js` — all chat state (`messages`, `busy`, `aborted`, `contextLimit`), persistence (`saveHistory`/`loadHistory`/`clearHistory`), `setBusy`/`stopAgent`, LLM timeout wrapper, friendly error mapping, a11y updates
- `render.js` — all DOM rendering: `renderMessage`, `renderToolCall`, `renderError`, `renderCompaction`, `renderThinking`/`updateThinking`/`removeThinking`, `appendAndScroll`, `renderAllMessages`
- `system.js` — builds system prompt from `getDomain()` + `getGuidance()` (supports user overrides via `prompt.js`)
- `prompt.js` — Agent/Prompt tab switcher, custom domain/guidance editing, save/defaults, persisted in `chrome.storage.local`
- `compaction.js` — LLM-powered compaction (summarizes older messages when context exceeds 75% threshold)
- `context.js` — tool result summarization + history pruning (fallback to compaction)
- `tools/` — one file per tool (11 tools) + `index.js` barrel with `TOOL_DEFS` and `executeTool`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LewdLeah/Multiple-Choice-Assistant](https://github.com/LewdLeah/Multiple-Choice-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
