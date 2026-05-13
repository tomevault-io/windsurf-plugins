---
trigger: always_on
description: Electron + React + TypeScript terminal application with AI-powered features.
---

# Tron — AI-Powered Terminal

Electron + React + TypeScript terminal application with AI-powered features.

## Architecture

```
src/
  constants/          # String constants (IPC channels, localStorage keys)
  types/index.ts      # Single source of truth for all shared types
  services/
    ai/               # AIService class — multi-provider streaming, agent loop (runAgent)
      index.ts        # Provider handling, streaming, tool dispatch, safety mechanisms
      agent.md        # Compact system prompt appended to agent instructions (keep concise — sent every call)
    mode.ts           # TronMode singleton (local/gateway/demo) and predicates
    ws-bridge.ts      # WebSocket IPC bridge for web mode, modeReady promise
    demo-bridge.ts    # Mock IPC handlers for demo mode (no server)
  hooks/              # Custom React hooks (useAgentRunner, useModels, useHotkey)
  utils/
    platform.ts           # Cross-platform path utilities (isWindows, extractFilename, abbreviateHome, etc.)
    commandClassifier.ts  # Command classification (case-insensitive), smartQuotePaths(), isInteractiveCommand()
    terminalState.ts      # Terminal state classifier (idle/busy/server/input_needed), scaffold duplicate check, autoCd
    contextCleaner.ts     # ANSI stripping, output collapsing, truncation
    dangerousCommand.ts   # Dangerous command detection (rm -rf, sudo, git force-push, etc.)
    theme.ts              # Theme token registry, themeClass() helper
    motion.ts             # Shared framer-motion variants
  contexts/           # React contexts (Layout, Theme, History, Agent)
  components/
    layout/           # TabBar, SplitPane (recursive), TerminalPane, ContextBar, CloseConfirmModal, NotificationOverlay, SavedTabsModal, EmptyState
    ui/               # Modal, FolderPickerModal, FeatureIcon, SpotlightOverlay
  features/
    terminal/         # Terminal.tsx (xterm.js), SmartInput.tsx
    agent/            # AgentOverlay.tsx, TokenHeatBar.tsx
    settings/         # SettingsPane.tsx (per-provider config caching)
    ssh/              # SSHConnectModal.tsx, SSHStatusBadge.tsx
    onboarding/       # OnboardingWizard.tsx (theme + AI setup)
  App.tsx             # Root: providers + TabBar + workspace + close confirm modal
  main.tsx            # React entry point
  index.css           # Tailwind + scrollbar styles

electron/
  main.ts             # Window creation, app lifecycle, close interception
  ipc/
    terminal.ts       # PTY session management, exec, execInTerminal (sentinel-based), completions, history, /log handler, session file persistence
    ssh.ts            # SSH session adapter (PtyLike interface over ssh2), profile persistence, IPC handlers
    config.ts         # Config/session IPC handlers (readSessions, writeSessions)
    system.ts         # Folder picker, shell:openPath, shell:openExternal
    ai.ts             # AI connection test — cloud providers validate config only (no API call), local providers ping endpoint
    web.ts            # Web search (Brave→DuckDuckGo→Startpage fallback) and web fetch IPC handlers
  preload.ts          # Context bridge with channel allowlist

server/               # Web mode (Express + WebSocket, no Electron)
  index.ts            # HTTP server + WS bridge + mode routing (local/gateway)
  handlers/
    terminal.ts       # Terminal session handlers (mirrors electron/ipc/terminal.ts)
    ssh.ts            # SSH session adapter for web mode (same PtyLike interface)
    ai.ts             # AI connection test handler

e2e/                  # Playwright E2E test suite
  playwright.config.ts  # workers:1, 60s timeout, html+list reporters
  fixtures/app.ts       # Electron launch fixture, page access, cleanup
  helpers/              # selectors.ts, wait.ts
  tests/                # 10 spec files
```

## Key Patterns

- **Types**: All shared types live in `src/types/index.ts`. Import from there, not from services.
- **IPC Constants**: All channel names in `src/constants/ipc.ts`. Electron handlers use matching literals.
- **Storage Constants**: All localStorage keys in `src/constants/storage.ts`.
- **Theme**: Use `themeClass(resolvedTheme, { dark, light, modern })` from `src/utils/theme.ts` instead of nested ternaries.
- **Motion**: Shared framer-motion variants in `src/utils/motion.ts` — import from there, don't duplicate animation configs.
- **Component extraction**: `SplitPane` is a thin recursive router. Terminal leaf logic lives in `TerminalPane`. Agent orchestration lives in `useAgentRunner` hook.
- **Preload safety**: `electron/preload.ts` enforces channel allowlists — only whitelisted channels can be invoked/sent/received.
- **Provider config caching**: Settings stores per-provider configs (model, apiKey) in localStorage. Switching providers preserves previously entered credentials and auto-saves provider switch.
- **Provider helpers**: `providerUsesBaseUrl()`, `isAnthropicProtocol()`, `isProviderUsable()` in `ai/index.ts`.
- **Window close**: Electron intercepts close, sends `window.confirmClose` to renderer. "Exit Without Saving" uses `discardPersistedLayout()` with file-based flag. `before-quit` (Cmd+Q) bypasses.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shadowhusky/Tron](https://github.com/Shadowhusky/Tron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
