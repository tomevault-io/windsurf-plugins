---
trigger: always_on
description: Novel-production operating system skill for long-form fiction, web novels, serials, fanfiction, and side stories. Use when designing or running a structured writing pipeline with persistent world state, chapter summaries, hooks, character matrices, continuity audits, rewrite/revise loops, style guides, or per-book rules. Best for requests like building an InkOS-inspired novel workflow skill, creating a long-novel workflow, keeping multi-chapter story consistency, generating/auditing/revising cha
---


# NovelOps Skill

Version: 1.0.0

Build and run long-form fiction as a stateful pipeline, not a one-shot prompt.

Treat this skill as a **novel OS skeleton**: it provides project structure, workflow, audit criteria, and helper scripts for an OpenClaw-based writing system that is conceptually close to InkOS.

## Core operating model / 核心运行模型

Use this loop:

1. **Load truth files**
2. **Plan the next chapter**
3. **Draft with constraints**
4. **Run knowledge-boundary checks when the chapter depends on hidden truths or limited POV**
5. **Audit for continuity / style / pacing / leaks**
6. **Revise with spot fixes first**
7. **Extract candidate state updates from the accepted draft**
8. **Update story state**
9. **Queue unresolved issues for human review**

Prefer stable files over fragile memory. If information matters in later chapters, write it into a truth file.

## Project layout / 项目结构

When starting a new project, copy the template from `assets/project-template/`.

Expected files:

- `story_bible.md` — world rules, premise, factions, locations, power system
- `book_rules.md` — per-book rules, prohibitions, protagonist locks, tone limits
- `outline.md` — macro arc, chapter targets, payoff plan
- `current_state.md` — authoritative latest world state
- `chapter_summaries.md` — per-chapter summaries and state deltas
- `pending_hooks.md` — open promises, foreshadowing, unresolved conflicts
- `character_matrix.md` — who met whom, trust/conflict, information boundaries
- `character_knowledge.md` — optional structured knowledge boundary tracker for major characters
- `emotional_arcs.md` — tracked emotional movement by key character
- `subplot_board.md` — A/B/C line progress and stagnation notes
- `continuity_issues.md` — known inconsistencies or manual review backlog
- `style_guide.md` — qualitative style guide
- `style_profile.json` — optional quantitative style stats
- `chapters/` — chapter markdown files
- `reviews/` — audit and revision reports

`current_state.md` is the most important operational file. Keep it compact, current, and explicit.

## Recommended CLI workflow / 推荐 CLI 工作流

默认优先使用统一入口：

```bash
python scripts/novelops_cli.py <command> ...
```

这样更适合日常使用、文档引用和后续自动化；底层脚本仍可直接调用，但建议视为 advanced / lower-level usage。

### Quick start / 快速开始

#### 1) Initialize a project / 初始化项目

推荐：

```bash
python scripts/novelops_cli.py init /path/to/project "Book Title"
```

默认不会初始化到非空目录，避免覆盖已有小说项目；如果你确认要覆盖模板文件，可显式追加 `--force`。

Lower-level usage / 底层脚本方式：

```bash
bash scripts/init_novel_project.sh /path/to/project "Book Title"
```

This copies the template and creates the standard directory layout.
For cross-platform usage, prefer `python scripts/novelops_cli.py init ...`.

#### 2) Build next-chapter context / 构建下一章上下文

推荐：

```bash
python scripts/novelops_cli.py context \
  --project /path/to/project \
  --recent-chapters 3 \
  --json
```

Lower-level usage / 底层脚本方式：

```bash
python scripts/build_next_chapter_context.py \
  --project /path/to/project \
  --recent-chapters 3 \
  --json
```

#### 3) Build write-next packet / 构建下一章工作包

推荐：

```bash
python scripts/novelops_cli.py write-next \
  --project /path/to/project \
  --json
```

Lower-level usage / 底层脚本方式：

```bash
python scripts/build_write_next_packet.py \
  --project /path/to/project \
  --json
```

Use this when you want a more structured handoff than raw context: chapter goal, active hooks, constraints, state targets, suggested scene beats, and a single-chapter output contract.

## Workflow / 工作流

### 1) Before writing a chapter / 写章前准备

Read at least:

- `story_bible.md`
- `book_rules.md`
- `outline.md`
- `current_state.md`
- latest section of `chapter_summaries.md`
- `pending_hooks.md`
- `character_matrix.md`

If maintained, also read:

- `character_knowledge.md`

If the request is for a side story, prequel, sequel, or alternate timeline, also establish:

- parent canon constraints
- divergence point
- what characters do **not** know yet
- which original hooks must remain untouched

### 2) Planning rules / 规划规则

Before drafting, explicitly decide:

- chapter purpose
- POV
- conflict driver
- payoff or partial payoff
- hooks to advance, delay, or close
- state changes that must occur
- constraints that must not be violated

Write a short chapter plan into the response or a scratch file if useful.

### 3) Drafting rules / 起草规则

Draft from observed reality, not abstract explanation.

Prefer:

- concrete action
- sensory evidence
- state changes that can be tracked later
- character knowledge limited to what they have seen, inferred, or been told
- exactly one target chapter per draft

Avoid:

- breaking protagonist personality lock
- introducing untracked items/powers/injuries
- resolving major hooks accidentally
- report-speak in narrative prose
- broad whole-crowd reactions unless earned

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qiyan233/novelops-skill](https://github.com/qiyan233/novelops-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
