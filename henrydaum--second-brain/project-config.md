---
trigger: always_on
description: Local-first AI kernel with SQLite persistence, a REPL frontend, package
---

# Second Brain — Architecture Notes

Local-first AI kernel with SQLite persistence, a REPL frontend, package
install/uninstall, and live plugin loading. Python / SQLite. Solo dev (Henry).
The Flet GUI was removed; do not reintroduce.

---

# ⚡ THE KERNEL (READ FIRST)

Second Brain is a **microkernel**: a minimal, reliable core that boots, runs
the conversation loop + agent turn, persists conversations, and loads/unloads
plugins. Product capabilities arrive through a **package store** (the
agentskills.io model: a registry you browse, install, and uninstall from).
Do not bake heavy features into the kernel; they belong in packages.

> Goal in priority order: (1) the kernel works **flawlessly and reliably**, then
> (2) build install/uninstall against a cloud store, then (3) versioning and
> possibly containerization. We are at the end of step (1).

## What ships in the kernel (`plugins/`)

Plugins are discovered purely by file presence (`plugins/plugin_discovery.py`).
The kernel was produced by **moving** non-essential plugins into `store/` (a
staging catalog that mirrors `plugins/`, preserved via `git mv` to seed the
future store) — *not* by deleting them. What remains:

- **Services:** `service_llm`, `service_compactor` (context-safety),
  `service_parser` (text + image helper discovery), `service_timekeeper`
  (lightweight event clock), and `service_plugin_watcher` (hot-reload = the
  install/uninstall substrate). If another tracked service remains, treat it as
  kernel-boundary debt unless the user explicitly keeps it.
- **Tasks:** none.
- **Tools:** none in the tracked kernel tree. `tool_read_file`,
  `tool_ask_user_question`, shell/file-editing tools, SQL tools, and plugin
  authoring tools are package capabilities unless discovery shows they are
  installed.
- **Frontend:** `frontend_repl` only. Telegram and the MCP server
  (`frontend_mcp_server` — exposes Second Brain to external MCP clients over
  streamable HTTP; tested from main via `tests/test_frontend_mcp.py`, which
  loads it off the store ref) live on the store branch. `enabled_frontends`
  is deliberately not whitelisted by the kernel: config normalization keeps
  unknown names so installed store frontends survive load, and bootstrap
  warns/skips what discovery can't resolve.
- **Commands:** REPL UX + introspection only — `config`, `setup` (LLM onboarding
  wizard), `llm`, `conversations`, `clear`, `cancel`, `debug`, `frontends`,
  `locations`, `commands`, `tools`, `services`, `tasks`, `packages`.
  Profile/scheduling/MCP/update commands are package capabilities unless the
  tracked tree still carries a transitional command.

The pipeline substrate (`pipeline/` — orchestrator, watcher, event_trigger) still
boots, but ships **zero pipeline tasks**: it idles until a pipeline plugin
(extract/chunk/index/embed) is installed.

**Parsers.** The kernel keeps only the dependency-light `parse_text` parser
(UTF-8 / code / CSV / TSV, stdlib). Shared text helpers live in
`parsing_utils.py`. The registry (`parser_registry.py`) carries a static
native-modality default map so `get_modality` resolves image/audio/video even
with no parser installed (attachment routing relies on this). Every heavier
parser is an installable store package (`parser-pdf`, `parser-office`,
`parser-tabular`, `parser-image`, `parser-audio`, `parser-video`, `parser-gdoc`,
`parser-container`) that ships a `services/helpers/parse_*.py` file —
**not** a plugin entrypoint. `ParserService._load()` rebuilds the registry by
discovery-scanning `services/helpers/parse_*.py` across the built-in, sandbox,
and installed roots, so installed parsers light up on load; `package_manager`
reloads the parser service on install/uninstall of any such file so it takes
effect live. The attachment system is unified onto this one registry:
`attachments/parse.py` builds an `Attachment` via `parser.get_modality` +
`parser.parse(path, "text")` (no separate attachment-parser registry).

## The kernel boundary (the one rule)

Core code (`pipeline/`, `runtime/`, `state_machine/`, `agent/`, `events/`,
`config/`, `attachments/`, `main.pyw`) hard-imports **exactly two** plugin
modules. Keep these two resolvable in any kernel:
1. `service_llm` — `runtime/conversation_loop.py`.
2. `parser_registry` — `pipeline/orchestrator.py`, `pipeline/watcher.py`.

This rule is executable: `tests/test_kernel_boundary.py` AST-walks every core
module and pins the complete set of `plugins.*` import edges (the plugin
substrate plus the two implementations above, including lazy function-local
imports). Widening the boundary fails the suite until the test's allowlist —
and this section — are updated deliberately.

Everything else is discovery-based. The agent system prompt collects optional
guidance from each in-scope plugin's `agent_prompt_for(ctx)` (see `_collect` in
`agent/system_prompt.py`), so missing plugins degrade silently and correctly —
uninstalling a package removes its prompt text with it.

## Hardening applied for kernel reliability

These edits exist so the kernel degrades cleanly when a stdlib plugin is absent —
the difference between a microkernel and a pile of assumptions:
- **`plugins/services/service_compactor.py`** — context compaction is a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henrydaum/second-brain](https://github.com/henrydaum/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
