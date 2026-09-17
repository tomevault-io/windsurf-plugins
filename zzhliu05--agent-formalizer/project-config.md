---
trigger: always_on
description: This project is a lightweight Obsidian-first research vault for 构建将数学材料转化为可教学、可追踪、可由 Lean 检验内容的完整多 Agent 形式化流程.
---

# AGENTS.md

# 多 Agent 形式化数学教材 Research Vault Instructions

## Purpose

This project is a lightweight Obsidian-first research vault for 构建将数学材料转化为可教学、可追踪、可由 Lean 检验内容的完整多 Agent 形式化流程.

Domain: 形式化数学、多智能体协作、自动定理证明与教材工程

User background and preferred stance: 面向具备大学数学或软件工程基础、但不要求已有 Lean 经验的数学与工程协作者

Vault balance: 工程与代码优先，同时保留研究和写作区域

Explanation style: 数学直觉 → 明确规格 → Agent 输入输出 → Lean 验证

The vault is meant to accumulate a structured network of knowledge over time while remaining useful for research notes, derivations, source reading, code experiments, outputs, figures, and technical writing.

## First Rule: Decide Whether Vault Context Matters

Before answering any user prompt in this project, classify the request:

1. `standalone direct answer`: answer from current context and general knowledge.
2. `project-continuity task`: read vault context before answering.
3. `durable vault update`: read vault context, make the requested change, and update the process log.

Default to direct answers for isolated questions. Use vault context when continuity matters.

Use vault context when the request:

- references this project's prior work, files, sources, decisions, or ongoing research direction;
- asks to continue, synthesize, summarize, or revise an existing process;
- references a known topic, source, note, wiki page, code artifact, output, or figure;
- asks what has been done, decided, read, derived, or implemented;
- asks to add, reorganize, or update durable project knowledge.

For project-continuity tasks, read in this order:

1. `AGENTS.md`;
2. the newest relevant entries of `wiki/process-log.md`;
3. `wiki/index.md`;
4. linked wiki, note, raw, code, output, or figure pages as needed.

Do not read the whole process log by default. It is newest-first; start at the top and follow links outward only when needed.

## Hook-Ready Reminder

The project-local `UserPromptSubmit` hook should inject this reminder:

```text
Before answering in this project:
1. Classify the prompt: standalone direct answer, project-continuity task, or durable vault update.
2. If project continuity matters, read newest relevant entries of [[wiki/process-log]] and [[wiki/index]] before answering.
3. If durable knowledge changes, update the relevant wiki/note pages and prepend a compact entry to [[wiki/process-log]].
4. Prefer links/backlinks over long summaries.
5. If the task involves Markdown files, use the Obsidian Markdown skill for detailed formatting/linking instructions when available.
6. If the prompt belongs to an existing research process, identify its process ID from recent [[wiki/process-log]] entries or [[wiki/index]] before continuing; when starting a clearly new research process, assign the next simple process ID and mention it in the note and process log.

Do not read the whole process log by default. Read the newest relevant entries first, then follow links outward only when continuity matters.
```

Do not modify global Codex hook configuration unless the user explicitly asks for runtime hook installation.

## Vault Layout

- `raw/`: immutable source material, including papers, clipped articles, datasets, transcripts, screenshots, and source assets.
- `wiki/`: durable synthesized knowledge maintained by agents.
- `wiki/index.md`: main navigational map.
- `wiki/process-log.md`: newest-first agent memory of the research trajectory.
- `wiki/topics/`: broad research areas.
- `wiki/concepts/`: reusable concepts, definitions, and conceptual building blocks.
- `wiki/methods/`: algorithms, mathematical methods, workflows, and technical procedures.
- `wiki/papers/`: durable paper summaries and source-oriented pages.
- `notes/`: working research notes, including derivations, reading notes, speculative ideas, scratch calculations, and open questions.
- `code/`: scripts, notebooks, toy implementations, numerical experiments, and utilities.
- `outputs/`: generated artifacts such as tables, data exports, rendered summaries, and intermediate results.
- `figures/`: generated or imported figures intended for reuse in notes and wiki pages.
- `docs/`: project-level documentation and design specs.

Derivations belong in `notes/`, not in a separate derivation tree, because derivation and research interpretation are usually intertwined.

## Obsidian Conventions

Use Obsidian-style `[[Wiki Links]]` for internal links. Prefer links and backlinks over long duplicated summaries.

When a task involves creating, editing, reviewing, or formatting Markdown files for this vault, use the Obsidian Markdown skill for detailed formatting and link-handling instructions when it is available.

Use YAML frontmatter where it helps search and filtering:

```yaml
---
type: wiki | note | source | code | output | figure | process-log
status: seed | active | stable | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Use tags sparingly for broad categories only. Do not create dense tag taxonomies early.

## Human-Readable Names

Keep the approved top-level folders as they are, but make future human-facing folders and Markdown filenames easy to read in Obsidian.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zzhliu05/agent-formalizer](https://github.com/zzhliu05/agent-formalizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
