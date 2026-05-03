---
trigger: always_on
description: CodMate – AGENTS Guidelines
---

CodMate – AGENTS Guidelines

Purpose
- This document tells AI/code agents how to work inside the CodMate repository (macOS desktop GUI for Codex session management).
- Scope: applies to the entire repo. Prefer macOS SwiftUI/AppKit APIs; avoid iOS‑only placements or components.

Architecture
- App type: macOS SwiftUI app (min macOS 13.5). SwiftPM-only build (no Xcode project).
- Layering (MVVM):
  - Models: pure data structures (SessionSummary, SessionEvent, DateDimension, SessionLoadScope, …)
  - Services: IO and side effects (SessionIndexer, SessionCacheStore, SessionActions, SessionTimelineLoader, LLMClient)
  - ViewModels: async orchestration, filtering, state (SessionListViewModel)
  - Views: SwiftUI views only (no business logic)

UI Rules (macOS specific)
- Use macOS SwiftUI and AppKit bridges; do NOT use iOS‑only placements such as `.navigationBarTrailing`.
- Settings uses macOS 15's new TabView API (`Tab("…", systemImage: "…")`) when available; provide a macOS 13.5/14 fallback with `tabItem` + `tag`. Container padding is unified (horizontal 16pt, top 16pt).
  - Tab content uniformly uses `SettingsTabContent` container (top-aligned, overall 8pt padding) to ensure consistent layout and spacing across pages.
- Notifications is a top-level Settings page between Terminal and Providers; sections are Common, Codex, Claude Code, and Gemini CLI. Common toggles commit message, title/comment, and copy New/Resume command notifications.
- Providers has been separated from the Codex tab into a top-level Settings page: Settings › Providers manages API key providers, OAuth providers, and Codex/Claude bindings; Settings › Codex only retains Runtime/Privacy/Raw Config (notifications live in Settings › Notifications).
  - OAuth providers (Codex/Claude/Gemini/Antigravity/Qwen) are added from the Providers “Add” menu and appear under an OAuth list section with login status and info actions.
  - CLI Proxy API status, reroute, and public access live under Providers as shared capabilities; deep diagnostics and installation details live under Settings › Advanced › CLI Proxy API.
  - Built-in providers are auto-loaded from an app-bundled `payload/providers.json` (managedByCodMate=true). This avoids hardcoding and lets users simply provide API keys; base URLs/models come pre-filled. The list merges bundled entries with `~/.codmate/providers.json` (user overrides win).
  - Schema note: use a single provider-level `envKey` (preferred) for both Codex and Claude Code connectors. Connector-level `envKey` remains tolerated for backward compatibility but is considered deprecated and will be ignored at save time to avoid duplication.
- Extensions page (aligned with Providers style):
  - Settings › Extensions replaces the old MCP Server page (icon: puzzlepiece.extension).
  - Tab 1: MCP Servers (existing list/editor/Uni‑Import UI kept as-is inside the tab); add an Import button to scan Home MCP configs into CodMate.
  - Tab 2: Skills (left list + right details split; Add menu supports folder/zip/URL; auto‑sync on changes); add an Import button to scan Home skills into CodMate.
  - Commands tab includes Add and Import buttons (Import scans Home command folders into CodMate).
  - Import sheets show a vertical list; each row has a right‑aligned strategy control (Skip/Overwrite/Rename) and a context menu “Open in…” to review source files.
  - MCP Servers tab keeps: enable toggle on left, edit on right, fixed "Add" button, Uni‑Import preview and confirmation.
  - Advanced capabilities (MCPMate download and instructions) remain as a footer/section in MCP Servers tab.
- Search: prefer a toolbar `SearchField` in macOS, not `.searchable` when exact placement (far right) matters.
- Toolbars: place refresh as the last ToolbarItem to pin it at the far right. Keep destructive actions in the detail pane, not in the main toolbar. Command+R and the refresh button also invalidate and recompute global sidebar statistics (projects/path tree and calendar day counts) to reflect new sessions immediately.
- Menu Bar (status item): keep it lightweight with status + quick actions. Show provider/model/sandbox/approval, New/Resume/Search/Open, Recent Projects/Sessions (max 5), Usage summary, Provider switch, Settings/Quit; avoid destructive actions.
- Sidebar (left):
  - Top (fixed): "All Sessions" row showing total count and selection state.
  - Middle (scrollable): path tree built from `cwd` counts. Rows are compact: default min row height 18, small control size, reduced insets. Single-click selects/expands; double-click applies filter (enter the directory).
  - Projects mode mirrors the compact list style; Cmd-click toggles multi-selection so users can filter sessions by several projects simultaneously (descendants remain included).
  - Bottom (fixed): calendar month view (240pt height) with per-day counts (created/last-updated switch). Always pinned to the bottom with 8pt spacing above. Supports multi-select via Command-click to toggle multiple days; plain click selects a single day (click the same day to clear).
  - Only the middle path tree scrolls; top "All Sessions" and bottom calendar remain fixed.
  - Sidebar width: min 220pt, max 25% of window width, ideal 260pt.
- Content (middle):
  - Default scope loads “today” only for speed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loocor/codmate](https://github.com/loocor/codmate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
