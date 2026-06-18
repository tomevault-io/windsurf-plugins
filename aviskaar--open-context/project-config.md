---
trigger: always_on
description: **opencontext** is a tool that lets users migrate their full chat history from AI providers (ChatGPT, Google Gemini, etc.) into Claude-compatible formats. It ships as both a **CLI** and a **web UI**, and includes an **MCP server** so Claude itself can save and recall context across conversations.
---

# CLAUDE.md — AI Assistant Guide for opencontext

## Project Overview

**opencontext** is a tool that lets users migrate their full chat history from AI providers (ChatGPT, Google Gemini, etc.) into Claude-compatible formats. It ships as both a **CLI** and a **web UI**, and includes an **MCP server** so Claude itself can save and recall context across conversations.

### The Problem

When users switch to Claude, they lose all prior conversation context from other AI platforms. Chat exports from providers like ChatGPT come in provider-specific JSON formats that Claude can't read directly.

### The Solution

opencontext reads exported conversation archives, normalizes them, and outputs Claude-compatible conversation data — so users can hit the ground running on a new Claude account with their full history.

### Supported Sources

- **ChatGPT** — `conversations.json` from the ChatGPT data export (Settings → Export data) ✅ Implemented
- **Google Gemini** — Gemini activity export via Google Takeout (planned)
- **Other providers** — Extensible parser system for adding new sources

### Components

1. **CLI** (`src/`) — Node.js/TypeScript CLI for batch converting chat exports
2. **HTTP server** (`src/server.ts`) — Express REST API that serves the built UI and exposes the conversion pipeline + context store over HTTP
3. **Web UI** (`ui/`) — React + Vite dashboard for managing preferences, importing conversations, and exporting to multiple vendors
4. **MCP server** (`src/mcp/`) — Model Context Protocol server that lets Claude save/recall persistent context

### Docker Hub

**Image:** [`adityakarnam/opencontext:latest`](https://hub.docker.com/r/adityakarnam/opencontext)

Single image containing UI + API server + MCP server. Default CMD runs the HTTP server on port 3000. Override CMD to `node dist/mcp/index.js` for MCP stdio mode.

---

## Repository Structure

```
opencontext/
├── CLAUDE.md                   # This file
├── README.md                   # User-facing docs
├── package.json                # CLI/MCP dependencies and scripts
│
├── src/                        # CLI + HTTP server + MCP server
│   ├── index.ts                # CLI entry point (Commander.js)
│   ├── server.ts               # Express HTTP server (UI + REST API, port 3000)
│   ├── extractor.ts            # ZIP extraction & temp file management
│   ├── parsers/
│   │   ├── types.ts            # TypeScript interfaces for parser output
│   │   ├── chatgpt.ts          # ChatGPT conversations.json parser
│   │   └── normalizer.ts       # Normalize parsed data to common schema
│   ├── formatters/
│   │   └── markdown.ts         # Markdown output formatter
│   ├── analyzers/
│   │   └── ollama-preferences.ts  # AI-powered preference analysis via Ollama
│   ├── utils/
│   │   └── file.ts             # File I/O utilities
│   └── mcp/                    # MCP server
│       ├── index.ts            # MCP entry point (stdio transport)
│       ├── server.ts           # Tool definitions (save/recall/list/search/update/delete)
│       ├── store.ts            # JSON file-based context store (~/.opencontext/contexts.json)
│       └── types.ts            # ContextEntry, ContextStore types
│
├── ui/                         # Web UI (React + Vite)
│   ├── package.json            # UI dependencies (React 19, React Router 7, Lucide)
│   ├── vite.config.ts          # Vite build config
│   ├── src/
│   │   ├── main.tsx            # React app entry
│   │   ├── App.tsx             # Router and route definitions
│   │   ├── App.css             # App-level styles
│   │   ├── index.css           # Global reset, Tailwind directives, shadcn CSS variables
│   │   ├── components/
│   │   │   ├── Layout.tsx          # App shell with sidebar nav
│   │   │   ├── Dashboard.tsx       # Home page: context overview + privacy toggle + MCP setup
│   │   │   ├── PreferencesEditor.tsx  # Full preferences form (6 sections)
│   │   │   ├── ContextViewer.tsx   # Conversation import and management
│   │   │   ├── ConversionPipeline.tsx # Pipeline progress visualization
│   │   │   └── VendorExport.tsx    # Export to Claude/ChatGPT/Gemini
│   │   ├── store/
│   │   │   └── context.tsx         # React Context + useReducer state management
│   │   ├── types/
│   │   │   └── preferences.ts      # Shared TypeScript types
│   │   └── exporters/
│   │       ├── index.ts            # Exporter registry
│   │       ├── base.ts             # VendorExporter interface
│   │       ├── claude.ts           # Claude preferences/memory exporter
│   │       ├── chatgpt.ts          # ChatGPT custom instructions exporter
│   │       └── gemini.ts           # Gemini instructions exporter
│
└── tests/                      # CLI test suite (vitest)
    └── ...
```

---

## Tech Stack

### CLI / MCP Server (`src/`)
- **Runtime**: Node.js 25+ / TypeScript 5.9
- **CLI framework**: Commander.js
- **MCP**: `@modelcontextprotocol/sdk`
- **AI analysis**: Ollama (local LLM, optional)
- **ZIP handling**: adm-zip
- **Build**: `tsc`, run with `tsx` in dev

### Web UI (`ui/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviskaar/open-context](https://github.com/aviskaar/open-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
