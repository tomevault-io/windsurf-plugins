---
trigger: always_on
description: These instructions are for AI coding agents (including GitHub Copilot) working in this repo.
---

# Copilot Instructions for maenifold

These instructions are for AI coding agents (including GitHub Copilot) working in this repo.

## 1. Always Re-read the Ground Truth
- Never work from memory; **re-open key docs on every session**:
  - `README.md` and `docs/README.md` for the big-picture architecture and cognitive stack.
  - `CONTRIBUTING.md`, `docs/WHAT_WE_DONT_DO.md`, `docs/TESTING_PHILOSOPHY.md`, `docs/SECURITY_PHILOSOPHY.md` for project philosophy.
- Treat these design principles as **inviolable, not bugs**:
  - **NO FAKE AI** – do not add retries, background “auto-fix”, or hidden fallbacks. Let errors propagate clearly.
  - **NO UNNECESSARY ABSTRACTIONS** – no DI containers, no interfaces for single implementations, no layering for its own sake.
  - **NO FAKE TESTS** – no mocks/stubs in new tests; always hit real SQLite and the real filesystem.
  - **NO FAKE SECURITY** – do not add path guards, quotas, or sandboxing; the only “security” is parameterized SQL.

## 2. Mental Model of the System
- This is a **.NET 9 C# MCP server + CLI** that turns markdown + WikiLinks into a **SQLite-backed knowledge graph**.
- Two entry modes (see `src/Program.cs`):
  - `maenifold --mcp` – stdio JSON-RPC server (tools annotated with `[McpServerTool]`).
  - `maenifold --tool <Name> --payload '<json>'` – CLI tool runner wired via `ToolRegistry`.
- Major namespaces (under `src/Tools/`):
  - `MemoryTools` – file CRUD and URI handling (`memory://...`).
  - `GraphTools` – SQLite graph + FTS5 sync/visualization.
  - `MemorySearchTools` – full-text, vector, and hybrid RRF search.
  - `SequentialThinkingTools` – long-running reasoning sessions.
  - `WorkflowTools` – YAML workflows + orchestration.
  - `MaintenanceTools` – concept repair and corruption analysis.
  - `SystemTools` – config, diagnostics, status.
- Core infrastructure lives in `src/Utils/` (config, markdown IO, vectors, `GraphDatabase`). **Do not introduce new infra layers; extend these directly.**

## 3. Project-specific Conventions
- **WikiLinks are the backbone**:
  - Always use `[[concept-name]]` (double brackets, normalized to lowercase-with-hyphens, usually singular).
  - Every new memory write/edit MUST include at least one WikiLink.
  - Never mechanically merge class names (e.g. `VectorTools`) with generic concepts (e.g. `[[tool]]`). Use `MaintenanceTools.AnalyzeConceptCorruption` first, then `RepairConcepts` with `dryRun: true`.
- **Memory URIs**:
  - Logical form: `memory://folder/subfolder/name` → `~/maenifold/memory/folder/subfolder/name.md`.
  - Keep URIs stable; when moving files, use the `MoveMemory` tool rather than ad‑hoc file moves.
- **File format**:
  - Always include YAML frontmatter with `checksum`. Use existing helpers instead of hand-writing checksums.
  - Content is plain markdown; keep it Obsidian-friendly.
- **Tools**:
  - Tools are **static methods** with `[McpServerTool]` attributes and are also registered in `ToolRegistry` with a `ToolDescriptor`.
  - When adding a tool, update both the partial class and `ToolRegistry`, and add a usage doc under `src/assets/usage/tools/`.

## 4. Working Effectively in This Codebase
- **Before changing behavior**, locate the existing pattern:
  - Search in `src/Tools/` for an analogous tool (e.g., how `SearchMemories`, `WriteMemory`, or `BuildContext` is implemented).
  - For graph/DB changes, read `src/Utils/GraphDatabase.cs` and follow the existing schema + trigger patterns.
- **When adding a feature**:
  - Prefer small, explicit methods in the relevant partial class instead of new layers.
  - Thread config via `Config` (in `src/Utils/Config.cs`); do not introduce new global state or hard-coded paths.
  - If the feature interacts with the graph or memory, ensure `Sync` behavior remains predictable and explicit.
- **When adding tests**:
  - Follow `tests/Maenifold.Tests/*` patterns, especially `MemoryToolsTests`.
  - Use `Config.TestMemoryPath` and real SQLite DBs; do not mock the DB or filesystem.
  - Use the `Run ma-core tests` VS Code task (Release, `--no-build`) as the default test workflow.

## 5. Integration & External Surfaces
- **MCP clients**: The public surface is tools exposed via MCP; keep tool signatures simple, JSON-friendly, and stable.
- **CLI**: Assume scripts and other agents call `--tool` with a JSON payload; avoid breaking payload shapes.
- **Docs as contract**:
  - `docs/integrations/claude-code/README.md` and other integration docs describe expected tool behavior—treat them as part of the API.
  - When you change a tool that appears in docs, update the relevant doc in the same PR.

## 6. Safe Change Patterns
- For **graph/knowledge changes**:
  - Run `Sync` tools as described in docs after structural changes.
  - Use `AnalyzeConceptCorruption` + `RepairConcepts` **with `dryRun: true` first**, and read the diff-like output.
- For **memory semantics** (write/move/edit):
  - Study the bug found in the demo around extension loss in move operations (`docs/demo-artifacts/part1-pm-lite/*`) and avoid re‑introducing similar issues.
  - Preserve checksums and frontmatter invariants when editing.

## 7. When in Doubt
- Re-read `CONTRIBUTING.md` and `docs/MA_MANIFESTO.md` to realign with the Ma (間) Protocol.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msbrettorg/maenifold](https://github.com/msbrettorg/maenifold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
