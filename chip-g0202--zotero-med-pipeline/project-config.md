---
trigger: always_on
description: - Default to completing tasks within the current repository/workspace.
---

# Medical Literature Workflow Agent Reference

## General Execution Environment And Permission Boundaries

- Default to completing tasks within the current repository/workspace.
- Recommended sandbox permission is Workspace write.
- Use Read only mode for auditing and planning work.
- Full access is a temporary exception only. Before requesting it, report exact paths, reason, scope of impact, risk, and rollback method, then wait for user approval.
- Keep project files, generated artifacts, temporary files, and test outputs inside the workspace.
- Prefer workspace runtime, project-local configuration, or standard user-level cache directories for Node, dependencies, and caches.
- Do not access, copy, index, or expose unrelated sensitive paths, including SSH private keys, browser data, Desktop, Downloads, personal Documents, or system configuration directories.
- Before network access, dependency installation, Zotero access, external API calls, global configuration changes, or reading/writing files outside workspace, report exact commands, necessity, impact scope, rollback availability, and whether a workspace-local alternative exists, then wait for approval.
- Before each major modification round, run `git status --short`; if the working tree is not clean, report existing changes first.
- Suggest creating a branch before large or high-risk modifications.
- Never reset, checkout, clean, delete, or overwrite existing user changes without explicit permission.
- If `node`/`npm`/`python` or other runtimes are not executable or lack permission, classify as an environment failure, not a business test failure.
- Prioritize diagnosing `PATH` and runtime paths, and prioritize workspace-provided or project-approved runtimes. Do not request Full access only because one path is not executable.

## Goal

Operate a two-day medical literature pipeline with parallel entry channels:

- RSS ingestion
- PubMed/PMC retrieval via `academic-search`

Then run unified dedup, triage, Zotero writeback, translation backfill, and workbook export in the existing Research OS structure.

Cadence defaults:
- Full pipeline interval: every 2 days (`RESEARCH_OS_RUN_INTERVAL_DAYS=2`)
- Force override: `FORCE_RESEARCH_OS_RUN=true` or `RESEARCH_OS_FORCE_RUN=true`
- Report label: `隔日报`
- Synthesis interval: 14 days (`RESEARCH_OS_SYNTHESIS_INTERVAL_DAYS=14`)
- Synthesis label: `双周报`
- If interval not reached and no force flag: skip full stages and emit skip report fields (`skipped_due_to_interval`, `next_eligible_run_at`)
- Interval gate uses `Asia/Shanghai` 15:00 planned slot semantics; the 48h comparison is based on planned slots, not actual start/end/Stage4 times.

## Capability Delegation and Existing Tool Boundary

- Prefer existing capabilities first. If a capability already exists in plugins, skills, MCP servers, existing adapters, CLI/scripts, library functions, or documented workflow contracts, reuse that capability instead of re-implementing it.
- Do not bypass abstraction layers. When an upper-layer interface already owns a capability, call that interface instead of jumping to lower-level internals.
- Do not duplicate internals of existing tools. If MCP already provides semantic search, do not independently build embedding generation, vector indexing, or vector similarity logic in pipeline business code.
- Before adding new code, determine capability ownership:
  - plugin
  - skill
  - MCP server
  - existing adapter
  - existing CLI/script
  - existing library function
  - documented workflow contract
- If ownership already exists, prefer:
  - thin adapter integration
  - explicit degrade path
  - auditable report fields
  - mock tests
- Only add new implementation when existing capability is unavailable or unverified. In that case:
  - explain why reuse is not possible
  - keep new code as a thin adapter
  - keep degradation + audit signals
  - avoid parallel competing implementations
- If interface name/shape is unknown:
  - do not fabricate APIs or success claims
  - do not bypass to lower-level services
  - degrade safely and report blocked/unverified state
- Rationale:
  - avoid duplicate implementations and config drift
  - avoid inconsistent results from double call chains
  - preserve plugin/MCP caching, indexing, permission and audit boundaries
  - keep module responsibilities clear
  - lower maintenance and debugging costs

## Reproducibility baseline (added)

- Required runtime: **Node.js >= 18.0.0** and **PowerShell 7 (pwsh) >= 7.0.0**.
- This repository currently has **no committed `package.json` and no npm/pnpm/yarn lockfile**.
- Local commands run directly with the Node binary, for example `node tools/run_zotero_literature_filter.mjs` or `node --test tests/*.test.mjs`.
- If local tests are runnable, the narrow default verification command is `node --test tests/*.test.mjs`.
- Because no package manager metadata is committed, reproducibility currently depends on the documented runtime versions, `.env.example`, and local Node-built-in test execution only.

## Hard Constraints

- PowerShell gate requires `pwsh >= 7.0.0` (exact pinning like `7.6.1` is not allowed unless explicitly required by a future task).
- Never use Windows PowerShell 5.1.
- Never directly edit `E:\zotero\zotero.sqlite`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chip-G0202/zotero-med-pipeline](https://github.com/Chip-G0202/zotero-med-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
