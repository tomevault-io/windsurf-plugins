---
trigger: always_on
description: /incubate — Fully autonomous project incubation pipeline. Takes an idea through PRD → Architecture → Development → Testing → README → Website with persistent cross-session memory.
---


# /incubate — Autonomous Project Incubation Pipeline

## Overview

`/incubate <idea>` launches a fully autonomous 11-phase pipeline that takes any project idea from raw concept to a shipped GitHub repo with README and GitHub Pages website — **without requiring any user interaction during execution**.

Designed for solo developers who want to spin up side projects on weekends using compute power as a force multiplier. Run best-of-N instances in parallel and choose the best output.

## Activation

```
/incubate <idea>
/incubate <idea> --tag <tagname>
/resume <tagname>
```

Examples:
- `/incubate "CLI tool to batch-rename files with AI"`
- `/incubate "personal finance dashboard" --tag finance-dash`
- `/resume finance-dash`

## ⚠️ Autonomous Operation Declaration

**This skill operates in fully autonomous mode by design.**

- ❌ **Never** call `ask_user`, `AskQuestion`, `AskUserQuestion`, or any interactive prompt during Phases 1–11
- ✅ When ambiguous: interpret the idea three ways, choose the most actionable, document in Decision Log
- ✅ When blocked: apply the Error Recovery table, log as `mistake` in memory, continue
- ✅ Every significant decision is logged for best-of-N parallel-run comparison

The user is expected to be away. When they return, they review `STATUS.md` and `user-memory.jsonl`.

---

## Memory Protocol

### Storage Paths

| Platform | Memory Path |
|----------|-------------|
| GitHub Copilot (VS Code) | `~/.copilot/skills/incubate/data/user-memory.jsonl` |
| Cursor | `~/.cursor/skills/incubate/data/user-memory.jsonl` |
| Windows (Copilot) | `%USERPROFILE%\.copilot\skills\incubate\data\user-memory.jsonl` |
| Windows (Cursor) | `%USERPROFILE%\.cursor\skills\incubate\data\user-memory.jsonl` |

### Entry Schema

```json
{
  "id": "uuid-v4",
  "type": "preference | fact | pattern | mistake | decision",
  "entity": "user | project | tool",
  "fact": "human-readable fact statement",
  "confidence": 0.9,
  "source": "phase-N",
  "created_at": "2026-01-01T00:00:00Z",
  "status": "active"
}
```

### Memory Rules

- Load at Phase 0; apply `preference` and `pattern` entries throughout execution
- Save at Phase 11 (minimum 3 new entries)
- Archive when JSONL exceeds 200 entries (move oldest 100 to `*-archive.jsonl`)
- On corrupted file: rename to `.corrupted.<timestamp>`, create new file, log as `mistake`

### Platform Detection

| Signal | Platform | Repo Operations |
|--------|----------|-----------------|
| `github-create_repository` MCP tool available | Copilot (VS Code) | GitHub MCP tools |
| Tool unavailable + `git` in PATH | Cursor / Claude Code | git CLI |
| Neither available | Unknown | Local folder only |

---

## Phase 0: Memory Load & Setup

**Goal:** Initialize session state, load memory, detect platform, generate project slug.

### Steps

1. **Load Memory**
   - Check if `user-memory.jsonl` exists at the platform path
   - If exists: parse all `active` entries, build working context
   - If not exists: create directory structure, initialize empty file
   - If corrupted: rename to `<name>.corrupted.<timestamp>`, create fresh file

2. **Detect Platform**
   - Test if `github-create_repository` MCP tool is available → `PLATFORM=copilot`
   - Else test if `git --version` succeeds → `PLATFORM=cursor`
   - Else → `PLATFORM=local`
   - Log to `session.json`

3. **Generate Project Slug**
   - Normalize idea: lowercase, replace spaces with `-`, strip special chars, truncate to 40 chars
   - If `--tag` provided: use tag as slug override
   - Check if slug exists in memory (previous incubations):
     - If exists: append `-v2` (or `-v3` etc.) and continue
   - Store slug in session state

4. **Create Project Directory**
   - Local: `./incubations/<slug>/`
   - Initialize `session.json`: `{slug, idea, started_at, platform, phase: 0}`

**Exit Criteria:** Memory loaded, platform detected, slug set, directory created.

---

## Phase 1: PRD (Product Requirements Document)

**Goal:** Write a complete 8-section PRD without user input.

### PRD Template

```markdown
# PRD: <Project Name>

## 1. Problem Statement
What specific pain point does this solve? Who experiences it?

## 2. Target Users
Primary user persona(s). Be specific: "solo developer with 2–5 side projects."

## 3. Core Value Proposition
One sentence: "X helps Y do Z without W."

## 4. MVP Features
Bullet list of must-have features for v1. Max 7 items.

## 5. Non-Goals
Explicit exclusions to prevent scope creep.

## 6. Success Metrics
3–5 measurable outcomes. At least one quantitative metric.

## 7. Technical Constraints
Known constraints: language, platform, dependencies, auth requirements.

## 8. Autonomous Interpretation
(a) 3 possible interpretations of the idea
(b) Chosen interpretation
(c) Rationale for choice
```

### Steps

1. Analyze the idea using loaded memory (apply relevant `preference` and `pattern` entries)
2. Fill each section — minimum 2 sentences per section
3. For Section 8: always populate, even for clear ideas (document assumptions)
4. Save to `./incubations/<slug>/PRD.md`
5. Update `session.json` → `phase: 1, prd_complete: true`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codes1gn/incubate](https://github.com/codes1gn/incubate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
