---
trigger: always_on
description: SuperCmd is an open-source alternative to Raycast, designed to provide a similar launcher experience while maintaining full compatibility with Raycast extensions. The project aims to achieve feature parity with Raycast while remaining open-source and community-driven.
---

# SuperCmd - Open Source Raycast Alternative

## Project Overview

SuperCmd is an open-source alternative to Raycast, designed to provide a similar launcher experience while maintaining full compatibility with Raycast extensions. The project aims to achieve feature parity with Raycast while remaining open-source and community-driven.

### Core Principles

1. **Extension Compatibility**: The app must be compatible with existing Raycast extensions without requiring modifications to extension code
2. **Runtime Control**: All changes and enhancements must be implemented in SuperCmd itself, not in extensions, since we cannot control extension code at runtime
3. **API Parity**: Keep APIs in sync with `@raycast/api` and track implementation status against the official Raycast API
4. **Progressive Enhancement**: Gradually implement all Raycast APIs to achieve full parity

## Architecture

### Project Structure

```
launcher/
├── src/
│   ├── main/                          # Electron main process
│   │   ├── main.ts                    # Entry point; IPC handlers, window management, global shortcuts
│   │   ├── preload.ts                 # contextBridge — exposes window.electron API to renderer
│   │   ├── commands.ts                # App/settings/extension/script discovery; getAvailableCommands() with cache
│   │   ├── extension-runner.ts        # Extension execution engine (esbuild bundle + require shim)
│   │   ├── extension-registry.ts      # Extension catalog, install, uninstall, update
│   │   ├── script-command-runner.ts   # Raycast-compatible script command execution
│   │   ├── ai-provider.ts             # AI streaming (OpenAI / Anthropic / Ollama) via Node http/https
│   │   └── settings-store.ts          # JSON settings persistence (AppSettings, cached in memory)
│   ├── renderer/                      # Electron renderer process (UI)
│   │   ├── types/
│   │   │   └── electron.d.ts          # TypeScript types for window.electron IPC bridge
│   │   └── src/
│   │       ├── App.tsx                # Root component — composes all hooks, routes to view components
│   │       ├── raycast-api/            # @raycast/api + @raycast/utils compatibility runtime modules
│   │       │   ├── index.tsx          # Integration/export surface (wires runtime modules)
│   │       │   ├── action-runtime*.tsx # Action/ActionPanel registry + overlay runtime
│   │       │   ├── list-runtime*.tsx   # List runtime (item registry, renderers, detail)
│   │       │   ├── form-runtime*.tsx   # Form runtime (container + fields + context)
│   │       │   ├── grid-runtime*.tsx   # Grid runtime (item registry + renderer + container)
│   │       │   ├── detail-runtime.tsx  # Detail runtime
│   │       │   └── menubar-runtime*.tsx # MenuBarExtra runtime
│   │       ├── hooks/                 # Feature hooks (state + logic, no JSX)
│   │       │   ├── useAppViewManager.ts      # View state machine — which screen is active
│   │       │   ├── useAiChat.ts              # AI chat mode state + streaming
│   │       │   ├── useCursorPrompt.ts        # Inline AI cursor prompt state + streaming
│   │       │   ├── useMenuBarExtensions.ts   # Menu-bar extension lifecycle
│   │       │   ├── useBackgroundRefresh.ts   # Interval-based background refresh for extensions/scripts
│   │       │   ├── useSpeakManager.ts        # TTS (Read) overlay state + portal
│   │       │   └── useWhisperManager.ts      # Whisper STT overlay state + portals
│   │       ├── views/                 # Full-screen view components (pure UI, state from hooks)
│   │       │   ├── AiChatView.tsx                  # Full-screen AI chat panel
│   │       │   ├── CursorPromptView.tsx             # Inline/portal AI cursor prompt UI
│   │       │   ├── ScriptCommandSetupView.tsx       # Script argument collection form
│   │       │   ├── ScriptCommandOutputView.tsx      # Script stdout/stderr output viewer
│   │       │   └── ExtensionPreferenceSetupView.tsx # Extension preference/argument form
│   │       ├── utils/                 # Pure utility modules (no side-effects)
│   │       │   ├── constants.ts              # localStorage keys, magic numbers, error strings
│   │       │   ├── command-helpers.tsx        # filterCommands, icon renderers, display helpers
│   │       │   └── extension-preferences.ts  # localStorage helpers, preference hydration, missing-pref checks
│   │       ├── ExtensionView.tsx      # Renders a live Raycast extension inside the launcher
│   │       ├── settings/              # Settings window UI (AITab, ExtensionsTab, GeneralTab, etc.)
│   │       └── useDetachedPortalWindow.ts    # Hook to open/manage a detached Electron overlay window
│   └── native/                        # Native Swift modules
└── dist/                              # Build output
```

### Extension Execution Model

1. **Extension Loading**: Extensions are loaded from the Raycast extension registry
2. **Code Bundling**: Extension code is bundled using esbuild to CommonJS
3. **Runtime Shim**: A custom `require()` function provides:
   - React (shared instance with host app)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperCmdLabs/SuperCmd](https://github.com/SuperCmdLabs/SuperCmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
