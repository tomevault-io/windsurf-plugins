---
trigger: always_on
description: Instructions for Claude Code when working on this project.
---

# CLAUDE.md

Instructions for Claude Code when working on this project.

## Project Overview

FileEngine is a Go + Vue 3 monorepo. The Go backend uses Gin (HTTP), GORM (ORM), and Cloudwego Eino (AI agent framework). The Vue frontend is embedded into the Go binary via `go:embed`.

## Build Commands

```bash
# Frontend build (must run first, output goes to web/dist/ which is embedded)
cd web && npm run build

# Backend build (includes embedded frontend)
go build -o fileengine .

# Full build
cd web && npm run build && cd .. && go build -o fileengine .

# Run
./fileengine              # uses config.yaml in CWD
./fileengine path/to/config.yaml
```

## Code Style & Conventions

### Design Principles
- **No backward compatibility** — Do not add legacy fallbacks, compatibility shims, or migration paths. Old data/config formats are not supported. If a schema changes, the old version is simply dropped.
- **No over-engineering** — Don't add feature flags, version negotiation, or graceful degradation for removed features. If something is removed, delete it completely with no trace.
- **Single source of truth** — Each piece of data lives in one place. Filesystem connections are in the `Filesystem` DB table. Model providers are in the `ModelProvider` DB table. Categories are scoped to filesystems. Per-session agent config (allow_read_file, allow_auto_category, model_provider_id) lives on `ScanSession`.

### Go
- Module path: `FileEngine` (not a URL-style path)
- All internal packages under `internal/`
- API handlers: one file per resource (`handler_session.go`, `handler_file.go`, `handler_model.go`, etc.)
- Config: global singleton with `config.Get()` / `config.Update()` / `config.Save()`
- DB: repository pattern via `db.Repository` wrapping `*gorm.DB`; `db.WithRetry()` for SQLite lock retries
- GORM models use struct tags for composite indexes (e.g., `gorm:"index:idx_name,priority:N"`)
- Agent tools use Eino's `utils.InferTool` with struct tags for JSON schema generation
- Error handling: return errors up, log at the handler level (including background goroutines)
- No CLI framework — pure HTTP server with embedded frontend

### Vue / TypeScript
- Vue 3 Composition API (`<script setup lang="ts">`)
- Element Plus UI framework
- Hash-based routing (`createWebHashHistory`)
- i18n: Chinese (zh-CN) as default, English (en) as fallback
- Locale files: `web/src/i18n/locales/{zh-CN,en}.yaml`
- API client: Axios wrapper at `web/src/api/index.ts`
- Types: `web/src/types/index.ts`
- All icons globally registered from `@element-plus/icons-vue`
- CodeMirror for prompt editing (One Dark theme + markdown)
- Colored tags for protocols (SMB/SFTP/FTP/NFS/LOCAL), file types (文件夹/文件), providers (OpenAI/Claude/Ollama), session status (i18n)
- localStorage for user preferences: `fe_last_fs_id`, `fe_log_order`, `fe_columns`

## Architecture Notes

### Two-Phase Design
- **Phase 1 (DB-only):** Agent scans, tags, and sets target paths — only modifies database records
- **Phase 2 (Execution):** Executor reads plans from DB, user chooses copy or move mode, performs actual file operations via RemoteFS

### Scanner
- Recursive directory walker, creates `FileEntry` records for all files/directories
- Supports blacklist/whitelist directory filtering (`ScanSession.FilterMode` + `FilterDirs`)
- `ExcludeCategoryDirs` option auto-appends category paths to blacklist
- Filter dirs stored as newline-separated text, parsed at scan time

### Agent Tagging Algorithm
- Processes directories **bottom-up by depth level** (deepest first)
- Each directory gets its own agent call (not batched)
- Agent explores upward to find project roots, marks entire project at once
- `mark_tagged` cascades to all descendants via `LIKE` query; also writes `batch_index`
- `set_target` on a directory cascades: clears all children's targets (outer overrides inner)
- `set_target` with empty `new_path` clears the target
- Before processing each directory, re-checks tagged status (another worker may have cascade-tagged it)
- Configurable concurrency: each worker gets its own FS connection + ReAct agent instance + token tracker
- All agent config (concurrency, batch_size, max_retries, system_prompt) read from live `config.Get()` each iteration — hot-reloadable without restart
- Retry on failure: up to `max_retries` with linear backoff (2s, 4s, 6s), 60s timeout per Generate call
- `processRemainingFiles` has 1000-iteration safety limit to prevent infinite loops

### Agent Tools
- 8 always-on tools: `list_files`, `get_file_info`, `update_description`, `mark_tagged`, `list_categories`, `list_category_files`, `set_target`, `update_category`, `delete_category`
- 2 conditional tools: `read_file` (when `allow_read_file` is true), `create_category` (when `allow_auto_category` is true)
- Tool availability is per-session, configured via `ScanSession.AllowReadFile` / `AllowAutoCategory`
- All tool path inputs are normalized via `normalizePath()` (strips leading `/`)
- `delete_category` clears planned files under that category and resets `tagged=false` for re-classification
- Categories have `agent_created` and `agent_editable` flags; agent can only modify/delete `agent_editable=true` categories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangzheng2016/FileEngine](https://github.com/huangzheng2016/FileEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
