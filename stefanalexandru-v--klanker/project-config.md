---
trigger: always_on
description: | Dev server | `npm run dev` |
---

# AGENTS.md

## Commands

| Action | Command |
|--------|---------|
| Dev server | `npm run dev` |
| API server (SQLite) | `npm run api` |
| Both together | `npm run dev:full` |
| Build | `npm run build` |
| Run all web tests | `npm test` |
| Watch tests | `npm run test:watch` |
| Run single test file | `npx vitest run src/__tests__/api.test.js` |
| Start SearXNG | `docker start klanker-searxng` |
| Restart SearXNG | `docker restart klanker-searxng` |
| Run voice tests (WSL) | `cd klanker-voice && python3 -m pytest tests/ -v` |
| Voice app (Windows) | `cd klanker-voice && python -m src --debug` |
| Voice app demo mode | `cd klanker-voice && python -m src --demo --debug` |

## Architecture

Two applications sharing a SQLite database:

1. **Klanker Web** — Svelte 5 + Vite chat app with SSE streaming, web search, file/image support, LLM tool framework (shell, read_file, search)
2. **Klanker Voice** — Python voice assistant with wake word, STT, LLM streaming, floating widget

Both talk to the same **LM Studio** instance and **SearXNG** for web search.

### Web App

```
src/
  main.js                  # Mounts App into #app
  App.svelte               # Root layout: sidebar-rail, header, chat, input
  app.css                  # Global styles — Linear design system tokens
  lib/
    api.js                 # streamChat() async generator — SSE streaming client
    store.svelte.js        # createChatStore() — reactive state via Svelte 5 runes
    db.js                  # HTTP client → SQLite API server (was IndexedDB)
    fileParser.js          # Text extraction from file types (PDF, DOCX, XLSX, etc.)
    fileHandler.js         # File processing pipeline (image/document handling)
    search.js              # Web search integration via SearXNG
    tools.js               # Tool directive parser, API client, system prompt builder
  components/
    Chat.svelte            # Scrollable message list with smart auto-scroll + scroll-to-bottom
    Message.svelte         # Message bubble orchestrator (markdown + citation styling)
    MessageAttachments.svelte  # Image and file attachment display
    MessageSources.svelte  # Web search source citation chips
    ThinkingBlock.svelte   # Collapsible reasoning/thought display
    ToolCall.svelte        # Tool call status display with approve/deny buttons
    Input.svelte           # Textarea + send/stop buttons + drag-and-drop
    FileAttachments.svelte # Pending file/image attachment UI
    Sidebar.svelte         # Conversation list layout shell
    ConversationItem.svelte # Single conversation row with context menu + auto-focus rename
    SearchBox.svelte       # Search input with clear button
    ModelSelector.svelte   # Model dropdown with keyboard navigation + auto-focus
  __tests__/
    api.test.js            # streamChat tests with mocked fetch/ReadableStream
    store.test.js          # Store integration tests (28 tests — search, tools, streaming)
    tools.test.js          # Tool parsing + prompt builder tests (23 tests)
server/
  api.js                   # Node.js SQLite API server (better-sqlite3) — 10 REST endpoints
  tools/
    registry.js            # Tool definitions (search, shell, read_file)
    classify.js            # 3-tier command classifier (safe/approval/blocked)
    shell.js               # Shell executor with timeout + truncation
    readFile.js            # File reader with path validation + binary detection
    __tests__/             # 56 server-side unit tests (classify, shell, readFile)
```

### Voice App

```
klanker-voice/
  src/
    __init__.py
    __main__.py            # python -m entry point
    main.py                # App orchestration, system tray, QThread workers, --demo mode
    widget.py              # PyQt6 floating overlay — solid dark panel with animated orb
    db.py                  # Shared SQLite persistence (conversations + messages)
    llm.py                 # LM Studio streaming client (httpx SSE) with search loop
    search.py              # SearXNG web search client (Python port of search.js)
    wake.py                # OpenWakeWord background listener — "Hey Clanker" / Winston / Hey Jarvis
    transcribe.py          # faster-whisper STT with adaptive silence detection
  assets/
    hey_clanker.onnx       # Custom wake word model (trained via Colab)
    winston.onnx           # Fallback wake word — say "Winston"
  train/
    train_on_gpu.bat       # Windows batch script for GPU training on NVIDIA card
    COLAB_GUIDE.md         # Google Colab training instructions
    openwakeword-training/ # CoreWorxLab Docker-based trainer (CPU Dockerfile patched)
  tests/
    test_db.py             # 15 tests — SQLite CRUD
    test_llm.py            # 4 tests — SSE streaming, split chunks, system prompt
    test_main.py           # 5 tests — dismiss phrase detection (EN + RO)
  requirements.txt
  pyproject.toml
  README.md
```

### Shared SQLite Database

Both apps read/write the same SQLite file:
- **Windows**: `%APPDATA%\klanker\conversations.db`
- **Linux/WSL**: `~/.local/share/klanker/conversations.db`
- Override: `KLANKER_DB_PATH` env var

The web app talks to SQLite via the API server (`server/api.js` on port 3100, proxied via Vite `/api`). The voice app writes directly via Python `sqlite3`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StefanAlexandru-V) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
