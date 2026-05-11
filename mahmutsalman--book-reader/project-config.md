---
trigger: always_on
description: Project context for Claude Code. Read this before making changes.
---

# Smart Book — CLAUDE.md

Project context for Claude Code. Read this before making changes.

---

## What This App Is

**Smart Book** is an Electron desktop app for reading books while learning languages.
Core features: word lookup (AI-powered), vocabulary tracking, TTS pronunciation, IPA phonetics,
manga OCR, pre-study note generation, and multi-language support.

Stack: **Electron + React + TypeScript + SQLite + Python (FastAPI)**

---

## Project Structure

```
src/
├── main.ts                  ← Electron main process entry
├── preload.ts               ← IPC bridge (contextBridge → window.electronAPI)
├── renderer.tsx             ← React DOM entry
│
├── main/                    ← Main process only
│   ├── ipc/                 ← One file per domain (book, ai, update, etc.)
│   └── services/            ← Business logic (AI providers, import, Python mgr, etc.)
│
├── renderer/                ← React UI
│   ├── pages/               ← LibraryPage, ReaderPage, VocabularyPage, SettingsPage
│   ├── components/
│   │   ├── reader/          ← DynamicReaderView, MangaImageView, WordPanel, etc.
│   │   ├── word-panel/      ← WordPanel, PronunciationButton, LoopPlayButton
│   │   ├── vocabulary/      ← VocabularyTabs, BookFilter, ExportContextMenu
│   │   └── Settings/        ← OCRSettings
│   ├── context/             ← React contexts (Book, Settings, FocusMode, etc.)
│   ├── hooks/               ← useReaderTheme, useAudioPlayer, useMeaningAnalysis, etc.
│   └── services/            ← Renderer-side services (audio cache, deferred context)
│
├── database/
│   ├── index.ts             ← SQLite init (better-sqlite3)
│   ├── migrations/          ← Numbered SQL migrations (001–007)
│   └── repositories/        ← book, vocabulary, progress, settings
│
├── shared/                  ← Used by BOTH main and renderer
│   ├── constants/
│   │   └── ipc-channels.ts  ← ALL IPC channel name strings live here
│   └── types/               ← TypeScript interfaces for every domain
│
└── python-server/
    ├── server.py             ← FastAPI server (TTS + IPA endpoints)
    ├── generators/           ← tts.py, ipa.py (MUST be packaged — see forge.config.ts)
    ├── python-runtime/       ← Embedded Python 3.11 (bundled, no user install needed)
    ├── build.sh / build.bat  ← Sets up the embedded runtime
    └── launch-server.sh/.bat ← Spawned by python-manager.service.ts at startup
```

---

## IPC Pattern — How Main ↔ Renderer Talk

Every feature follows the same 4-file pattern. Always update all 4 when adding a feature:

```
1. src/shared/constants/ipc-channels.ts   → add channel name constant
2. src/main/ipc/{domain}.ipc.ts           → add ipcMain.handle() handler
3. src/preload.ts                          → expose via contextBridge
4. src/shared/types/ipc.types.ts          → add to ElectronAPI interface
```

Renderer calls: `window.electronAPI.{domain}.{method}()`
Main pushes events to renderer: `mainWindow.webContents.send('channel-name', data)`
Renderer listens: `ipcRenderer.on('channel-name', handler)` exposed via preload.

---

## Key Files to Know

| File | What it does |
|---|---|
| `src/main.ts` | App lifecycle, DB init, Python server start, Squirrel auto-update (Windows) |
| `src/preload.ts` | The ONLY bridge between main and renderer — keep secure |
| `forge.config.ts` | Build config — Windows uses MakerSquirrel, macOS uses MakerZIP + MakerDMG |
| `src/main/services/python-manager.service.ts` | Spawns/kills the Python server process |
| `src/main/services/update.service.ts` | Custom update checker (fetches VPS latest.json, used for macOS) |
| `src/main/ipc/update.ipc.ts` | Update IPC + Squirrel `quitAndInstall` handler |
| `src/renderer/components/reader/DynamicReaderView.tsx` | The main reading component — complex, many refs |
| `src/database/migrations/` | Add a new numbered file here for schema changes |
| `.github/workflows/build-release.yml` | CI/CD: builds both platforms, deploys to VPS |
| `scripts/generate-update-manifest.js` | Generates `latest.json` for macOS update feed |

---

## Auto-Update Architecture

Two separate mechanisms — one per platform:

### Windows (Squirrel)
- `MakerSquirrel` creates `SmartBookSetup.exe` + `RELEASES` + `.nupkg`
- In `main.ts`: `autoUpdater.setFeedURL({ url: 'https://smartbook.mahmutsalman.cloud/releases/windows' })`
- Checks silently 10s after launch
- When ready: sends `squirrel:update-downloaded` → renderer shows `UpdateReadyBanner`
- User clicks "Restart Now" → `UPDATE_INSTALL` IPC → `autoUpdater.quitAndInstall()`
- VPS must serve: `RELEASES` file + `.nupkg` packages at `/releases/windows/`

### macOS (Custom)
- `update.service.ts` fetches `https://smartbook.mahmutsalman.cloud/releases/latest.json`
- Checks 3s after launch (renderer-initiated via `window.electronAPI.update.check()`)
- Shows `UpdateNotification` modal → "Download Update" opens browser
- VPS must serve: `latest.json` manifest + ZIP/DMG files at `/releases/macos/`

---

## Python Server

- Runs as a **subprocess** on port 5000 (started by `python-manager.service.ts`)
- Provides: `/tts` (text-to-speech), `/ipa` (phonetic transcription)
- Uses **embedded Python runtime** — users don't install Python
- The `generators/` directory MUST be packaged (see `forge.config.ts` `extraResource`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mahmutsalman/book-reader](https://github.com/mahmutsalman/book-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
