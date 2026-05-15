---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Python CLI tool that converts a Claude.ai data export ZIP into a browsable static HTML website. The export ZIP contains `users.json`, `projects.json`, `memories.json`, and `conversations.json`. The output is a flat directory with `index.html`, `style.css`, and `conversations/{uuid}.html`.

## Commands

```bash
# Run against a real export
uv run claude-export-viewer path/to/export.zip -o path/to/output/

# Tests
uv run pytest tests/ -v
uv run pytest tests/test_renderer.py::test_render_message_artifact_create  # single test

# Lint
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run ruff check --fix src/ tests/ && uv run ruff format src/ tests/  # auto-fix
```

## Architecture

The data flows in one direction: **ZIP → models → renderer → templates → HTML files**.

- **`models.py`** — Pydantic v2 models for all four JSON files. Content blocks use a `ContentBlock` discriminated union on the `type` field (`text`, `thinking`, `tool_use`, `tool_result`, `token_budget`). The `ToolUseContent` model has special handling for artifacts via `is_artifact` / `artifact_input` properties — artifact tool_use blocks have `name="artifacts"` and their `input` dict contains `command`, `type`, `title`, `content`, etc.

- **`loader.py`** — Opens the ZIP, finds JSON files by suffix (handles nested paths), and returns an `ExportData` instance.

- **`renderer.py`** — Converts each `ChatMessage` into an HTML string. The key function is `render_message()` which dispatches to per-block-type renderers. Markdown rendering uses mistune v3 with a custom `_HighlightRenderer` that extends `mistune.HTMLRenderer` (not `BaseRenderer` — this matters because `HTMLRenderer.render_token` unpacks token dicts into method args). Pygments handles code highlighting at build time.

- **`html_builder.py`** — Loads Jinja2 templates, renders index + each conversation page, copies `style.css`. Templates use `{{ message_html | safe }}` since message HTML is pre-rendered by the renderer.

- **`cli.py`** — Thin argparse wrapper.

## Rendering Rules

| Content type | How it renders |
| --- | --- |
| `text` | Full markdown via mistune + Pygments |
| `thinking` | Collapsible `<details>`, last summary as label |
| `tool_use` (artifacts, create/rewrite) | Bordered card with title bar; markdown artifacts rendered, code highlighted |
| `tool_use` (artifacts, update) | Skipped (no full content) |
| `tool_use` (other tools) | Collapsible with message text |
| `tool_result` (artifacts) | Skipped |
| `tool_result` (other) | Collapsible with pre-formatted output, truncated at 2000 chars |
| `token_budget` | Skipped |

## Claude.ai Data Export Format Specification

Complete reference for the ZIP export format produced by Claude.ai's "Export Data" feature.

### ZIP Structure

A standard ZIP containing four JSON files. Files may be at the root or in subdirectories — the loader finds them by suffix match (e.g. `data/users.json` works). Missing files are treated as empty arrays.

```text
export.zip
├── users.json          # Array of User
├── projects.json       # Array of Project
├── memories.json       # Array of Memories
└── conversations.json  # Array of Conversation
```

### users.json

```json
[
  {
    "uuid": "string (required)",
    "full_name": "string (required)",
    "email_address": "string (required)",
    "verified_phone_number": "string | null (optional)"
  }
]
```

### projects.json

```json
[
  {
    "uuid": "string (required)",
    "name": "string (required)",
    "description": "string (optional, default '')",
    "is_private": "bool (optional, default false)",
    "is_starter_project": "bool (optional, default false)",
    "prompt_template": "string (optional, default '')",
    "created_at": "ISO 8601 datetime | null (optional)",
    "updated_at": "ISO 8601 datetime | null (optional)",
    "creator": "object | null (optional, typically {uuid: string})",
    "docs": [
      {
        "uuid": "string (required)",
        "filename": "string (required)",
        "content": "string (optional, default '')"
      }
    ]
  }
]
```

### memories.json

```json
[
  {
    "project_memories": {"memory_id": "memory text (string→string map, default {})"},
    "account_uuid": "string (optional, default '')"
  }
]
```

### conversations.json

```json
[
  {
    "uuid": "string (required)",
    "name": "string (optional, default '')",
    "summary": "string (optional, default '')",
    "created_at": "ISO 8601 datetime | null (optional)",
    "updated_at": "ISO 8601 datetime | null (optional)",
    "account": {"uuid": "string"} | null,
    "project_uuid": "string | null (optional)",
    "chat_messages": ["ChatMessage (see below, default [])"]
  }
]
```

### ChatMessage

```json
{
  "uuid": "string (required)",
  "text": "string (optional, default '')",
  "content": ["ContentBlock (see below, default [])"],
  "sender": "'human' | 'assistant' (required)",
  "created_at": "ISO 8601 datetime | null (optional)",
  "updated_at": "ISO 8601 datetime | null (optional)",
  "attachments": [
    {
      "file_name": "string (default '')",
      "file_size": "int bytes (default 0)",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lordjabez/claude-export-viewer](https://github.com/lordjabez/claude-export-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
