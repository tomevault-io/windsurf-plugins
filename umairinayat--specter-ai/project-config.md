---
trigger: always_on
description: > **Project Codename:** `OpenCluely` (working name — rename as desired)
---

# CLAUDE.md — OpenCluely: Open-Source AI Screen & Meeting Copilot

> **Project Codename:** `OpenCluely` (working name — rename as desired)
> **Goal:** A free, open-source, privacy-first alternative to Cluely using OpenRouter as the AI backend, built as a cross-platform desktop app.

---

## 🧠 What Are We Building?

A **desktop app** that:

1. Captures your screen (via OCR) and microphone (via Whisper-style transcription)
2. Sends that context + your query to an **OpenRouter** model of your choice
3. Shows the AI response in an **always-on-top, transparent overlay** that is **invisible to screen capture / screen share**
4. Runs fully locally except the AI API call
5. Is fully open-source and free (BYOK — Bring Your Own OpenRouter API key)

---

## 🏗️ Tech Stack Decision

### Framework: **Electron + React + TypeScript**

Why Electron:
- Cluely itself is built with Electron (confirmed via reverse engineering)
- Provides `transparent: true` and `alwaysOnTop: true` BrowserWindow — the core of the invisible overlay
- Native OS APIs via Node.js (screen capture, audio, global hotkeys)
- Cross-platform: macOS, Windows, Linux from one codebase
- Large ecosystem, easy packaging

### Full Stack:

| Layer | Technology |
|---|---|
| Desktop Shell | Electron 28+ |
| UI Framework | React 18 + TypeScript |
| Styling | Tailwind CSS |
| Screen Capture / OCR | `screenshot-desktop` + `Tesseract.js` |
| Audio Capture | Node.js `mic` / `node-audiorecorder` |
| Speech-to-Text | OpenAI Whisper API (via OpenRouter) or `whisper.cpp` locally |
| AI Backend | OpenRouter API (OpenAI-compatible) |
| Global Hotkeys | `iohook` or `electron-globalshortcut` |
| State Management | Zustand |
| Packaging | `electron-builder` |
| Build Tool | Vite + `electron-vite` |

---

## 📁 Project Structure

```
opencluely/
├── CLAUDE.md                    ← You are here
├── package.json
├── electron.vite.config.ts
├── tsconfig.json
├── .env.example
│
├── src/
│   ├── main/                    ← Electron main process
│   │   ├── index.ts             ← App entry, window management
│   │   ├── overlay-window.ts    ← Invisible overlay BrowserWindow
│   │   ├── dashboard-window.ts  ← Settings/dashboard BrowserWindow
│   │   ├── screen-capture.ts    ← Screenshot + OCR pipeline
│   │   ├── audio-capture.ts     ← Mic recording + transcription
│   │   ├── hotkey-manager.ts    ← Global keyboard shortcuts
│   │   ├── tray.ts              ← System tray icon/menu
│   │   └── ipc-handlers.ts      ← IPC bridge main↔renderer
│   │
│   ├── renderer/                ← Electron renderer (React)
│   │   ├── overlay/             ← The invisible overlay UI
│   │   │   ├── App.tsx
│   │   │   ├── ResponseCard.tsx
│   │   │   ├── TranscriptBar.tsx
│   │   │   └── index.html
│   │   └── dashboard/           ← Settings/config UI
│   │       ├── App.tsx
│   │       ├── pages/
│   │       │   ├── Settings.tsx
│   │       │   ├── Models.tsx
│   │       │   ├── Playbooks.tsx
│   │       │   └── History.tsx
│   │       └── index.html
│   │
│   ├── shared/                  ← Shared types/utils
│   │   ├── types.ts
│   │   ├── constants.ts
│   │   └── ipc-channels.ts
│   │
│   └── services/                ← Core service modules
│       ├── openrouter.ts        ← OpenRouter API client
│       ├── ocr.ts               ← Tesseract OCR wrapper
│       ├── transcription.ts     ← Whisper / STT
│       ├── context-builder.ts   ← Combine screen+audio→prompt
│       └── store.ts             ← Persistent settings (electron-store)
│
├── assets/
│   ├── icon.png
│   └── tray-icon.png
│
└── build/                       ← electron-builder output
```

---

## 🔑 Core Concepts to Implement

### 1. The Invisible Overlay Window

This is the most critical feature. Electron's `BrowserWindow` with these flags makes it invisible to screen share:

```typescript
// src/main/overlay-window.ts
import { BrowserWindow, screen } from 'electron'

export function createOverlayWindow(): BrowserWindow {
  const { width, height } = screen.getPrimaryDisplay().workAreaSize

  const overlay = new BrowserWindow({
    width: 420,
    height: 600,
    x: width - 440,
    y: 20,
    transparent: true,          // ← transparent background
    frame: false,                // ← no window chrome
    alwaysOnTop: true,           // ← always on top of everything
    skipTaskbar: true,           // ← hidden from taskbar
    resizable: false,
    focusable: false,            // ← doesn't steal focus
    hasShadow: false,
    // The magic flag that makes it invisible to screen share:
    type: 'panel',               // macOS: excluded from screen capture
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true,
      preload: path.join(__dirname, 'preload.js'),
    },
  })

  // macOS: set window level above screen saver, excluded from capture
  overlay.setAlwaysOnTop(true, 'screen-saver')
  overlay.setVisibleOnAllWorkspaces(true, { visibleOnFullScreen: true })

  // Windows: use SetWindowDisplayAffinity to exclude from capture
  // This requires a native module or electron-overlay-window package
  if (process.platform === 'win32') {
    // overlay.setContentProtection(true) — this hides from ALL capture
    // For selective hiding (visible to user, not to screen share), use:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umairinayat/Specter-AI](https://github.com/umairinayat/Specter-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
