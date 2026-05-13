---
trigger: always_on
description: A vendor-free, open-source desktop app that wraps OpenClaw. No login, no credits, no vendor lock-in — just a double-click installer that manages OpenClaw's setup and provides a polished GUI.
---

# MaxAuto

## What is MaxAuto

A vendor-free, open-source desktop app that wraps OpenClaw. No login, no credits, no vendor lock-in — just a double-click installer that manages OpenClaw's setup and provides a polished GUI.

## Tech Stack

- **Frontend:** React 19 + TypeScript, Tailwind CSS 3.4 + shadcn/ui (Radix primitives), Zustand 5 (state), Vite 6, i18next (i18n)
- **Backend:** Tauri v2 (Rust), tokio, reqwest, serde
- **UI Components:** shadcn/ui with HSL CSS variable theming (13 primitives: Button, Input, Textarea, Badge, Label, Dialog, Card, Separator, Switch, Tabs, ScrollArea, Collapsible, Tooltip)
- **Communication:** WebSocket to OpenClaw gateway (`ws://127.0.0.1:51789`), Tauri IPC for Rust commands
- **Platforms:** Windows (.msi) + macOS (.dmg)
- **Package manager:** pnpm 10

## Project Structure

```
├── src/                          # React/TypeScript frontend
│   ├── main.tsx                  # React entry point + theme initialization
│   ├── App.tsx                   # Root (SetupPage or AppShell)
│   ├── env.d.ts                  # Vite/Tauri type declarations
│   ├── global.css                # Tailwind + shadcn HSL CSS variables + slider styles
│   ├── api/
│   │   ├── device-identity.ts    # Ed25519 device keypair (generate, persist, sign)
│   │   ├── gateway-client.ts     # WebSocket client for OpenClaw gateway
│   │   ├── tauri-commands.ts     # Typed Tauri invoke() wrappers (gateway, system, setup, config, pairing, docker, shell, winget)
│   │   ├── config-helpers.ts     # Config patching & gateway reconnection utilities
│   │   └── telegram-accounts.ts  # Telegram multi-account config management & migration
│   ├── lib/
│   │   ├── utils.ts              # cn() utility (clsx + tailwind-merge)
│   │   └── theme-utils.ts        # HSL color math, theme derivation, 10 built-in presets
│   ├── components/
│   │   ├── ui/                   # shadcn/ui primitives (13 components)
│   │   │   ├── button.tsx, input.tsx, textarea.tsx, badge.tsx, label.tsx
│   │   │   ├── dialog.tsx, card.tsx, separator.tsx
│   │   │   ├── switch.tsx, tabs.tsx, scroll-area.tsx, collapsible.tsx, tooltip.tsx
│   │   ├── layout/               # AppShell (with close-to-tray dialog), TitleBar
│   │   ├── chat/                 # ChatPanel, ChatInput, Sidebar, SidebarTabs,
│   │   │                         # AgentCard, AgentList, CreateAgentDialog, EditAgentDialog
│   │   ├── settings/             # AppearanceSection, ModelsAndApiSection, AddModelDialog,
│   │   │                         # QuickConfigModal, GeneralSection, IMChannelsSection,
│   │   │                         # McpSection (local + remote servers), SkillsSection,
│   │   │                         # WorkspaceSection, AboutSection,
│   │   │                         # BotCard, BotCardList, AddBotDialog, RemoveBotDialog,
│   │   │                         # TagInput, skills-utils.ts
│   │   └── common/               # GatewayStatus (debug log dialog), UpdateBanner
│   ├── i18n/                     # i18next setup + locale files
│   │   ├── index.ts              # i18next init with LanguageDetector
│   │   └── locales/
│   │       ├── en/translation.json
│   │       └── zh-TW/translation.json
│   ├── pages/
│   │   ├── SetupPage.tsx         # First-run setup flow (native or Docker mode) + debug log
│   │   └── SettingsPage.tsx      # Settings navigation (10 sections) + debug button in sidebar
│   └── stores/
│       ├── app-store.ts          # Global app state (setup, gateway, page, port, installMode)
│       ├── appearance-store.ts   # Theme state (preset, colors, contrast) + localStorage
│       ├── chat-store.ts         # Chat state + agent CRUD + streaming + tool activity
│       ├── settings-store.ts     # Settings, models, provider defaults, config
│       └── update-store.ts       # App auto-update checking & installation
├── src-tauri/                    # Rust backend
│   ├── Cargo.toml
│   ├── build.rs                  # Tauri build script
│   ├── tauri.conf.json           # Window 1200×800, decorated, com.openclaw.maxauto
│   ├── capabilities/default.json # Permission grants (shell, updater, process, dialog, fs, window hide/show/close)
│   └── src/
│       ├── main.rs / lib.rs      # Tauri app builder + plugin setup (17 commands) + RunEvent::Exit gateway cleanup
│       ├── commands/
│       │   ├── mod.rs             # Command module re-exports
│       │   ├── gateway.rs         # start/stop/status gateway, token generation, port cleanup (CREATE_NO_WINDOW on Windows)
│       │   ├── system.rs          # check Node.js/Git/OpenClaw, platform info
│       │   ├── setup.rs           # install Node.js 24, Git, OpenClaw, winget packages
│       │   ├── config.rs          # read/write openclaw.json, read provider API key
│       │   ├── pairing.rs         # Telegram pairing (list/approve/reject, 1hr TTL)
│       │   └── docker.rs          # Docker container lifecycle (check/pull/start/stop/status)
│       ├── state/
│       │   ├── mod.rs
│       │   └── gateway_process.rs # Mutex-wrapped child process + port holder
│       └── tray/
│           ├── mod.rs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Maxch3306/openclaw-maxauto](https://github.com/Maxch3306/openclaw-maxauto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
