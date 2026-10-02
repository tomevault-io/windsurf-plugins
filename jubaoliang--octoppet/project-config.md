---
trigger: always_on
description: Navigation guide for AI coding agents working in **octop-pet**.
---

# AGENTS.md

Navigation guide for AI coding agents working in **octop-pet**.

## 1. Collaboration principles

> Favor caution over speed; trivial tasks may relax these rules.

### Think before writing

- State assumptions up front; ask when unsure — do not guess.
- When multiple interpretations exist, list them and let the user choose.
- Suggest simpler approaches when they exist; stop when blocked and name what is unclear.

### Simplicity first

- Write the minimum code that solves the problem; no unrequested features or abstractions.
- Do not add defensive error handling for scenarios that cannot realistically happen.
- Trim the diff when it grows unnecessarily large.

### Surgical edits

- Touch only lines directly related to the task; do not opportunistically clean up nearby code.
- Do not refactor working code or unify style just because it differs from yours.
- Remove orphan imports, variables, and functions **you** introduced.

### Verifiable outcomes

- Turn tasks into verifiable goals (what to test, which command proves success).
- Before saying "done", provide verification evidence; the default ship bar is **`make all` green**.

## 2. What this is

**Octop Pet** — desktop companion for a remote [Octop](https://github.com/TencentCloud/Octop) server.

- **Tauri 2** (Rust): tray, multi-window, config, keyring, global shortcuts
- **React 19 + TypeScript + Vite**: pet, chat, and settings surfaces in one SPA
- **Runtime dependency:** a reachable Octop HTTP/WebSocket API — **no Octop source tree at runtime**

Design specs:

- [Desktop pet design](docs/superpowers/specs/2026-08-04-octop-desktop-pet-design.md)
- [Pet window UX](docs/superpowers/specs/2026-08-04-pet-window-ux-design.md)

## 3. Tech stack

| Layer          | Technology                                                                                                        |
| -------------- | ----------------------------------------------------------------------------------------------------------------- |
| Desktop        | Tauri 2, Rust stable                                                                                              |
| UI             | React 19, TypeScript, Vite 7                                                                                      |
| HTTP           | `@tauri-apps/plugin-http` (Tauri) / `fetch` (Vitest)                                                              |
| WebSocket      | Browser `WebSocket` in chat window                                                                                |
| Config         | JSON file via Rust `config_cmd` + `@tauri-apps/plugin-store`                                                      |
| Secrets        | Rust `secrets_cmd` → OS keyring                                                                                   |
| Markdown       | `react-markdown` + `remark-gfm`                                                                                   |
| Frontend tests | Vitest 3, Testing Library, jsdom                                                                                  |
| Rust tests     | `cargo test` in `src-tauri/tests/`                                                                                |
| CI             | GitHub Actions — Ubuntu (`ci.yml`: test + typecheck + cargo) + tag release (`release.yml`: macOS ×2 + Windows ×2) |

## 4. Project layout

```
src/
  main.tsx                 routes by Tauri window label (pet | chat | settings)
  App.css                  global styles; window-specific via data-window-label
  windows/
    PetWindow.tsx          floating mascot, drag, context menu
    ChatWindow.tsx         chat state machine, streaming, init/retry
    SettingsWindow.tsx     credentials, hotkeys, mascot picker
  components/
    Composer.tsx           input, agent/model/connector pickers, attachments
    MessageList.tsx        history + assistant actions (copy/retry/speak)
    AgentSelect.tsx        agent dropdown
    AssistantMarkdown.tsx  streaming-safe Markdown
    QueuedMessages.tsx     pending queue while streaming
    ShortcutRecorder.tsx   global shortcut capture
    …
  lib/
    octopHttp.ts           login, agents, threads, history, uploads
    octopTypes.ts          Octop DTOs + model/attachment helpers
    chatStream.ts          WS URL, payloads, stream chunk reducer
    chatHelpers.ts         chat constants, history mapping, error text
    configLogic.ts         thread map helpers, base URL normalization
    tauriApi.ts            invoke + event wrappers
    tauriWindowApi.ts      window sizing, hide, drag, resize
    messageQueue.ts        queue data structure
    streamStatus.ts        tool/status labels during stream
    stabilizeStreamingMarkdown.ts
    shortcutFormat.ts
    types.ts               shared TS types (mirror Rust config field names in camelCase)
  hooks/
    useChatController.ts   chat init, auth, streaming, queue, layout
    useWindowChrome.ts     auto-fit window height, Escape to hide
  styles/
    base.css, pet.css, settings.css, chat.css
src-tauri/src/
  lib.rs                   Tauri builder, plugins, command registration
  main.rs                  entry
  tray.rs                  system tray menu + global shortcuts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jubaoliang/OctopPet](https://github.com/jubaoliang/OctopPet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
