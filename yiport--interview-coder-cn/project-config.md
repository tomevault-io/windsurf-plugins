---
trigger: always_on
description: **Interview Coder CN (Voice Enhanced)** is a desktop application forked from [yiport/interview-coder-cn](https://github.com/yiport/interview-coder-cn). It captures screenshots of coding problems and uses AI (vision models) to generate solutions in real-time. The window is invisible to screen-sharing software.
---

# AGENTS.md

## Project Overview

**Interview Coder CN (Voice Enhanced)** is a desktop application forked from [yiport/interview-coder-cn](https://github.com/yiport/interview-coder-cn). It captures screenshots of coding problems and uses AI (vision models) to generate solutions in real-time. The window is invisible to screen-sharing software.

This fork adds a complete voice interaction system: TTS (text-to-speech) for reading AI answers aloud, voice conversation mode (speak to AI without screenshots), microphone input support, and chat-style conversation display.

Key capabilities:
- Global shortcuts trigger screenshot capture → AI analysis → streamed solution display
- Frameless, transparent, always-on-top overlay window invisible to screen-sharing
- Mouse passthrough mode (window ignores mouse events)
- Multi-screenshot conversation continuity (append screenshots to existing context)
- Follow-up questions within the same conversation
- Real-time speech transcription (DashScope Fun-ASR) — transcribed text is attached to screenshots when sent to AI
- **NEW: TTS (Text-to-Speech)** — AI answers read aloud via Web Speech API or DashScope CosyVoice
- **NEW: Voice conversation mode** — speak questions without screenshots (Alt+Z to toggle)
- **NEW: Microphone input** — audio source selectable between system audio and microphone
- **NEW: Chat-style display** — user voice messages and AI responses shown as conversation bubbles
- **NEW: Conversation export** — download full conversation history as Markdown
- Configurable AI provider (OpenAI, SiliconFlow, DeepSeek, OpenRouter, or any OpenAI-compatible API)

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Electron 37 (electron-vite 4) |
| Frontend | React 19, TypeScript 5.8 |
| Styling | Tailwind CSS v4, shadcn/ui (New York style), Radix primitives |
| State | Zustand 5 (6 stores, 2 with localStorage persistence) |
| Routing | react-router v7 (HashRouter, 3 routes) |
| AI | Vercel AI SDK (`ai` + `@ai-sdk/openai`), streaming via `streamText()` |
| STT | DashScope Fun-ASR (WebSocket, PCM 16kHz) |
| TTS | Web Speech API (free) + DashScope CosyVoice (WebSocket, PCM 24kHz) |
| Build | electron-vite (Vite 7), electron-builder 25 |
| Linting | ESLint 9 (flat config), Prettier |

## Directory Structure

```
src/
├── main/                    # Electron main process
│   ├── index.ts             # App entry: lifecycle, error handling, app.whenReady()
│   ├── main-window.ts       # BrowserWindow creation (frameless, transparent, always-on-top)
│   ├── shortcuts.ts         # Global shortcuts + AI streaming orchestration + voice query handler
│   ├── ai.ts                # Vercel AI SDK integration, 4 streaming functions (incl. getVoiceStream)
│   ├── settings.ts          # App settings object + IPC handlers
│   ├── state.ts             # App state object + IPC handlers
│   ├── take-screenshot.ts   # desktopCapturer → base64 PNG
│   ├── transcription.ts     # DashScope WebSocket real-time speech-to-text (Fun-ASR)
│   ├── tts.ts               # DashScope WebSocket text-to-speech (CosyVoice)
│   ├── auto-updater.ts      # electron-updater (non-macOS only)
│   ├── prompts.md           # System prompt for AI (copied to build output via vite-plugin-static-copy)
│   └── index.d.ts           # global.mainWindow type declaration
├── preload/
│   ├── index.ts             # contextBridge API: exposes window.api to renderer
│   └── index.d.ts           # Type declarations for window.electron and window.api
└── renderer/
    ├── index.html            # SPA entry
    └── src/
        ├── main.tsx          # React root render
        ├── App.tsx           # Router + settings sync + shortcut init + Toaster
        ├── coder/            # Main page: screenshot display + AI solution stream
        │   ├── index.tsx     # CoderPage layout + state sync + transcription/TTS lifecycle + voice mode
        │   ├── AppHeader.tsx # Draggable title bar with nav buttons
        │   ├── AppContent.tsx# Screenshots gallery + markdown solution + error banner + auto-scroll
        │   ├── AppStatusBar.tsx    # Loading indicator, follow-up/export/TTS controls, voice mode indicator
        │   ├── TranscriptionBar.tsx # Absolute-positioned real-time transcription overlay
        │   └── PrerequisitesChecker.tsx  # Modal for API key setup
        ├── settings/         # Settings page
        │   ├── index.tsx     # AI config, TTS config, audio source, coding, appearance, shortcuts, privacy
        │   ├── SelectLanguage.tsx  # Combobox with custom language input
        │   ├── SelectModel.tsx     # Combobox with custom model input
        │   └── CustomShortcuts.tsx # Shortcut key recorder (incl. voice category)
        ├── help/             # Help page
        │   ├── index.tsx     # Quick start guide, shortcuts, FAQ
        │   ├── Shortcuts.tsx
        │   ├── FAQ.tsx
        │   └── components/index.tsx  # HelpSection wrapper
        ├── components/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YiPort/interview-coder-cn](https://github.com/YiPort/interview-coder-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
