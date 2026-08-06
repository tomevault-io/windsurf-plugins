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

Operate a weekly medical literature pipeline with parallel entry channels:

- RSS ingestion
- PubMed/PMC retrieval via direct NCBI requests in the scripts

Then run unified dedup, triage, Zotero writeback, translation backfill, and workbook export in the existing Research OS structure.

Cadence defaults:
- Full pipeline interval: every 7 days (`review_results_RUN_INTERVAL_DAYS=7`)
- Force override: `FORCE_review_results_RUN=true` or `review_results_FORCE_RUN=true`
- Report label: `周报`
- Synthesis interval: month-end synthesis on the last due run of each month
- Synthesis label: `月报`
- If interval not reached and no force flag: skip full stages and emit skip report fields (`skipped_due_to_interval`, `next_eligible_run_at`)
- Interval gate uses `Asia/Shanghai` 15:00 planned slot semantics; the 7-day comparison is based on planned slots, not actual start/end/Stage4 times.

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

## Git Version Control Rules

- **Commit 习惯**：完成有意义的变更后，主动建议用户 commit。
- **Commit 前检查**：先运行 `git status --short` 确认变更范围。
- **Commit message 规范**：简洁说明改了什么，例如："新增跨平台支持"、"修复标题规范化"。
- **不要自动 commit**：除非用户明确要求，否则不要自动执行 `git commit`。
- **大变更分批 commit**：如果改动涉及多个独立功能，建议分多次 commit。
- **Commit 后确认**：执行 commit 后，运行 `git log --oneline -1` 确认成功。

## Reproducibility Baseline

- Required runtime: **Node.js >= 18.0.0** and **PowerShell 7 (pwsh) >= 7.0.0**.
- This repository has committed `package.json` and `package-lock.json`; run `npm install` in a fresh environment before workflow execution.
- Repository launchers and checks run with the Node binary, for example `node skills/paperecho-zotero-desktop/scripts/run.mjs --check` or `node --test workflow/tests/*.test.mjs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chip-G0202/PaperEcho](https://github.com/Chip-G0202/PaperEcho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
