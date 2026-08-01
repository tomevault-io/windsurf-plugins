---
trigger: always_on
description: MCP Server für persistentes Code-Indexing. Ermöglicht Claude Code schnelle, präzise Suchen statt Grep/Glob.
---

# AiDex - CLAUDE.md

MCP Server für persistentes Code-Indexing. Ermöglicht Claude Code schnelle, präzise Suchen statt Grep/Glob.

**Version:** 2.2.0 | **Sprachen:** 12 | **Repo:** https://github.com/CSCSoftware/AiDex

## Build & Run

```bash
npm install && npm run build    # Einmalig
npm run build                   # Nach Code-Änderungen
```

Registriert als MCP Server `aidex` (Prefix: `mcp__aidex__aidex_*`).

**Claude Code** (`~/.claude/settings.json`):
```json
"mcpServers": {
  "aidex": {
    "command": "node",
    "args": ["Q:/develop/Tools/AiDex/build/index.js"]
  }
}
```

**Claude Desktop** (`%APPDATA%/Claude/claude_desktop_config.json`):
```json
"mcpServers": {
  "aidex": {
    "command": "C:\\Program Files\\nodejs\\node.exe",
    "args": ["Q:\\develop\\Tools\\AiDex\\build\\index.js"]
  }
}
```

**Nach Änderungen:** Build ausführen, dann Claude Code neu starten.
**MCP-Name:** Server muss als `"aidex"` registriert sein → Prefix wird `mcp__aidex__aidex_*`.

## Tools (30)

### Suche & Index
| Tool | Beschreibung |
|------|--------------|
| `aidex_init` | Projekt indexieren |
| `aidex_query` | Terme suchen (exact/contains/starts_with), Zeit-Filter |
| `aidex_status` | Index-Statistiken |
| `aidex_update` | Einzelne Datei neu indexieren |
| `aidex_remove` | Datei aus Index entfernen |

### Signaturen (statt Read!)
| Tool | Beschreibung |
|------|--------------|
| `aidex_signature` | Datei-Signatur (Types + Methods) |
| `aidex_signatures` | Mehrere Dateien (Glob-Pattern) |

### Projekt-Übersicht
| Tool | Beschreibung |
|------|--------------|
| `aidex_summary` | Projekt-Übersicht mit Entry Points |
| `aidex_tree` | Dateibaum mit Stats |
| `aidex_describe` | Dokumentation zu summary.md |
| `aidex_files` | Projektdateien nach Typ, `modified_since` |

### Cross-Project
| Tool | Beschreibung |
|------|--------------|
| `aidex_link/unlink/links` | Dependencies verlinken |
| `aidex_scan` | Indexierte Projekte finden |

### Session (v1.2+)
| Tool | Beschreibung |
|------|--------------|
| `aidex_session` | Session starten, externe Änderungen erkennen |
| `aidex_note` | Session-Notizen (persistiert in DB) |
| `aidex_viewer` | Browser-Explorer mit Live-Reload (v1.3) |

### Task Backlog (v1.8+)
| Tool | Beschreibung |
|------|--------------|
| `aidex_task` | Task CRUD + Log + Scheduler (due/interval/action/auto_go) |
| `aidex_tasks` | Tasks auflisten, filtern nach Status/Priority/Tag. Zeigt due-Daten + Intervalle |

Status: `backlog → active → done | cancelled`

### Log Hub (v1.16+)
| Tool | Beschreibung |
|------|--------------|
| `aidex_log` | Universal-Logging: init/free/status/query/clear/write. HTTP-Server empfängt Logs von externen Programmen |

Actions: `init` (Server starten) → `query` (Logs abfragen) → `free` (Server stoppen)

### Screenshots (v1.9+, Optimierung v1.13)
| Tool | Beschreibung |
|------|--------------|
| `aidex_screenshot` | Screenshot aufnehmen + optional optimieren (`scale`, `colors`) |
| `aidex_windows` | Offene Fenster auflisten (Helper für window-Modus) |

### Global Search (v1.11+)
| Tool | Beschreibung |
|------|--------------|
| `aidex_global_init` | Verzeichnisbaum scannen, Projekte in `~/.aidex/global.db` registrieren. `index_unindexed`: Auto-Index ≤500 Dateien. `show_progress`: Browser Progress-UI |
| `aidex_global_status` | Alle registrierten Projekte mit Stats anzeigen |
| `aidex_global_query` | Terme über ALLE Projekte suchen (ATTACH DATABASE, 5-Min Cache) |
| `aidex_global_signatures` | Methoden/Typen nach Name über alle Projekte suchen |
| `aidex_global_refresh` | Stats aktualisieren, veraltete Projekte entfernen |

## Sprachen

C# · TypeScript · JavaScript · Rust · Python · C · C++ · Java · Go · PHP · Ruby · HCL/Terraform

## Architektur

```
src/
├── index.ts              # Entry Point (MCP + CLI)
├── server/
│   ├── mcp-server.ts     # MCP Protocol
│   └── tools.ts          # Tool-Handler
├── commands/             # Tool-Implementierungen
│   ├── init.ts, query.ts, signature.ts, update.ts
│   ├── summary.ts, link.ts, scan.ts, files.ts
│   ├── session.ts, note.ts, task.ts, log.ts
│   ├── screenshot/              # Plattform-Screenshots
│   └── global/                  # Global Search (v1.11)
│       ├── global-init.ts       # Scan + Bulk-Index
│       ├── global-query.ts      # ATTACH DATABASE Queries
│       ├── global-signatures.ts # Methoden/Typen suchen
│       ├── global-status.ts     # Projekt-Übersicht
│       └── global-refresh.ts    # Stats aktualisieren
├── loghub/                      # Log Hub (v1.16)
│   ├── log-types.ts       # Shared Types
│   ├── log-buffer.ts      # Ring Buffer (FIFO)
│   └── log-server.ts      # HTTP Server Singleton (Port 3335)
├── viewer/
│   ├── server.ts         # Interactive Viewer (Port 3333)
│   └── progress.ts       # SSE Progress UI (Port 3334)
├── db/
│   ├── database.ts       # SQLite (WAL)
│   ├── queries.ts        # Prepared Statements
│   ├── schema.sql        # Projekt-DB Schema
│   └── global-database.ts # ~/.aidex/global.db
└── parser/
    ├── tree-sitter.ts    # Parser (1MB Buffer)
    ├── extractor.ts      # Identifier + Signaturen
    └── languages/        # Keyword-Filter (12 Sprachen)
```

## Datenbank-Tabellen

| Tabelle | Inhalt |
|---------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CSCSoftware/AiDex](https://github.com/CSCSoftware/AiDex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
