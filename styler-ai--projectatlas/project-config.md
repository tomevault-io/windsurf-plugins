---
trigger: always_on
description: 0. If ProjectAtlas MCP tools are available, use `atlas_*` tools for normal ProjectAtlas command families before shelling out. Expected parity tools include `atlas_init`, `atlas_config`, `atlas_root`/`atlas_root_set`, `atlas_ignore_list`/`atlas_ignore_init_gitignore`/`atlas_ignore_add`/`atlas_ignore_remove`, `atlas_lint`, `atlas_runtime_info`, `atlas_mcp_config`, `atlas_session_brief`, `atlas_task_status`/`atlas_task_cancel`, and `atlas_map`, plus the existing scan, overview, folder, file, summar
---

# ProjectAtlas Startup Snippet

## Startup
0. If ProjectAtlas MCP tools are available, use `atlas_*` tools for normal ProjectAtlas command families before shelling out. Expected parity tools include `atlas_init`, `atlas_config`, `atlas_root`/`atlas_root_set`, `atlas_ignore_list`/`atlas_ignore_init_gitignore`/`atlas_ignore_add`/`atlas_ignore_remove`, `atlas_lint`, `atlas_runtime_info`, `atlas_mcp_config`, `atlas_session_brief`, `atlas_task_status`/`atlas_task_cancel`, and `atlas_map`, plus the existing scan, overview, folder, file, summary, search, slice, health, purpose, token, settings, and watcher-status tools. Use the CLI for plugin install/update/release/CI workflows, MCP server startup/debugging, continuous `watch`, terminal TUI views, or when an MCP tool is unavailable.
1. Establish the project root and run ProjectAtlas from that root so `.projectatlas/projectatlas.db` is project-local.
2. Refresh with `atlas_watch_once`, `atlas_scan`, `projectatlas watch --once`, or `projectatlas scan` only when the index may be stale; do not scan merely because a session started.
3. For task-directed MCP work, call `atlas_session_brief` once with the task query, `project_path` when needed, and `compact: true`. Follow its typed next call directly and do not repeat folder/file discovery.
4. Use the returned compact summary, search, relation, health, or slice call. Copy returned selectors and continuations instead of guessing or restarting discovery.
5. Use compact summary connections for an ordinary trusted direct caller or dependency. Call detailed relations only when resolution, completeness, ambiguity, an omitted connection, or an exact occurrence matters.
6. Run `atlas_slice` or `projectatlas slice <file> --start-line <n> --end-line <m>` for the smallest exact source range that answers the task.
7. Fall back to `atlas_overview` or `projectatlas overview`, then folders and files, only when the brief is unavailable, has no actionable candidate, or broader repository structure is itself the task. Use `projectatlas files --file-pattern <glob>` when the path pattern is already known.
8. Run `atlas_outline`, `atlas_search`, or their CLI equivalents when selected-file context is insufficient.
9. Run `atlas_health` or `projectatlas health-check` for cleanup/refactor work.
10. Run `atlas_lint` or `projectatlas lint --report-untracked --purpose-level low`; low fails stale, duplicate, and temporary-folder health but keeps first-pass purpose curation advisory. Use `atlas_purpose_queue` or `projectatlas purpose queue` for the next curation actions, `--purpose-level medium` when all source files must be agent-reviewed, and `--purpose-level strict` only when every indexed file and folder must be agent-reviewed.
11. Only then use language-server lookups or broad file reads on selected files.
13. Run `atlas_token_report` or `projectatlas token` when asked for token savings; use `projectatlas token --view tui` only when a human asks for the terminal dashboard.
14. After ProjectAtlas plugin/runtime updates, verify Codex plugin drift with `codex plugin list --marketplace projectatlas --json` and global MCP registry drift with `codex mcp get projectatlas` or `codex mcp list` when `codex` is available. A stale official `projectatlas` marketplace/plugin cache, stale Codex ProjectAtlas skill artifact, stale global `projectatlas` MCP entry pointing at an old runtime, wrong version, or invalid config, or old ProjectAtlas release pin in an official downstream workflow, is a bug; rerun the ProjectAtlas installer so it repairs or reports the drift automatically. The installer also verifies Claude Code/OpenCode generated MCP configs against the verified runtime, version guard, selected DB/config, and host-specific fields; when those host CLIs are installed, smoke their config readers too. A global entry whose default DB belongs to another repository can still serve the current repository through process-scoped `atlas_set_project_path` or per-call `project_path`; prefer per-call `project_path` for shared or concurrent hosts. Use `PROJECTATLAS_SKIP_CODEX_PLUGIN_UPDATE=1` only for intentionally managed Codex ProjectAtlas plugin marketplaces, and `PROJECTATLAS_SKIP_CODEX_MCP_REGISTRY_UPDATE=1` only for intentionally managed global registries.
15. If work moves outside the selected ProjectAtlas root, switch with `atlas_set_project_path` in a single-client stdio session or per-call `project_path` when isolation matters. Root-level MCP paths may route to another repository only when that addressed root already has `.projectatlas/projectatlas.db`; otherwise use normal filesystem tools such as `rg`, `Get-Content`, or targeted shell reads instead of ProjectAtlas for out-of-project files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [styler-ai/ProjectAtlas](https://github.com/styler-ai/ProjectAtlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
