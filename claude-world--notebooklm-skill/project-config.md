---
trigger: always_on
description: >
---


# NotebookLM Research Agent

A fully autonomous AI research agent that ingests sources into Google NotebookLM,
runs deep web research, synthesizes knowledge through cited Q&A and 9 downloadable artifact types,
creates polished content drafts, and optionally publishes to social platforms.

**Zero-cost research engine** -- NotebookLM is free. No API keys. No per-query charges.

## Authentication

NotebookLM uses RPC/HTTP calls after a one-time browser cookie auth. No browser
automation per operation -- the session is stored and reused.

```
~/.notebooklm/storage_state.json
```

Login once via the built-in CLI:

```bash
notebooklm login              # One-time browser auth, saves session
notebooklm login --check      # Verify stored session is still valid
```

The session persists until Google expires it (typically weeks). All scripts and the
MCP server auto-load the stored session. No API keys or environment variables needed.

## Architecture Overview

**Core Principle: NotebookLM provides cited research, Claude creates content.**

NotebookLM handles source ingestion, indexing, deep web research, cited answers,
and native artifact generation (9 downloadable types). Claude uses that research output to write
original articles, social posts, and reports. The pipeline is zero-cost and produces
citation-backed content.

| Component | Role |
|---|---|
| **notebooklm-py** (v0.3.4) | Python client for NotebookLM (8 sub-APIs, 50+ methods, built-in CLI) |
| **notebooklm CLI** | Built-in CLI: `notebooklm login`, `notebook`, `source`, `chat`, `generate`, `download`, `research`, `share` |
| **MCP Server** (mcp_server/) | FastMCP server exposing 13 tools for Claude Code / Cursor / Gemini CLI |
| **Wrapper CLI** (scripts/) | Our higher-level wrappers: `notebooklm_client.py`, `pipeline.py` |
| **LLM** (Claude) | Content creator (writes original text using NotebookLM research) |
| **trend-pulse** (optional) | Trending topic discovery for research-to-content pipelines |
| **threads-viral-agent** (optional) | Social publishing for content distribution |

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                          NOTEBOOKLM RESEARCH AGENT                              │
├──────────────┬──────────────┬─────────────────┬─────────────────────────────────┤
│  Phase 1     │  Phase 2     │   Phase 3       │    Phase 4                      │
│  INGEST      │  SYNTHESIZE  │   CREATE        │    PUBLISH (optional)           │
│              │              │                 │                                 │
│ Sources:     │ Chat:        │ Claude writes:  │ threads-viral-agent:            │
│  URL         │  ask()       │  Articles ────→ │  → Threads                      │
│  Text        │  → cited     │  Social posts → │  → Instagram                    │
│  PDF/DOCX    │    answers   │  Newsletters  → │  → Facebook                     │
│  YouTube     │  → follow-up │  Reports ─────→ │                                 │
│  Google Drive│  → citations │                 │ Direct output:                  │
│  File upload │              │ trend-pulse     │  → Markdown file                │
│              │ Artifacts    │  → topic ideas  │  → JSON data                    │
│ Research:    │ (9 types):   │                 │  → Newsletter draft             │
│  web (fast)  │  audio       │ NotebookLM      │  → Podcast MP4                  │
│  web (deep)  │  video       │ artifacts used  │  → Video MP4                    │
│  drive       │  cinematic*  │ directly:       │  → Slide deck PDF               │
│              │  slide_deck  │  → Podcast      │  → Quiz / Flashcards            │
│ Auto-import  │  report      │  → Report       │                                 │
│ discovered   │  quiz        │  → Data table   │                                 │
│ sources      │  flashcards  │  → Infographic   │                                 │
│              │  mind_map    │                 │ * cinematic = Veo 3,            │
│              │  infographic │                 │   AI Ultra only                 │
│              │  data_table  │                 │                                 │
│              │  study_guide │                 │                                 │
└──────────────┴──────────────┴─────────────────┴─────────────────────────────────┘
```

### 8 Sub-APIs (notebooklm-py v0.3.4)

| Sub-API | Accessor | Description |
|---|---|---|
| **Notebooks** | `client.notebooks` | Create, list, get, delete, rename, describe, share |
| **Sources** | `client.sources` | Add URL/text/file/Drive, list, delete, rename, refresh, guide, fulltext, wait |
| **Artifacts** | `client.artifacts` | Generate 9 downloadable types, poll status, download, list, delete, rename, revise slides |
| **Chat** | `client.chat` | Ask with citations, follow-up, conversation history, configure persona |
| **Research** | `client.research` | Web/Drive research, poll results, import discovered sources |
| **Notes** | `client.notes` | Create, list, update, delete text notes and mind maps |
| **Settings** | `client.settings` | User settings (output language) |
| **Sharing** | `client.sharing` | Public links, user permissions, view levels |

## Phase 1: INGEST -- Source Collection

Create a notebook and populate it with sources. NotebookLM accepts 8 source types:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claude-world/notebooklm-skill](https://github.com/claude-world/notebooklm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
