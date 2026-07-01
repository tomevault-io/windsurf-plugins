---
trigger: always_on
description: Guidance for Claude Code when working in this repository — an Obsidian plugin providing LLM chat interfaces for OpenAI, Anthropic Claude, Google Gemini, Mistral, and local Ollama / LM Studio / GPT4All.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository — an Obsidian plugin providing LLM chat interfaces for OpenAI, Anthropic Claude, Google Gemini, Mistral, and local Ollama / LM Studio / GPT4All.

## Build Commands

```bash
npm run dev      # watch mode (esbuild)
npm run build    # production build (tsc type-check + esbuild bundle)
npm run lint     # eslint src  (lint:fix to autofix) — prefer-const/eqeqeq/no-console are warnings
npm run version  # bump manifest.json and versions.json
npm run test:e2e # E2E suite — real sandboxed Obsidian via wdio-obsidian-service (see test/README.md)
```

Output bundles to `main.js` in the root. esbuild targets CommonJS/ES2018; `obsidian`, `electron`, `@codemirror/*`, and Node builtins are external; SVGs load inline. TypeScript uses strict null checks, baseUrl `src`. The esbuild banner defines an `import.meta.url` shim (`__import_meta_url`) — `@anthropic-ai/claude-agent-sdk` calls `createRequire(import.meta.url)` at module scope, which would otherwise throw on load in CJS output. Don't remove the `define`/banner pair.

E2E conventions live in `test/README.md` — notably: never point `wdio.conf.mts` `plugins:` at `"."` (the service copies `data.json` — real API keys — into test vaults; always stage via `scripts/stage-plugin.mjs`), and never hardcode the plugin id in specs (read `PLUGIN_ID` from `test/specs/helpers.ts`).

## Architecture Overview

### Entry point

`src/main.ts` — `LLMPlugin` class: initializes platform abstractions (Desktop/Mobile in `src/services/`), loads settings (`loadData`/`saveData`), registers commands/views, initializes MessageStore, History, and FAB.

### View architecture (four UIs, shared components)

- **Modal** — `src/Plugin/Modal/ChatModal2.ts`
- **Widget** (tab view) — `src/Plugin/Widget/Widget.ts`
- **FAB** — `src/Plugin/FAB/FAB.ts`
- **StatusBarButton** — `src/Plugin/StatusBar/StatusBarButton.ts` — "Ask AI" popover; uses `viewType: "floating-action-button"` and shares `fabSettings` with the FAB. The popover is built once on `generate()`, so call `chatContainer.syncModelDropdown()` whenever it is shown.

All compose shared components from `src/Plugin/Components/`: `Header.ts` (tab nav), `ChatContainer.ts` (messages, input, API calls), `HistoryContainer.ts`, `SettingsContainer.ts`.

### Multiple chat widget tabs

Multiple `WidgetView` instances can be open at once, each owning its own `ChatContainer` + `MessageStore` + chat file path (fully isolated conversations).

- `new-chat-widget` command always creates a fresh tab; `open-LLM-widget-tab` and the ribbon icon use focus-or-open-one-tab.
- `LLMPlugin.lastFocusedWidgetLeaf` is updated on `active-leaf-change`; `openChatFileInWidget()` / `activateTab()` prefer it so "open chat file" lands in the last-used widget.
- `ChatsSidebar.onOpenFile` callback: `WidgetView.onOpen()` sets it to `this.loadChatFile` so sidebar rows load into *that* widget. The standalone `ChatsView` still routes via `plugin.openChatFileInWidget()`.
- **Known limitation:** all widget tabs share `plugin.settings.widgetSettings`; model changes don't push reactively to other tabs' dropdowns. v2: per-view `ViewSettings` clone.

### State management

- **MessageStore** (`src/Plugin/Components/MessageStore.ts`) — pub/sub message state; synchronizes views. `setMessages` stores a shallow copy (`[...messages]`) so later `addMessage` pushes can't mutate the caller's array (notably legacy `promptHistory[n].messages`).
- **HistoryHandler** (`src/History/HistoryHandler.ts`) — legacy in-settings history; superseded by file-based `ChatHistory` when `chatHistoryEnabled: true` (the default).

### Message flow

Input → `handleGenerateClick()` → message added to MessageStore (notifies subscribers) → provider API call → streaming UI updates → saved to History.

**Context injection order** (top → bottom): recalled memories → project instructions → assistant system prompt → skill instructions → vault/file context.

#### Render generation guard (`renderGeneration`) — do not remove

`updateMessages` re-renders the full list via `resetChat()` + async `generateIMLikeMessages()`. A stale async render can append into a container already cleared by a newer render (duplicated/out-of-order messages). `ChatContainer.renderGeneration` counter prevents this: `updateMessages` increments it and passes it down; the render function bails whenever `gen !== this.renderGeneration`. The race only shows up on rapid successive sends — easy to miss in manual testing. Never remove this guard or make `generateIMLikeMessages` synchronous without understanding it.

### Stop button / generation abort

`ChatContainer._abortController: AbortController | null` is non-null while a generation is in-flight.

- `enterStopMode(sendButton)` creates the controller and swaps send → red `square` stop icon (`.llm-stop-mode`); `exitStopMode(sendButton)` clears and restores. Called by `handleGenerateClick` at every entry/exit point (including `/remember` early return and pure-prompt skill path).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eharris128/Obsidian-LLM-Plugin](https://github.com/eharris128/Obsidian-LLM-Plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
