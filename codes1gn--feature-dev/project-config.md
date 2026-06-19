---
trigger: always_on
description: /feature-dev — Autonomous feature development pipeline with TDD-first workflow. Takes a feature description in an existing project and drives it through Spec → Tests → Implementation → Review → PR without user interaction. Supports GitHub Copilot (VS Code) and Cursor.
---


# /feature-dev — Autonomous Feature Development Pipeline

## Overview

`/feature-dev <feature description>` launches a fully autonomous 6-phase pipeline that takes any feature request in an existing project from description to merged PR — **without requiring any user interaction during execution**.

Key differentiator: **TDD-first** — failing tests are written before a single line of implementation code. The pipeline never ships code that doesn't pass its own tests.

Designed to run as one of many parallel agents. The user can spin up best-of-N instances and pick the best PR.

## Activation

```
/feature-dev <feature description>
/feature-dev <feature description> --branch <branch-name>
/feature-dev <feature description> --tag <tagname>
/feature-dev <feature description> --base <base-branch>
```

Examples:
- `/feature-dev "add dark mode toggle to settings page"`
- `/feature-dev "rate limit API endpoint" --tag rate-limit-v1`
- `/feature-dev "export data as CSV" --branch feat/csv-export --base develop`

## ⚠️ Autonomous Operation Declaration

**This skill operates in fully autonomous mode by design.**

- ❌ **Never** call `ask_user`, `AskQuestion`, `AskUserQuestion`, or any interactive prompt during Phases 0–6
- ✅ When ambiguous: document 2–3 interpretations, choose the most aligned with existing codebase patterns
- ✅ When blocked by a missing dependency: document in spec, stub it, continue
- ✅ When tests fail after implementation: retry up to 3 times, log as `mistake` in memory if unresolved
- ✅ Every significant decision is logged in `FEATURE_LOG.md` for best-of-N run comparison

The user is expected to be away. When they return, they review the PR description and `FEATURE_LOG.md`.

---

## Memory Protocol

### Shared Memory with /incubate

`/feature-dev` shares the same `user-memory.jsonl` as `/incubate`. This means:
- User preferences learned during incubation apply to feature dev
- Feature development insights are visible to future incubation runs

### Storage Paths

| Platform | Memory Path |
|----------|-------------|
| GitHub Copilot (VS Code) | `~/.copilot/skills/incubate/data/user-memory.jsonl` |
| Cursor | `~/.cursor/skills/incubate/data/user-memory.jsonl` |
| Windows (Copilot) | `%USERPROFILE%\.copilot\skills\incubate\data\user-memory.jsonl` |
| Windows (Cursor) | `%USERPROFILE%\.cursor\skills\incubate\data\user-memory.jsonl` |

### Memory Rules

- Load at Phase 0; apply `preference` and `pattern` entries throughout execution
- Save at Phase 6 (minimum 3 new entries of type `feature`)
- Entry types added by this skill: `feature` (new type alongside existing types)
- Archive when JSONL exceeds 200 entries (move oldest 100 to `*-archive.jsonl`)

### Platform Detection

| Signal | Platform | Repo Operations |
|--------|----------|-----------------|
| `github-create_branch` MCP tool available | Copilot (VS Code) | GitHub MCP tools |
| Tool unavailable + `git` in PATH | Cursor / Claude Code | git CLI |
| Neither available | Unknown | Local folder only |

---

## Phase 0: Setup

**Goal:** Parse feature request, load memory, detect platform, explore codebase, create feature branch.

### Steps

1. **Parse Invocation**
   - Extract feature description from invocation
   - Generate feature slug: lowercase, replace spaces with `-`, strip special chars, truncate to 40 chars
   - If `--branch` provided: use as branch name override
   - If `--tag` provided: use as slug override
   - If `--base` provided: use as base branch; otherwise detect default branch (`main`, `master`, or `develop`)
   - If slug already exists as a branch: append `-v2` (or `-v3` etc.) and continue

2. **Load Memory**
   - Check if `user-memory.jsonl` exists at the platform path
   - If exists: parse all `active` entries, build working context
   - If not exists: create directory structure, initialize empty file
   - If corrupted: rename to `<name>.corrupted.<timestamp>`, create fresh file

3. **Detect Platform**
   - Test if `github-create_branch` MCP tool is available → `PLATFORM=copilot`
   - Else test if `git --version` succeeds → `PLATFORM=cursor`
   - Else → `PLATFORM=local`
   - Log to `FEATURE_LOG.md`

4. **Explore Codebase**
   - List top-level directory structure
   - Identify: primary language, test framework, test directory location, code style
   - Apply memory `preference` entries (e.g., preferred language, test conventions)

5. **Create Feature Branch**
   - **Copilot path:** `github-create_branch(branch="feat/<slug>", from_branch=<base>)`
   - **Cursor/git path:** `git checkout -b feat/<slug> <base>`
   - Log branch name to `FEATURE_LOG.md`

6. **Initialize FEATURE_LOG.md**
   ```markdown
   # Feature Log: <slug>
   
   - Feature: <description>
   - Branch: feat/<slug>
   - Base: <base-branch>
   - Platform: <copilot|cursor|local>
   - Started: <ISO timestamp>
   - Phase: 0
   ```

**Exit Criteria:** Memory loaded, platform detected, feature branch created, FEATURE_LOG.md initialized.

---

## Phase 1: Feature Spec

**Goal:** Write a tight 5-section feature specification without user input.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codes1gn/feature-dev](https://github.com/codes1gn/feature-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
