---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants when working with this repository.

## Project Overview

SQLMap Web UI is a comprehensive SQL injection testing platform that includes:
- **Main Application**: FastAPI backend + Vue 3 frontend web interface
- **VulnShop Lab**: Built-in vulnerability testing environment
- **Browser Extensions**: Burp Suite plugins

## Documentation Structure

### Core Documents
- `README.md` / `README_EN.md` - Project overview and quick start guide
- `doc/CHANGELOG.md` - Chinese version changelog (**all version updates maintained here**)
- `doc/CHANGELOG_EN.md` - English version changelog (**all version updates maintained here**)
- `doc/USAGE_GUIDE.md` / `doc/USAGE_GUIDE_EN.md` - Detailed user guides

### Important Note
> **Changelog has been separated into `doc/CHANGELOG.md` and `doc/CHANGELOG_EN.md` documents**. README no longer includes detailed changelog. All future version updates should be written to the changelog documents.


## Project Structure

```
sqlmapWebUI/
├── src/
│   ├── backEnd/           # FastAPI backend service (Python 3.10+)
│   │   ├── api/           # API routes
│   │   │   ├── webApi/           # Web browser page API
│   │   │   ├── burpSuiteExApi/   # Burp Suite plugin API
│   │   │   └── commonApi/        # Common APIs (auth, headers, config)
│   │   ├── model/         # Data models
│   │   │   ├── requestModel/     # Request DTOs
│   │   │   ├── Task.py           # Task model
│   │   │   ├── ScanPreset.py     # Scan configuration presets
│   │   │   ├── ScanPresetDatabase.py  # Preset database operations
│   │   │   ├── HeaderScope.py    # Header scope configuration
│   │   │   ├── PersistentHeaderRule.py  # Persistent header rules
│   │   │   ├── SessionHeader.py  # Session-level headers
│   │   │   ├── SessionBodyField.py   # Session body field model
│   │   │   ├── HeaderBatch.py    # Batch header import
│   │   │   ├── HeaderDatabase.py # Header database operations
│   │   │   ├── LogRecorder.py    # Log recording
│   │   │   ├── StdDbOut.py       # SQLMap stdout capture
│   │   │   └── ...
│   │   ├── service/       # Business logic layer
│   │   │   ├── taskService.py    # Task management
│   │   │   ├── headerRuleService.py  # Header rules management
│   │   │   └── scanPresetService.py  # Scan preset management
│   │   ├── utils/         # Utility functions
│   │   │   ├── header_processor.py   # Header processing
│   │   │   ├── header_parser.py      # HTTP header parsing
│   │   │   ├── body_field_processor.py   # Body field processing
│   │   │   ├── scope_matcher.py      # Scope matching logic
│   │   │   ├── session_header_manager.py  # Session header management
│   │   │   ├── session_body_field_manager.py  # Session body field management
│   │   │   ├── content_type_helper.py    # Content-Type detection
│   │   │   ├── task_monitor.py       # Task monitoring
│   │   │   └── websocket_manager.py  # WebSocket connection management
│   │   ├── third_lib/sqlmap/     # SQLMap integration (git submodule)
│   │   ├── app.py         # FastAPI application core
│   │   └── main.py        # Entry point
│   ├── frontEnd/          # Vue 3 frontend (TypeScript + Vite)
│   │   └── src/
│   │       ├── api/       # API request functions
│   │       ├── components/# Shared components
│   │       │   ├── TaskFilter.vue    # Task filtering component
│   │       │   ├── TaskSummary.vue   # Task statistics summary
│   │       │   ├── ScopeConfigPanel.vue  # Scope configuration UI
│   │       │   ├── HttpCodeEditor.vue    # Code editor with syntax highlighting
│   │       │   └── GuidedParamEditor.vue # Guided SQLMap parameter editor
│   │       ├── stores/    # Pinia state management
│   │       │   ├── task.ts          # Task state store
│   │       │   ├── config.ts        # Config state store
│   │       │   └── scanPreset.ts    # Scan preset state store
│   │       ├── types/     # TypeScript type definitions
│   │       ├── utils/     # Utility functions
│   │       └── views/     # Page views
│   │           ├── Home/            # Dashboard
│   │           ├── TaskList/        # Task list page
│   │           ├── TaskDetail/      # Task detail page
│   │           ├── AddTask/         # Add scan task page
│   │           └── Config/          # Configuration page
│   ├── burpEx/            # Burp Suite extensions
│   │   ├── legacy-api/    # Legacy Burp API (Java 11)
│   │   └── montoya-api/   # Montoya API (Java 17, Burp 2023.1+)
│   └── vulnTestServer/    # VulnShop vulnerability lab
│       ├── server.py      # HTTP server with vulnerable endpoints
│       ├── database.py    # SQLite database with vulnerable queries
│       ├── waf.py         # WAF module (3 difficulty levels)
│       └── static/        # Frontend static assets
├── .github/workflows/     # GitHub Actions CI/CD
└── doc/                   # Project documentation
```

## Technology Stack

| Component | Technologies |
|-----------|-------------|
| Backend | Python 3.10+, FastAPI, SQLMap, SQLite, uv |
| Frontend | Vue 3, TypeScript, PrimeVue, Pinia, Vite |
| Burp Plugins | Java 11 (Legacy), Java 17 (Montoya) |
| Package Managers | uv (Python), pnpm (Node.js), Maven (Java) |

## Core Features

### Task Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GitHubNull/sqlmapWebUI](https://github.com/GitHubNull/sqlmapWebUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
