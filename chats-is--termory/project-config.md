---
trigger: always_on
description: This file is for Claude Code working in this repository. Keep it limited to current codebase facts, implementation rules, and verification steps.
---

# CLAUDE.md

## Scope

This file is for Claude Code working in this repository. Keep it limited to current codebase facts, implementation rules, and verification steps.

## Current App

Tauri v2 desktop app for browsing local history records from supported AI tool clients, plus their on-disk memory and skill files.

Supported sources in the current code:

- Codex
- Claude Code
- Gemini CLI
- OpenCode
- Grok Build (xAI — sessions + memory (`<grok-home>/memory/`) + skills + provider switching + session/project delete + official-quota (weekly/monthly credit window); no migrate)

Top-level UI shell — six activity-rail routes (⌘1..6 in order):

1. **Providers** — third-party API platform switcher (per CLI). Default landing route. One tab per managed app: Claude Code / Codex / Gemini / OpenCode / Grok Build (the five CLIs) **plus Claude Desktop** — the GUI app, managed via its 3P gateway profile (direct mode only); its tab always shows (like the other apps), and install/platform gates activation + the InstallGuide + gateway binding, NOT tab visibility. See "Claude Desktop (3P gateway profile)" under Providers.
2. **Records** — chat / transcript history. Three internal panes:
   - *Sessions* — per-tool transcripts
   - *Memories* — on-disk memory files (CLAUDE.md, AGENTS.md, GEMINI.md, `~/.codex/memories/`, `~/.claude/projects/<slug>/memory/`, etc.)
   - *Skills* — `SKILL.md` files under each tool's skills directory plus the cross-tool `.agents/skills/` location
3. **Favorites** — saved per-message snapshots
4. **Search** — substring search + Cmd-K palette
5. **Stats** — KPI cards + all-history calendar heatmap + a Tokens chart (Type / Model breakdown)
6. **Settings** — Appearance / Storage / Search history / Keyboard shortcuts / About

Plus a **system-level macOS menu-bar tray** (outside the in-app rail) for quick provider switching — see "Menu-bar tray" below.

Current alignment target: data acquisition and message preview formatting should follow the official tools. UI layout, source filters, project grouping, search, stats, cross-source sorting, and the Memory/Skills views are app behavior and do not need official UI parity.

## Tech Stack

- Desktop framework: Tauri v2
- Frontend: React 18, TypeScript, Vite
- UI / styling: Tailwind CSS v4 + shadcn/ui (Radix primitives); charts via `recharts`; virtualized lists via `@tanstack/react-virtual`; theming via `next-themes`; toasts via `sonner`
- Backend: Rust 2021
- Database access: `rusqlite` with bundled SQLite
- JSON parsing: `serde`, `serde_json`
- Filesystem scanning: `walkdir`, `dirs`
- Time handling: `chrono`
- Icons: `lucide-react` plus inline SVG brand icons

## Code Map

- Frontend: `src/main.tsx`
- Styles: `src/styles.css`
- i18n (en / 简体 / 繁體): `src/i18n/index.tsx` (`<I18nProvider>` + `useT()`) + `src/i18n/locales/{en,zh-Hans,zh-Hant}.ts` — see "Internationalization (i18n)" under UI conventions
- Frontend local-store wrapper: `src/config.ts` (routes `getConfig`/`setConfig` to the three backing files via IPC — `providers` → providers.json, `favorites` → favorites.json, everything else → config.json)
- Tauri IPC commands: `src-tauri/src/lib.rs`
- Session/Memory/Skill scanning, parsing, and formatting: `src-tauri/src/sessions.rs`
- Provider switching (activate / deactivate / reverse-derive / test / fetch-models): `src-tauri/src/providers.rs`
- Claude **Desktop** 3P gateway-profile materialization (direct mode only — apply / restore-official / reverse-derive / installed-detect): `src-tauri/src/claude_desktop.rs` (inline tests at the bottom) — see "Claude Desktop (3P gateway profile)" under Providers
- Codex "follow sessions" on provider switch (re-tag rollout-file + threads `model_provider` so `codex resume` keeps finding a project's history): `src-tauri/src/codex_follow.rs` (+ frontend `src/components/providers/CodexFollowDialog.tsx`) — see "Codex 'follow sessions' on provider switch"
- Official-account subscription quota (read CLI OAuth creds → query official usage endpoint): `src-tauri/src/quota.rs` (inline tests at the bottom)
- Codex multi-account management (save / switch / delete login snapshots, token refresh, needsRelogin flag, cancel login via `CodexLoginCancel` managed state): `src-tauri/src/accounts.rs` (inline tests at the bottom); UI: `src/components/providers/OfficialAccountsSection.tsx`
- macOS menu-bar tray (build menu / rebuild / click handler): `src-tauri/src/tray.rs`; icon assets `src-tauri/icons/tray-icon.png` (36×36 template) + `tray-template.svg` (vector source)
- Terminal launching (detect installed terminals / open-and-resume): `src-tauri/src/terminal.rs` (per-OS `#[cfg]`; chosen via Settings → Terminal, `terminal` config key)
- Local KV store (config.json + providers.json + favorites.json under `~/.termory/`, chmod 0600): `src-tauri/src/config.rs`
- Filesystem watcher (static CLI-data dirs + dynamic recursive session-cwd watches → debounced re-scan → `termory:sources-changed`): `src-tauri/src/watcher.rs`
- Stats aggregations (pure, window-accurate): `src/lib/stats-utils.ts` (+ `stats-utils.test.ts`)
- Stats UI: `src/components/stats/{StatsPage,OverviewSection,TokensChart}.tsx` + `heatmap-shared.ts` (+ `StatsPage.test.tsx`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chats-is/termory](https://github.com/chats-is/termory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
