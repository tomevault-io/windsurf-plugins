---
trigger: always_on
description: - Do not run `bun run build` unless the user explicitly asks for production packaging or build verification.
---

# PulsarAI Agent Guide

## Hard Rules

- Do not run `bun run build` unless the user explicitly asks for production packaging or build verification.
- Do not run full test suites unless the user explicitly asks.
- When reading files with PowerShell `Get-Content`, always pass `-Encoding UTF8`.
- Renderer feature code imports native capability only from `@/host`; do not import Electron, Tauri, or a Tauri plugin outside `host/`.
- `host/index.ts` is the stable cross-platform facade. Its selected target implementation is `host/desktop-electron` or `host/mobile-tauri`; keep platform-exclusive APIs in `host.desktop` or `host.mobile`, not as no-op compatibility APIs.
- Electron owns frameless main-window geometry, drag regions, tray/close lifecycle, subwindows, desktop environment checks, and Playwright. Keep its preload bridge narrow, with `contextIsolation` and sandbox enabled; renderer code must never receive `ipcRenderer`, Node, or arbitrary command execution.
- Mobile Tauri owns Android battery controls, M3 navigation-bar control, and system speech recognition. It must not register desktop tray, desktop window lifecycle, multi-window, Playwright, or desktop-only settings/actions.
- `host/mobile-tauri/tauri.conf.json` is the Tauri project marker. Use `bun run mobile:*`; use `bun run desktop:*` for Electron. Do not restore a top-level `src-tauri/` host or ambiguous `electron`/`tauri` scripts.
- Prefer scoped implementation, current feature seams, and small verifiable steps.
- When implementing or updating UI components, prioritize animated components from `@/components/fluid` (e.g., `Button`, `Badge`, `Accordion`, `Card`, `Dialog`, `DropdownMenu`, `RadioGroup`, `Select`, `Slider`, `Switch`, `Tabs`, `Tooltip`, `ChatMessage`, `ThinkingSteps`, `ThinkingIndicator`, `AskUserQuestions`, `ColorPicker`, etc.) whenever a counterpart exists. Fluid components provide identical interfaces to shadcn-vue components while delivering spring physics and fluid motion styling. Do not delete original components under `src/components/ui/`; retain them as base fallbacks.
- Use shadcn-vue `ScrollArea` by default for application-owned scrolling regions such as settings navigation/content, plugin trees, and long resource panels. Its shared wrapper must mount visible vertical and horizontal `ScrollBar` components for overflowing content; keep native overflow only where a specialized editor or virtualizer owns scrolling.
- When implementing or changing components, explicitly consider narrow-window and mobile-platform behavior. Use the shared responsive state instead of scattering viewport or platform checks, keep touch targets usable, and provide an explicit layout fallback below 768px.
- Commands that expose feature behavior to search or hotkeys should keep their implementation in the owning feature's `actions.ts`; UI may register and route commands but should not own domain behavior.
- Plugin is owned by `src/features/Plugin`, but World is the public resource model. A local role source is persisted at `resource_worlds:local:<localPluginId>` and `/definition.package.json` stores its enabled global Plugin source folder names in `globalPlugins: string[]`. Replay routes each Pulse to its local or `/global/<source-folder>/` source and replays that source independently; only then are the currently enabled sources merged in array order. Every source root contains a `localSlot/` definition tree; a file stores its local-slot path in `file.slot`, and a local slot contributes through its optional global-contract `parent` path. `@/` remains source-local and `@pluginId/` does not exist. Do not restore flat files/empty-folder persistence, World configs, adoption maps, folder switches, or a second runtime pipeline.

- Pulse is the only replay primitive: persist one logical Pulse per user action, addressed solely by stable node IDs; filenames are display snapshots. Local Plugin Worlds use `resource_worlds:local:<localPluginId>`. `Conversation.localPluginId` owns the relationship; roles are computed from `/self/definition.package.json`, never stored in a separate table. Local preferences (pin/order/sync/category) are environment state keyed by `localPluginId`.
- The Plugin file editor keeps resource properties (slot, condition and priority) in the file surface. Slot contract properties live on folders below `/self/slot/`; their folder action filter must forbid adding resources. Render the file surface inside a centered, bounded, Obsidian-like background card with a full-width fallback below 768px.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NullStarrySky/PulsarAI](https://github.com/NullStarrySky/PulsarAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
