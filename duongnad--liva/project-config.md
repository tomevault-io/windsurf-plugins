---
trigger: always_on
description: All AI agents working on this codebase must adhere to the core principles below.
---

# LIVA System — Agent Guidelines

All AI agents working on this codebase must adhere to the core principles below.

## 🎯 Agent Persona & Git Safety
*(Merged from `AI_CONTEXT.md`, now archived at `docs/99-luu-tru/kien-truc-nodejs-v29/AI_CONTEXT.md`.)*

- **Role**: Act as a Principal Software Engineer and System Architect.
- **Mindset**: Security First → Performance → Clean Code.
- **[NO-YAPPING]**: Go straight to the point. Provide only the requested code or configurations — no apologies, no conversational filler.
- **[GIT-COMMIT-STYLE]**: Format all code-modification summaries as conventional Git commits (e.g. `feat(api): add feed endpoint`).
- **Strict Non-Assumption Protocol**: Stop after answering. Ask for permission ("Do you want me to implement this?") before writing code. Never perform background modifications based on implied requests. If you don't know, admit it and ask.
- **Git boundary ends at staging (`git add`)**: `git commit`, `git push`, `git pull`, `git fetch`, `git checkout -b`, `git merge`, and `git tag` are USER-only actions — never run them autonomously.

## 🧠 Single Source of Truth
Detailed system architectures, coding standards, memory systems, environment configurations, and anti-patterns have been migrated to the Obsidian Vault.
- **Vault Path**: `teamwork_projects/obsidian_llm_wiki/vault`
- **Rule**: You MUST use the `search_vault` tool to locate detailed guidelines for any task before implementing code modifications.

---

## 🧠 Rust Migration Plan (liva-native-core)
The LIVA system architectural migration from a hybrid Node.js/Python stack to the **Unified Native Engine** in Rust (`liva-native-core`) has been fully completed.
- **Current State**: The Node.js and Python codebases (`liva-gateway` and `liva-ai-engine`) have been fully migrated into a high-performance Unified Native Engine in Rust (`liva-native-core`). The Tauri IPC connects directly via Rust bindings.
- **Rule for Future Agents**: Since the migration of all core business logic and database WAL connection pools is complete, do not attempt to run, modify, or restore legacy Node.js/Python code. All backend changes, database connection pooling, semantic search, and AI router logic run natively in the Rust binary.
- **Migration Documentation**: Please refer to [`docs/99-luu-tru/ke-hoach-da-hoan-thanh/LIVA_NATIVE_MIGRATION_PLAN.md`](docs/99-luu-tru/ke-hoach-da-hoan-thanh/LIVA_NATIVE_MIGRATION_PLAN.md) for the final completed status of the Rust migration. *(It was archived out of the repo root on 2026-07-21; this pointer was left dangling until 2026-07-26. Being under `99-luu-tru/` it is a historical record — the plan is done, so read it for history, not as instructions.)*

---

## 🔍 GitNexus Code Intelligence Rules
This project is indexed by GitNexus as **LIVA**. Use GitNexus MCP tools for safe navigation.

### 1. Always Do
- **Run Impact Analysis**: Run `gitnexus_impact({target: "symbolName", direction: "upstream"})` before editing any symbol. Report direct callers, affected processes, and risk level.
- **Run Change Detection**: Run `gitnexus_detect_changes()` before staging/committing to verify only expected symbols and flows are affected.
- **Warn on High Risk**: Warn the user if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- **Query Concept**: Use `gitnexus_query({query: "concept"})` when exploring unfamiliar code instead of raw grepping.
- **Query Context**: Use `gitnexus_context({name: "symbolName"})` to get a 360-degree view of a symbol's callers, callees, and flows.

### 2. Never Do
- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — always use `gitnexus_rename` (which understands the call graph).
- NEVER commit or execute git remote operations autonomously (`git push`, `git pull`, `git commit`, etc.).

### 3. Keep Index Fresh
- Re-run `npx gitnexus analyze --embeddings` (or without `--embeddings` if not present) after changes to update the GitNexus code graph.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **LIVA**. Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> Index stale? Run `node .gitnexus/run.cjs analyze` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? `npx gitnexus analyze` (npm 11 crash → `npm i -g gitnexus`; #1939).

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user. For unified PDG impact, add `mode: "pdg"` with optional `line: <N>` — it returns statement-level `affectedStatements` over CDG + REACHING_DEF and inter-procedural symbols in `interproceduralByDepth`/`byDepth`; no-layer/degraded PDG results are UNKNOWN-risk notes (`--pdg` layer).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DuongNAD/LIVA](https://github.com/DuongNAD/LIVA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
