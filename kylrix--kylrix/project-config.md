---
trigger: always_on
description: 1. You are an autonomous software engineering agent tasked with maintaining the [Project Name] ecosystem.
---

# KylrixOrganization - Organizaion Local Agent Guide

# AGENTS.md - System Orchestration

## Core Operational Directives
1. You are an autonomous software engineering agent tasked with maintaining the [Project Name] ecosystem.
2. Your development workflow is strictly governed by financial and performance budgets detailed in `TOKENS.md`.

## Execution Lifecycle
*   **Phase 1 (Bootstrap):** On initialization, read `TOKENS.md` once to configure your output parser and tool-selection priority weights. For domain work, read `.agents/skills/SKILLS.md` first (catalog of every skill), then open only the matching skill — do not scan skills one-by-one.
*   **Phase 2 (Execution):** Maintain those constraints across all loop iterations. If your context window approaches 80% capacity, execute a self-directed context summary pass using the guidelines in `TOKENS.md`.

## 🏗️ Architectural Mandates

### 🚫 IMMUTABLE FILES & CLI SAFETY (STRICT)
- **NEVER Hand-Edit `appwrite.config.json`**: Hand-editing `appwrite.config.json` is strictly forbidden. It causes schema drift and catastrophic data loss.
- **NEVER Run `appwrite push`**: NEVER execute `appwrite push` or any of its push subcommands (`appwrite push tables`, `appwrite push all`, etc.). Pushing overwrites and wipes live databases and existing user records.
- **NEVER Run Ad-Hoc Node/TS Scripts Against Appwrite (STRICT)**: Never write or execute ad-hoc Node/TS scripts (`node -e`, `npx tsx`, inline scripts) passing Appwrite admin credentials to query or mutate backend state directly. Schema operations MUST strictly follow `.agents/skills/system.appwrite-cli-ops` via official Appwrite CLI commands (`appwrite tablesdb ...`). All agent and dogfooding interactions MUST go through the **Kylrix HTTP API (`/api/v1`)** using PATs/OAuth.
- **No internal APIs**: Prefer existing in-process functions, Server Actions, and SDK helpers for in-app flows to keep the open source productivity suite simple and consistent.
- **Prefer Internal Methods**: Use existing in-process functions, Server Actions, and SDK helpers instead of exposing new API surfaces.
- **Data Consolidation**: When returning shaped payloads to hydrate multiple UI widgets, use Server Actions or consolidated internal service methods.

### ✅ SOURCE CONTROL PERMISSIONS
- **Git Operations Permitted**: The agent is permitted and expected to perform Git operations. After implementing any fix or feature, the agent must consolidate the modifications, perform a commit with a descriptive message, and push the changes immediately. **Do not wait for the user to ask** — commit + push is part of finishing the task (see also `shipping-mode`).
- **Pure Commit Messages (STRICT)**: When committing, NEVER add any co-author metadata (e.g., `Co-authored-by:` headers, names, or emails). Commit messages must contain only the pure commit message description. Leave author identification entirely to the automatic system git configuration.

### ⚡ Development Standards
- **Canonical App**: Only implement against **`kylrix/`**. Legacy trees at repo root are for reference only.
- **Tailwind CSS**: Use Tailwind CSS and Vanilla CSS for maximum flexibility and modern looks according to openbricks design language. MUI and its co-dependencies are deprecated and must be removed.
- **Opaque Surfaces**: No gradients or translucent backgrounds on product chrome. Canonical UI rules: `.agents/skills/openbricks/SKILL.md`.
- **PNPM Only**: Always use `pnpm` for package management. NEVER use `npm` or `yarn`.
- **Global Unmount Policy**: Strictly use conditional rendering (`{isOpen && <Component />}`) for all overlays (drawers, modals, sidebars) instead of relying on visibility props. This physically removes the component and its invisible backdrops from the DOM when closed, mathematically preventing interaction blocking.
- **Interactivity Standards**: Use `keepMounted: false` and `disablePortal: true` for all OpenBricks drawers/modals to ensure they stay contained and cleanup correctly.
- **Surgical Execution**: For 'surgical fixes', prioritize direct, high-precision code modifications. Skip build/lint/test cycles unless explicitly instructed to validate. Aim for maximum velocity in resolving identified issues. Sometimes you only run `pnpm lint` or nothing at all (instead of running lint and build all the time), especially for minor edits, feature additions, or changes with low LOC and low chances of introducing new bugs.
- **Zero Speculation**: When the user identifies a specific error (ReferenceError, SyntaxError, etc.), fix exactly that error and stop. DO NOT check for similar errors in other files or attempt to 'proactively find' related issues. Resolve the reported problem surgically and get out of the way immediately.
- **Strict Scope Enforcement (STRICT)**: DO NOT edit, touch, clean up, refactor, or fix files that were not explicitly mentioned or directly affected by the user's explicit request. Strictly restrict all modifications to the exact target files requested. Unsolicited edits to adjacent or unrelated files are strictly prohibited.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kylrix/kylrix](https://github.com/Kylrix/kylrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
