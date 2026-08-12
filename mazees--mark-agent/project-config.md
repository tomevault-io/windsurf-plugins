---
trigger: always_on
description: **Project Name:** MARK (Metacognitive Artificial Relational Knowledge) — v4.0.0
---

# AI Context & Planning (AGENTS.md)

## 1. Project Overview

**Project Name:** MARK (Metacognitive Artificial Relational Knowledge) — v4.0.0
**Description:** A privacy-first, local-based autonomous AI OS companion designed to assist user productivity, automate tasks, and provide lifelike companionship. It uses a hybrid AI engine (Local LLM via LM Studio or Cloud API, plus a native Gemini Web RPC Engine) and features agentic planning with ReAct loop execution, **Durable Agent Tasks** (UI: **Agent Workflows**, branding: **Mission Control**) for persistent multi-step work, autonomous physical browser automation, a hybrid Full-Text & Vector Memory Management System (MMS) with Orama & Dexie, document RAG pipeline, OS-level Awareness Engine, dynamic 4D Relational Growth, a native Plugin System with Monaco Editor, Telegram Bot integration via Telegraf, Voice Activity Detection with Groq Whisper STT, Edge-TTS, and webcam vision capabilities.
**Environment:** Electron 39 desktop application optimized for Windows (Windows 10/11).
**Author:** Mazees | **Homepage:** https://github.com/Mazees/mark-agent/

## 2. Technology Stack & Core Dependencies

- **Framework:** Electron 39, React 19, Vite 7, electron-vite 5
- **UI/Styling:** Tailwind CSS 4 (via `@tailwindcss/vite`), DaisyUI 5 (theme: `forest`), Poppins + Inter fonts, React Markdown, React Syntax Highlighter (Prism, oneDark), Monaco Editor (`@monaco-editor/react`), Driver.js (guided tours), Lucide React, React Icons
- **AI Backend:** Gemini Web RPC (Native Bridge) / Groq API / LM Studio (Local, `localhost:1234`) / Cerebras / Custom OpenAI-compatible Endpoint
- **Embeddings/Memory:** `@huggingface/transformers` (Transformers.js) for fully local embeddings via WASM (`Xenova/paraphrase-multilingual-MiniLM-L12-v2`, 384 dimensions)
- **Local Database & Vector Search:** `dexie` (IndexedDB wrapper, v14 schema with 6 stores) and `@orama/orama` for Hybrid Full-Text & Vector search
- **Web Capabilities:** Physical background `BrowserWindow` automation (`src/main/browser-agent.js`) with DOM parsing, animated cursor injection, and React-compatible input binding
- **Voice/Audio:** Groq API (Speech-to-Text via `whisper-large-v3`), Edge-TTS (`msedge-tts`, voice: `id-ID-ArdiNeural`), Web Audio API (Voice Activity Detection via `useVAD.js`, 16kHz sample rate, RMS threshold 0.015)
- **Media/Integrations:** `youtube-transcript-plus`, `ytmusic-api` (YouTube Music with ad-blaster), `yt-search`, `youtube-dl-exec` + `ffmpeg-static`
- **Communication:** `telegraf` (Telegram Bot Framework)
- **Packaging:** `electron-builder` (NSIS installer, `appId: com.mark.agent`, `asarUnpack` for ffmpeg/yt-dlp binaries)

## 3. Project Architecture & File Structure

### `src/main/` — Electron Main Process

| File                           | Purpose                     | Key Details                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------------ | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index.js`                     | Main entry point            | 40+ IPC handlers, window management, System Tray with 16x16 icon extraction, global shortcut `Ctrl+Alt+M`, TTS via `MsEdgeTTS`, YouTube search/transcript/music, `desktopCapturer` screenshots (720p), single-instance lock, Chromium GPU mitigations (`disable-gpu-process-crash-limit`, `CalculateNativeWinOcclusion`, `disable-background-timer-throttling`)                                                                                                                                                                                                                  |
| `ai-bridge.js`                 | Centralized AI HTTP client  | `fetchAI()` with multi-provider routing (Groq/Cerebras/Custom/LM Studio), 3-tier JSON format fallback (`json_schema` → `json_object` → unrestricted), rate limiting (`CLOUD_DELAY_MS=3000`), 429/503 exponential backoff with auto model swap to `openai/gpt-oss-20b`, vision payload sanitization (strips old images), DeepSeek `<think>` tag extraction, `cleanAndParse()` with `jsonrepair`, 5-minute timeout for local LLM                                                                                                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mazees/mark-agent](https://github.com/Mazees/mark-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
