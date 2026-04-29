---
trigger: always_on
description: CodeDash (`codedash-app` on npm) is a zero-dependency Node.js browser dashboard for managing AI coding agent sessions. Supports 6 agents: Claude Code, Codex, Cursor, OpenCode, Kiro CLI, Kilo CLI. Single `npm i -g codedash-app && codedash run` opens a local web UI.
---

# CodeDash

## What is this

CodeDash (`codedash-app` on npm) is a zero-dependency Node.js browser dashboard for managing AI coding agent sessions. Supports 6 agents: Claude Code, Codex, Cursor, OpenCode, Kiro CLI, Kilo CLI. Single `npm i -g codedash-app && codedash run` opens a local web UI.

## Project structure

```
bin/cli.js              CLI entry point (run/list/stats/search/show/handoff/convert/export/import/update/restart/stop)
src/
  server.js             HTTP server + all API routes
  data.js               Session loading, search index, cost calculation, active detection for all 5 agents
  terminals.js          Terminal detection (iTerm2/Terminal.app/Warp/Kitty/cmux) + launch/focus
  html.js               Assembles HTML by inlining CSS+JS into template
  migrate.js            Export/import sessions as tar.gz
  convert.js            Cross-agent session conversion (Claude <-> Codex)
  handoff.js            Generate context documents for session handoff between agents
  changelog.js          In-app changelog data
  frontend/
    index.html          HTML template with {{STYLES}} and {{SCRIPT}} placeholders
    styles.css          All CSS including dark/light/monokai themes
    app.js              All frontend JavaScript (no build step, plain browser JS)
docs/
  README_RU.md          Russian translation
  README_ZH.md          Chinese translation
  ARCHITECTURE.md       Data flow, file formats, diagrams
```

## Supported agents and data sources

| Agent | Storage | Location | Format |
|-------|---------|----------|--------|
| Claude Code | JSONL | `~/.claude/projects/*/`, `~/.claude/history.jsonl` | `{type, message, timestamp}` |
| Codex CLI | JSONL | `~/.codex/sessions/`, `~/.codex/history.jsonl` | `{type: "response_item", payload}` |
| Cursor | JSONL | `~/.cursor/projects/*/agent-transcripts/` | `{role, message: {content}}` |
| OpenCode | SQLite | `~/.local/share/opencode/opencode.db` | tables: session, message, part |
| Kiro CLI | SQLite | `~/Library/Application Support/kiro-cli/data.sqlite3` | table: conversations_v2 |
| Kilo CLI | SQLite | `~/.local/share/kilo/kilo.db` | tables: session, message, part, project |

## Key architecture decisions

- **Zero dependencies** — only Node.js stdlib + system `sqlite3` CLI for SQLite agents
- **Node >= 18** — minimum supported version
- **Single process** — server + static HTML in one process
- **Template injection** — `html.js` reads CSS/JS files and injects via `split/join` (not `String.replace` which breaks on `$` characters in JS code)
- **Project key encoding** — Claude paths encoded as `path.replace(/[\/\.]/g, '-')` — both slashes AND dots replaced with dashes
- **Search index** — built in-memory on first query, cached 60 seconds. Do NOT remove the search index.
- **Cost calculation** — uses real `usage` data from assistant messages (input_tokens, output_tokens, cache_creation_input_tokens, cache_read_input_tokens) with per-model pricing in `MODEL_PRICING` object
- **Active session detection** — reads Claude PID files + scans `ps` for all agent processes (claude, codex, opencode, kiro-cli, cursor-agent)
- **Cursor sessions** — always show "Open in Cursor" button, never "Focus Terminal". Check tool type BEFORE active status.
- **cmux support** — walks parent process chain (up to 6 levels) to detect cmux, then activates via AppleScript

## API routes

```
GET  /                          Dashboard HTML
GET  /favicon.ico               SVG favicon
GET  /api/sessions              All sessions (all 5 agents)
GET  /api/session/:id           Full session messages
GET  /api/preview/:id           First N messages (lightweight)
GET  /api/replay/:id            Messages with timestamps for replay
GET  /api/cost/:id              Real cost from token usage
GET  /api/analytics/cost        Aggregated cost analytics (supports ?from=&to= date filters)
GET  /api/active                Running sessions (all agents — PID, CPU, memory, status)
GET  /api/terminals             Available terminal apps
GET  /api/git-commits           Git commits in time range
GET  /api/search?q=             Full-text search across all sessions
GET  /api/version               Current + latest npm version
GET  /api/changelog             Changelog data
GET  /api/handoff/:id           Generate handoff markdown document
POST /api/launch                Open session in terminal
POST /api/focus                 Focus terminal window by PID
POST /api/open-ide              Open project in Cursor/VS Code
POST /api/convert               Convert session between agents
POST /api/bulk-delete           Delete multiple sessions
DELETE /api/session/:id         Delete single session
GET  /api/session/:id/export    Download session as Markdown
```

## Important conventions

- Frontend JS is plain browser JavaScript — no modules, no build step, no ES6 imports
- CSS themes via `[data-theme="light"]` and `[data-theme="monokai"]` attribute overrides
- localStorage keys: `codedash-stars`, `codedash-tags`, `codedash-terminal`, `codedash-theme`, `codedash-layout`, `codedash-last-version`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vakovalskii/codbash](https://github.com/vakovalskii/codbash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
