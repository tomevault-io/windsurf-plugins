---
trigger: always_on
description: This file provides guidance to Hermes Agent when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Hermes Agent when working with code in this repository.

## Project

Webnovel Writer for Hermes — a long-form Chinese web novel AI writing system built on the Hermes Agent framework. Combats AI "forgetting" and "hallucination" in serialized fiction through layered RAG, story contracts, and structured quality review.

Migrated from [webnovel-writer (Claude Code)](https://github.com/lingfengQAQ/webnovel-writer) and [webnovel-writer-opencode](https://github.com/lujih/webnovel-writer-opencode).

## Hermes Integration

### Skills (12 skills in `.hermes-skills/`)

Load with `skill_view(name='webnovel-xxx')`. **Auto-load rule**: scan user message for trigger keywords; if matched, load the skill before responding.

| Skill | 中文触发词 | Auto-load when user says... | What it does |
|-------|----------|---------------------------|-------------|
| `webnovel-init` | 初始化, 创建小说, 新建项目, 开始写新书 | "帮我初始化一个玄幻小说" | Deep project init: genre picker, MASTER_SETTING seed, outline scaffold |
| `webnovel-plan` | 规划, 大纲, 卷纲, 章纲 | "规划第3卷大纲" | Volume/chapter planning with genre pacing templates |
| `webnovel-write` | 写, 写章, 继续写, 写第, 撰写 | "帮我写第5章" | 6-step pipeline: context→draft→review→polish→commit→backup |
| `webnovel-write-batch` | 批量, 连写, 连更 | "连写第10到15章" | Batch writing with context isolation per chapter |
| `webnovel-review` | 审查, review, 审阅 | "审查前10章的一致性" | Post-hoc 6-dimension review (consistency/continuity/OOC/high-point/pacing/reader-pull) |
| `webnovel-rewrite` | 重写, 改, 修改 | "重写第5章的高潮部分" | Targeted chapter rewrite, preserving commit history |
| `webnovel-delete` | 删除, 删章 | "删除第3章" | Safe deletion with index/index cleanup |
| `webnovel-export` | 导出 | "导出为EPUB" | Export to MD/EPUB/HTML/DOCX |
| `webnovel-publish` | 发布 | "发布到番茄小说" | Platform publishing with format adaptation |
| `webnovel-query` | 查询, 状态, 进度 | "查询当前写作进度" | Project health: chapter count, debt status, word count trends |
| `webnovel-learn` | 拆书, 学习, 分析 | "拆解《诡秘之主》的节奏" | Reference novel deconstruction → idea bank |
| `webnovel-dashboard` | 面板, dashboard | "打开写作面板" | Launch FastAPI+React visualization dashboard on port 8888 |

### Agents (5 agents in `agents/`)

Original OpenCode used `Agent()` subagent calls. In Hermes, use `delegate_task`:

| Agent | Hermes usage |
|-------|-------------|
| `context-agent` | `delegate_task(goal="Generate writing brief for chapter N", context="...")` |
| `data-agent` | `delegate_task(goal="Extract facts from chapter N", context="...")` |
| `reviewer` | `delegate_task(goal="Review chapter N", context="...")` × 6 parallel |
| `chapter-writer-agent` | `delegate_task(goal="Draft chapter N", context="...")` |
| `deconstruction-agent` | `delegate_task(goal="Deconstruct reference novel", context="...")` |

### Skill Loading Pattern

When the user says "写第5章":
1. Load the skill: `skill_view(name='webnovel-write')`
2. Follow the skill's 6-step pipeline
3. Call agents via `delegate_task` where the skill says to use Agent()

## Commands

### Testing

```bash
# Full test suite (from repo root)
python -m pytest scripts/data_modules/tests -q --no-cov

# Single test file
python -m pytest scripts/data_modules/tests/test_config.py -q --no-cov

# Single test function
python -m pytest scripts/data_modules/tests/test_config.py::test_load_env -q --no-cov
```

Tests live in `scripts/data_modules/tests/` (60 test files).

### CLI

```bash
# Unified entry point for all commands
python scripts/webnovel.py <command> [args]

# Common subcommands
python scripts/webnovel.py preflight       # validate runtime environment
python scripts/webnovel.py status          # project health report
python scripts/webnovel.py story-system    # story contract management
python scripts/webnovel.py review-pipeline # review pipeline management
python scripts/webnovel.py export          # export novel
python scripts/webnovel.py publish         # publish to platform
python scripts/webnovel.py memory          # memory system management
```

Full command list (28 commands): `where`, `preflight`, `use`, `index`, `state`, `rag`, `style`, `entity`, `context`, `memory`, `migrate`, `status`, `update-state`, `backup`, `archive`, `init`, `extract-context`, `story-system`, `story-events`, `chapter-commit`, `memory-contract`, `project-memory`, `review-pipeline`, `placeholder-scan`, `master-outline-sync`, `export`, `publish`, `knowledge`.

Most subcommands forward to `data_modules/<module>.py` via argparse dispatch. The entry point auto-resolves the book project root (directory containing `.webnovel/state.json`).

### Dashboard

```bash
# Backend (FastAPI on port 8888)
python -m dashboard

# Frontend dev server (React + Vite, separate terminal)
cd dashboard/frontend && npm run dev
```

## Architecture

### Six-Layer Data Flow

Code is organized as a pipeline — each layer feeds the next:

| Layer | What | Where |
|-------|------|-------|
| Knowledge | CSV tables + MD references + BM25 retrieval | `references/` |
| Reasoning | Genre routing + anti-pattern ranking | `genres/` |
| Contract | MASTER_SETTING + volume/chapter briefs + review contracts | `.story-system/` (per-project) |
| Context | JSON assembly of what the writer needs | `scripts/data_modules/context_manager.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starMagic/webnovel-writer-hermes](https://github.com/starMagic/webnovel-writer-hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
