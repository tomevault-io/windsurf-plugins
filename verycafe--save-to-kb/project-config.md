---
trigger: always_on
description: Personal knowledge base manager for LLM-Brain-Bases. Four modes: (1) AUTO-SAVE — automatically called at end of significant conversations to save reusable knowledge as Cards; (2) INIT — run "save-to-kb init" on first install to create directory structure and configure KB path; (3) CHECK — run "save-to-kb check" to scan the entire knowledge base for broken links, inconsistencies, and missing data, then fix with user confirmation; (4) REPLAY — run "save-to-kb replay" to retroactively process histo
---


# save-to-kb

## Route: Which Procedure to Run

**Read the invocation context first:**

- Invoked with argument `init`        → run **[INIT PROCEDURE]**
- Invoked with argument `check`       → run **[CHECK PROCEDURE]**
- Invoked with argument `replay`      → run **[REPLAY PROCEDURE]**
- Invoked with argument `save`        → run **[AUTO-INIT CHECK]** then **[SAVE PROCEDURE]**
- Invoked with argument `file <path>` → run **[FILE PROCEDURE]** with the given path
- Invoked without arguments          → run **[MENU]**

---

## [MENU]

Present a **two-level menu** using AskUserQuestion (max 4 options per question).

**Level 1** — main actions:
- save   — 从当前对话提取知识并保存到知识库
- file   — 分析指定文件并提取知识（PDF / MD / txt）
- check  — 健康检查，扫描断链和不一致
- 管理知识库… — 进入二级菜单（init / replay）

If user selects "管理知识库…", show **Level 2**:
- init   — 初始化知识库或更改路径
- replay — 回溯历史 session，补录遗漏的知识

Route based on final selection:
- save   → **[AUTO-INIT CHECK]** then **[SAVE PROCEDURE]**
- file   → **[FILE PROCEDURE]**
- check  → **[CHECK PROCEDURE]**
- init   → **[INIT PROCEDURE]**
- replay → **[REPLAY PROCEDURE]**

---

## [AUTO-INIT CHECK]

Before running the save procedure, verify the KB is initialized:

1. Check if `~/.claude/skills/save-to-kb/.config` exists and contains a path
2. If missing or empty → **run [INIT PROCEDURE] first**, then continue with save

---

## [INIT PROCEDURE]

Two scenarios: **first-time setup** and **change path**. Both follow the same steps but with different prompts and behaviors.

### Step I-1: Detect Scenario

Read `~/.claude/skills/save-to-kb/.config`:

- **File missing or empty** → **First-time setup**. Proceed to Step I-2.
- **File exists with a valid path** → **Already initialized**. Report:
  ```
  Knowledge base is configured at: {current-path}
  ```
  Ask: "Change to a new path? (yes/no)"
  - If no → stop.
  - If yes → this is a **path change**. Ask:
    ```
    Migrate existing knowledge base to the new location?
    - yes: copy all files (Bases/, Cards/, Wiki/, Claude.md) to new path. Old files are kept.
    - no:  create a new empty knowledge base at new path. Old files untouched.
    ```
    Record the user's choice as `{migrate: true/false}`. Proceed to Step I-2.

### Step I-2: Ask for Path

Ask the user:
> "Enter the full path for the knowledge base (e.g. ~/Documents/LLM-Brain-Bases or /Users/you/Obsidian/Work/LLM-Brain-Bases):"

Expand `~` to the actual absolute home directory path. Show the resolved path and ask: "Confirm: create knowledge base at {resolved-path}? (yes/no)"

If no → ask again for a new path.

### Step I-3: Create Directory Structure

Create the following directories (skip any that already exist):
```
{KB}/
{KB}/Bases/
{KB}/Cards/
{KB}/Wiki/
```

### Step I-3b: Migrate Files (path change + migrate: true only)

Skip this step for first-time setup or if user chose not to migrate.

Copy all contents from `{old-KB}/` to `{new-KB}/`:
- `{old-KB}/Bases/` → `{new-KB}/Bases/`
- `{old-KB}/Cards/` → `{new-KB}/Cards/`
- `{old-KB}/Wiki/`  → `{new-KB}/Wiki/`
- `{old-KB}/Claude.md` → `{new-KB}/Claude.md`

**Rules:**
- Copy, do not move. Old files remain untouched.
- Do not overwrite files that already exist at the new location.
- Wikilinks inside the files are relative (`[[Cards/slug]]`), so they require no modification.
- After copying, report: `Migrated {N} files to {new-KB}.`

### Step I-4: Write Master Index

**First-time setup**: Write `{KB}/Claude.md` only if it does not exist (never overwrite an existing index).
**Path change**: Write `{KB}/Claude.md` only if it does not exist at the new location.

```markdown
# LLM-Brain-Bases — Master Knowledge Index

*Auto-maintained by Claude. Last updated: {YYYY-MM-DD}.*

## How to Use This Index

Before starting any non-trivial task:
1. Check the **Scenario Quick-Lookup** table below for the task type
2. If found, read the linked Cards directly
3. If not found, identify the relevant category in **Knowledge Map** and open the Wiki file

To save new knowledge: invoke `save-to-kb` at conversation end.
To health-check the KB: invoke `save-to-kb check`.

---

## Knowledge Map

| Wiki | Cards | Focus | Key Tags |
|------|-------|-------|----------|
| [[Wiki/ai-tools]] | 0 | Claude Code, LLM tools, prompting, skills | claude, hooks, skills, llm |
| [[Wiki/workflow]] | 0 | Development process, tooling, automation | git, ci, planning |
| [[Wiki/debugging]] | 0 | Systematic diagnosis patterns | debugging, logging, tracing |
| [[Wiki/architecture]] | 0 | System design, decisions, tradeoffs | design, patterns, architecture |
| [[Wiki/knowledge-management]] | 0 | Knowledge base, Obsidian, PKM | obsidian, pkm |
| [[Wiki/frontend]] | 0 | Vue3, Nuxt, React, SSR, UI | vue3, nuxt, react, ssr |
| [[Wiki/backend]] | 0 | APIs, databases, server logic | postgres, redis, api |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verycafe/save-to-kb](https://github.com/verycafe/save-to-kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
