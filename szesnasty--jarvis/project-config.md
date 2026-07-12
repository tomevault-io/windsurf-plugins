---
trigger: always_on
description: Jarvis is a personal memory, planning, and knowledge system**.
---

# JARVIS — Full Project Plan

## 1. What Is Jarvis

Jarvis is a personal memory, planning, and knowledge system**.

It is **not**:
- A coding tool
- A Claude Code wrapper
- A generic chatbot with memory

It **is**:
- A voice interface to the user's own local memory
- A personal operating system / second brain / voice memory cockpit
- Built on local Markdown files, compatible with Obsidian
- Powered by Claude API
- Extended with a knowledge graph layer
- Customizable through user-created specialists

**Core thesis**: AI doesn't remember "on its own" — it works on memory that **belongs to the user**.

---

## 1a. Source of Truth Doctrine

**This is a hard rule for the entire codebase:**

- **Source of truth = Markdown files in `Jarvis/memory/`**
- **SQLite = operational index and cache, never the canonical store.** If SQLite is deleted, it must be fully rebuildable from Markdown files.
- **Graph = derived layer, rebuildable from memory.** If `graph.json` is deleted, it can be regenerated.
- **Config = `Jarvis/app/config.json`** for metadata and flags only.

Every write operation that affects user knowledge must end with a Markdown file on disk. SQLite and graph are acceleration layers, not storage layers.

---

## 2. Target Users

- People organizing life, projects, and plans
- People who take lots of notes and want to connect them
- People who want a personal assistant for health, learning, travel, relationships
- Obsidian users (optional power-up)
- Anyone who wants full control over their data and knowledge structure

---

## 3. Key Use Cases

### A. Weekly Planning
User says: "Plan my week based on recent notes."
System searches local memory, considers projects/relations, creates a plan, saves it.

### B. Brain Dump → Plan
User dumps chaotic thoughts. System transcribes, organizes, creates a checklist, splits into today/week/later.

### C. Conversational Memory Recall
User asks: "What did I decide about vacations?" / "What did I promise Michał?"
System searches memory + graph, returns concise answer.

### D. Working with User's Knowledge Sources
User adds health notes, PDFs, checklists, articles. Then asks: "What do my materials say about sleep and energy?"

### E. Custom Specialists
User creates specialists (Health Guide, Weekly Planner, Study Coach, etc.) with own sources, rules, style, examples, and tools.

---

## 4. Confirmed Product Decisions

1. Not a coding app
2. Not based on Claude Code
3. Main engine: **Claude API** (Messages API)
4. Interface runs **in the browser**
5. User provides only: **Anthropic API key** + microphone permission
6. **API key security**: Do not store Anthropic API key in plain text if avoidable. Prefer OS keychain integration (e.g., `keyring` Python package). `config.json` stores only a flag `api_key_set: true`, not the key itself. Fallback for MVP: environment variable or encrypted local storage.
7. Memory: local, Markdown-based, Obsidian-compatible
8. Obsidian: optional, not required
9. Knowledge graph layer: yes
10. Voice: important from the start
11. User-created specialists from UI: yes
12. Token-efficient architecture: yes
13. No over-engineered agent framework on start

---

## 5. Tech Stack (MVP)

### Frontend
- **Nuxt 3** (Vue 3 + file-based routing + auto-imports)
- **TypeScript** (strict mode)
- **Nitro** (built-in dev proxy to backend)
- **useState()** composable for shared state (no Pinia needed)
- CSS: simple scoped CSS (start minimal)

### Backend
- **Python 3.12+**
- **FastAPI** (HTTP + WebSocket)
- **SQLite** (operational DB via `aiosqlite`)
- **Anthropic Python SDK** (`anthropic`)
- File I/O for Markdown memory

### Voice
- **Web Speech API** (browser-native STT) — no extra API key needed
- **Browser SpeechSynthesis** (TTS) for MVP — upgrade later if needed
- Fallback: Anthropic or other TTS if quality is insufficient

**Voice abstraction rule**: Voice layer must be abstracted behind interfaces (`STTProvider`, `TTSProvider`). Browser-native STT/TTS is the MVP default. Future providers (Whisper local, Kokoro.js, Piper, cloud TTS) can replace implementation without changing app flow. No voice provider should be hardwired into components.

### Knowledge Graph
- Lightweight local graph stored as JSON
- Visualized with **D3.js** or **vis-network** in browser
- No heavy graph DB on start

### Communication
- REST API for CRUD operations
- WebSocket for streaming Claude responses and voice state sync

---

## 6. Repository Structure

```
jarvis/
├── docs/
│   └── JARVIS-PLAN.md          # Initial plan
├── backend/
│   ├── main.py                  # FastAPI entry point
│   ├── requirements.txt
│   ├── config.py                # App configuration
│   ├── routers/
│   │   ├── chat.py              # Chat / Claude API endpoints
│   │   ├── memory.py            # Memory CRUD endpoints
│   │   ├── graph.py             # Graph query endpoints
│   │   ├── specialists.py       # Specialist CRUD endpoints
│   │   ├── workspace.py         # Workspace setup endpoints
│   │   └── voice.py             # Voice-related endpoints
│   ├── services/
│   │   ├── claude.py            # Claude API wrapper
│   │   ├── memory_service.py    # Memory read/write/search
│   │   ├── graph_service.py     # Graph operations
│   │   ├── specialist_service.py# Specialist management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Szesnasty/Jarvis](https://github.com/Szesnasty/Jarvis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
