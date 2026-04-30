---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

茵蒂克丝.skill（LLM Wiki）is a knowledge management system built as Claude Code skills. It captures arbitrary input (text, URLs, files, folders), auto-classifies it into 6 types, and produces structured Obsidian notes with extracted images. It also provides persona-driven chat with memory-augmented retrieval.

## Skills

Four complementary skills in `skills/`:

- **index-init** (`/index-init`): First-run setup. Creates the Obsidian vault directory structure, copies templates from `skills/index-init/resources/` to the vault, and generates `persona.md` via interactive MBTI questionnaire.
- **index-note** (`/index-note INPUT`): Core skill. Classifies input → downloads content if needed → extracts images → fills type-specific template → writes note to `_new/`.
- **index-chat** (`/index-chat INPUT`): Chat skill. Loads persona from `persona.md` → extracts keywords from user input → retrieves relevant knowledge from `memory/` via layered navigation → responds in persona style with references → saves conversation to `_chat/`.
- **index-update** (`/index-update`): Knowledge organizer. Scans `_new/` for notes marked "已读" → extracts knowledge items → classifies into 4-type hierarchical memory system (`memory/`) → archives note to `deep/`. Also processes `_chat/` logs: applies persona updates to `persona.md`, extracts user-provided knowledge, and archives chat files to `deep/`. The read marker is a literal checkbox at the bottom of every note: `- [x] <big><big>已读</big></big>` triggers archiving; `- [ ] <big><big>已读</big></big>` is skipped.

Each skill is defined by a `SKILL.md` file with frontmatter:
```yaml
---
name: index-note
description: ...
allowed-tools: Read, Write, Bash, WebFetch, Glob, Grep, WebSearch
---
```

## Key Paths

All paths are relative to the project root:

```
IndexVault/              Obsidian vault root
IndexVault/_new/         Generated notes output (YYYY-MM-DD_TYPE_NNN.md)
IndexVault/deep/         Archive for read notes (moved from _new/ after reading)
IndexVault/memory/       Hierarchical knowledge base (see Memory System below)
IndexVault/_template/    6 note templates (idea/project/book/paper/webinfo/webnews)
IndexVault/_images/      Extracted images, organized per-note ({note_id}/{note_id}_{nn}.ext)
IndexVault/_downloads/   Downloaded source material
IndexVault/_chat/        Chat logs (YYYY-MM-DD_Chat.md, archived to deep/ by index-update)
IndexVault/persona.md    Agent personality config (created by index-init, updated by index-update from chat logs)
```

## Memory System

Hierarchical knowledge base in `IndexVault/memory/` with layered index navigation (never load all knowledge at once):

```
memory/
├── memory-index.md              # Level 0: routes to knowledge category
├── 事实性记忆/                    # "是什么" — definitions, facts, formulas
│   ├── _local_index.md      # Level 1: routes to keyword file
│   └── {KEY_WORD}.md            # Level 2: actual knowledge entries
├── 程序性记忆/                    # "怎么做" — methods, workflows, code patterns
│   ├── _local_index.md
│   └── {KEY_WORD}.md
├── 条件性记忆/                    # "何时/为何用" — trade-offs, selection criteria
│   ├── _local_index.md
│   └── {KEY_WORD}.md
└── 元认知记忆/                    # "怎么学/怎么错" — learning strategies, reflections
    ├── _local_index.md
    └── {KEY_WORD}.md
```

KEY_WORD files use PascalCase English (e.g., `LLM.md`, `Transformer.md`). Each contains multiple knowledge entries with summaries and wikilinks back to source notes in `deep/`.

**Invariants when working with memory:**
- **Layered access only**: Always navigate `memory-index.md` → `_local_index.md` → `KEY_WORD.md`. Never glob/read all `KEY_WORD.md` files at once.
- **Concept-level keywords**: Keywords must be reusable across notes (`LLM`, `Diffusion`, `ReinforcementLearning`), not note-specific titles (`AttentionIsAllYouNeed`, `GPT4TechnicalReport`). A keyword is expected to accumulate multiple entries over time.
- **Archive before memory write** (index-update): Move the note to `deep/` *first*, then write memory entries. This way every `来源` wikilink points to the actual archived location (and uses the post-collision filename if a suffix was added).

## Running Python Scripts

Python is accessed via `uv` (no system Python installed):

```bash
# Classification
uv run python ./skills/index-note/scripts/classify_input.py --input "INPUT_STRING"

# ID generation (scans both _new/ and deep/ to avoid collisions after archiving)
uv run python ./skills/index-note/scripts/generate_id.py --type TYPE --vault-new-dir ./IndexVault/_new/ --vault-deep-dir ./IndexVault/deep/

# Image extraction to temp dir (needs PyMuPDF)
# Progress goes to stderr; stdout is compact JSON: {"total": N, "images": [{"filename": "...", "width": N, "height": N}, ...]}
uv run --with pymupdf --with requests python ./skills/index-note/scripts/extract_images.py \
  --type paper --input "2505.00949" --note-id "2026-04-05_paper_001" \
  --output-dir ./IndexVault/_images/_tmp_2026-04-05_paper_001/

# Finalize images (keep only those referenced in note, move to _images/{note_id}/)
# stdout: {"kept": N, "removed": N}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaihuaTang/Index.skill](https://github.com/KaihuaTang/Index.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
