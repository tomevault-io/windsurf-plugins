---
trigger: always_on
description: This document describes Lumina Terminal's architecture, design principles, and
---

# AGENTS.md

This document describes Lumina Terminal's architecture, design principles, and
the rules any AI (or human) contributor must follow so the codebase stays
high-cohesion / low-coupling and does not regress into duplication.

> Read this **before** making changes. If a change would violate a rule below,
> extract or refactor first rather than adding another copy.
> This project does not rely on OpenSepc. Before doing any large changes, AI
> should enter the plan mode of the harness tool instead of write a spec document.

---

## 1. Tech Stack

| Layer | Technology |
|-------|-----------|
| Shell / backend | Rust + Tauri v2 |
| PTY | `portable_pty` |
| Frontend | React 19 + TypeScript (strict) |
| Terminal renderer | xterm.js v6 (+ webgl, fit, web-links, image addons) |
| UI components | HeroUI (`@heroui/react`) |
| Styling | Tailwind CSS v4 |
| Build | Vite 7, `pnpm` |
| i18n | JSON files in `translations/` |

The backend (`src-tauri/`) is intentionally thin: it spawns/kills PTYs,
streams output via Tauri events, and exposes a few filesystem helpers. All UI
logic, state, and derivation live in the frontend.

`@xterm/xterm` stays on stable 6.0.0 plus a local backport patch
(`patches/@xterm__xterm@6.0.0.patch`, declared in `pnpm-workspace.yaml`) that
vendors two upstream IME fixes the WebKitGTK duplicate-input fix depends on
(xterm.js #5439 + #5698). See `patches/README.md`; drop the patch when the
next stable xterm release containing both ships.

---

## 2. Source Map

### Frontend (`src/`)

```
src/
├── App.tsx                  # Root: composes chrome (TabBar/TitleBar/Term) + non-terminal key dispatch.
│                            #   Sidebar visibility lives in useSidebarVisibility (explicit toggle →
│                            #   one-shot CLI --sidebar → the showTabBar setting; setTabBarVisible
│                            #   is the single write path); theme-mode translation in lib/themeMode.ts.
│                            #   Tab lifecycle/state live in useTerminalManager; geometry in useWindowGeometry.
│                            #   Non-first-screen pages (Settings/About/Welcome) are React.lazy so
│                            #   Settings' subtree + the markdown renderer stay out of the startup chunk.
├── main.tsx                 # ReactDOM entry; wraps App in GlobalConfigProvider
├── constants.ts             # Default config, default bindings, tab-id sentinels
├── types/
│   ├── config.ts            # GlobalConfig, Binding, Actions, WithKeys, CommandIconRule +
│                            #   Languages (the UI-language union lives here so GlobalConfig can
│                            #   reference it without types/ reaching into hooks/i18n.tsx)
│   ├── cli.ts               # CliArgs — parsed launch flags (mirrors src-tauri/src/cli.rs CliArgs)
│   └── terminal.ts          # TerminalProfile (+ keepAfterExit: "exit"|"freeze"|"shell" — what
│                            #   happens after startupCommand finishes) + ProfileLauncher (the
│                            #   wrap-as-app section: title/workingDirectory/sidebar/icon; presence
│                            #   enables it), TerminalRenderOptions, SSHConfig
│
├── lib/                     # Pure, framework-agnostic logic (NO React)
│   ├── platform.ts          # isMacOS() / isLinux()
│   ├── configFile.ts        # Config-file IO domain: config.toml path + openConfigFile +
│   │                        #   readConfigDocument (toml preferred; legacy config.json parsed
│   │                        #   and migrated, then renamed config.json.bak) + writeConfigDocument
│   ├── configFormat.ts      # Pure config format layer: TOML parse + renderConfigToml (patches onto
│   │                        #   the existing document — key order, layout and comments survive
│   │                        #   settings rewrites; nullish pruning) + legacy JSON unwrap — zero
│   │                        #   internal imports so node --test loads it directly
│   ├── color.ts             # isColorDark, foregroundFor, adjustColor, visibleRed
│   ├── glass.ts             # glassSurface / glassBorder / elevationShadow / windowOutline —
│   │                        #   backdrop-filter material + Wayland/WebKitGTK opaque fallback
│   │                        #   (single source for the glass look; windowOutline is the Linux
│   │                        #   1px window hairline for DEs without compositor shadows)
│   ├── motion.ts            # framer-motion variants/transitions presets (one spring curve for all chrome)
│   ├── ssh.ts               # formatSshAddress / formatSshEntry
│   ├── term.ts              # parseProfile, parseProfileTheme, parseProfilePadding
│   ├── terminalApi.ts       # invoke wrappers: writeToTerminal, resizeTerminal, ... startTerminal also
│   │                        #   carries the spawn-time enableShellCompletions flag (hooks are baked into the
│   │                        #   shell's init files — toggling affects new terminals only, like webgl)
│   ├── mcpApi.ts            # startMcpServer/stopMcpServer invoke wrappers (log-on-reject) — the
│   │                        #   read-only MCP server domain API (sibling to terminalApi.ts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iewnfod/lumina-terminal](https://github.com/iewnfod/lumina-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
