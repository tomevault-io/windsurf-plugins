---
trigger: always_on
description: **Updated:** 2026-06-01 Asia/Seoul
---

# PROJECT KNOWLEDGE BASE

**Updated:** 2026-06-01 Asia/Seoul
**Commit:** ec74a99 | **Branch:** feat/dashboard-open-in-new-window

## OVERVIEW

VS Code extension that embeds ULW Terminal in the secondary sidebar. Extension host code manages PTY/native terminals, tmux/zellij sessions, HTTP prompt delivery, and VS Code state; webview code renders xterm.js and the terminal manager dashboard.

## STRUCTURE

```
./
├── src/
│   ├── extension.ts          # VS Code activate/deactivate entry
│   ├── types.ts              # host-webview DTOs, backend/tool config, tmux dashboard DTOs
│   ├── core/                 # activation, service wiring, command groups
│   ├── providers/            # VS Code webview providers and host-side message routing
│   ├── services/             # instance/session/backend state, tmux/zellij/API/context
│   ├── terminals/            # node-pty lifecycle wrapper
│   ├── webview/              # browser-only xterm/dashboard UI
│   ├── test/                 # e2e suite + manual vscode/node-pty mocks
│   └── __tests__/            # Vitest setup and cross-module regression tests
├── dist/                     # webpack output: extension.js, webview.js, dashboard.js
├── out/                      # tsc e2e output; generated
├── coverage/                 # Vitest coverage; generated
├── resources/                # activity bar icons
├── docs/, memories/          # notes and planning artifacts
└── package.json              # extension manifest, commands, config keys, scripts
```

## ARCHITECTURE

```
package.json main -> dist/extension.js
src/extension.ts -> ExtensionLifecycle.activate()
  ├── creates stateful services by manual DI
  ├── registers TerminalProvider + TerminalDashboardProvider
  ├── registers command groups from core/commands/
  ├── consumes SessionWindowHandoffService payloads on activation
  └── registers CodeActionProvider
```

Host-webview messages are discriminated unions in `src/types.ts`: `WebviewMessage`, `HostMessage`, `TmuxDashboardActionMessage`, and dashboard DTOs. New message shapes must update this file and the matching router/tests.

## WHERE TO LOOK

| Task                      | Location                                                                                   | Notes                                                        |
| ------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------ |
| Activation / DI           | `src/core/ExtensionLifecycle.ts`                                                           | service creation, provider registration, handoff consumption |
| Command wiring            | `src/core/commands/`                                                                       | terminal, tmux session, tmux pane, dashboard command modules |
| Sidebar terminal shell    | `src/providers/TerminalProvider.ts`                                                        | webview lifecycle, HTML/CSP, pane/editor panel bridge        |
| Host message handlers     | `src/providers/MessageRouter.ts`                                                           | webview input, clipboard, drops, VS Code terminal bridge     |
| Runtime/session switching | `src/providers/SessionRuntime.ts`                                                          | native/tmux/zellij startup, pane state, HTTP readiness       |
| Terminal dashboard        | `src/providers/TerminalDashboardProvider.ts` + `src/webview/dashboard-manager.tsx`         | provider shell + Preact dashboard bundle                     |
| Instance state            | `src/services/InstanceStore.ts`                                                            | in-memory EventEmitter hub; do not duplicate state           |
| Instance lifecycle        | `src/services/InstanceController.ts`, `InstanceDiscoveryService.ts`, `InstanceRegistry.ts` | spawn/connect/discover/persist                               |
| Tmux/zellij CLI           | `src/services/TmuxSessionManager.ts`, `ZellijSessionManager.ts`                            | backend-specific process/session wrappers                    |
| AI tool behavior          | `src/services/aiTools/` + `src/types.ts`                                                   | OpenCode/Claude/Codex launch and file-reference formatting   |
| Browser terminal UI       | `src/webview/main.ts`, `src/webview/terminal/`                                             | xterm.js setup, keyboard, resize, toolbar HTML               |
| Tests and mocks           | `src/test/`, `src/**/*.test.ts`                                                            | Vitest, manual mocks, e2e suite                              |

## CODE MAP

| Symbol                   | Type     | Location                                         | Role                                          |
| ------------------------ | -------- | ------------------------------------------------ | --------------------------------------------- |
| `activate`               | function | `src/extension.ts`                               | VS Code entry                                 |
| `ExtensionLifecycle`     | class    | `src/core/ExtensionLifecycle.ts`                 | activation, DI, command/provider registration |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [islee23520/ulwcode](https://github.com/islee23520/ulwcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
