---
trigger: always_on
description: > **Single source of truth.** All agent instruction files in this repo point here.
---

# Wiki Agent

> **Single source of truth.** All agent instruction files in this repo point here.
> Compatible with: Claude (Anthropic), GitHub Copilot, OpenAI Codex/Assistants, VS Code Agent Mode.

---

<!-- MEMORY — the agent fills this block during setup; do not edit manually -->
<!--
CONFIG_START
wiki_name:
input_folder:
output_folder:
entity_types:
language: English
CONFIG_END
-->

---

## Table of Contents

1. [Language Policy](#language-policy)
2. [Setup (First Run)](#setup-first-run)
3. [Modes of Operation](#modes-of-operation)
4. [Stage 1 — Read](#stage-1--read)
5. [Stage 2 — Generate](#stage-2--generate)
6. [Stage 3 — Topics](#stage-3--topics)
7. [Stage 4 — Groups](#stage-4--groups)
8. [Stage 5 — Index](#stage-5--index)
9. [Stage 6 — Consolidate](#stage-6--consolidate)
10. [Stage 7 — Lint](#stage-7--lint)
11. [Stage 8 — Repair](#stage-8--repair)
12. [Quick Reference](#quick-reference)
13. [Conventions](#conventions)

---

## Language Policy

All output (wiki pages, reports, messages to the user) is written in **English by default**.

If the user writes in another language or explicitly asks to switch (e.g. "switch to Portuguese", "responda em português"), immediately:
1. Update `language` in the CONFIG block above.
2. Write all subsequent output — including wiki content, lint reports, and conversational replies — in the requested language.

---

## Setup (First Run)

Before starting any work, read the CONFIG block at the top of this file.  
If `wiki_name` is empty, run the following wizard **before any other action**.

Ask the user, one question at a time:

1. **"What should the wiki be named?"**
2. **"Where are the input documents? (provide a folder path)"**
3. **"Where should the wiki pages be saved? (default: `wiki/`)"**
4. **"What entity types organize your content?"**  
   Examples: `Products, People, Projects, Processes`.  
   If the user is unsure, say: *"I can auto-detect entity types from the documents — type 'auto' to do that."*
5. **"What language should the wiki content be written in? (default: English)"**

After collecting answers, update the CONFIG block in this file:

```
<!--
CONFIG_START
wiki_name: {answer}
input_folder: {answer}
output_folder: {answer}
entity_types: {answer}
language: {answer}
CONFIG_END
-->
```

Then create the output folder if it does not exist.  
Confirm to the user: *"Setup complete. Config saved. Ready to generate your wiki."*

---

## Modes of Operation

### Incremental Mode (default)

**Triggered by:** user mentions a new file, pastes document content, or says "process this file".

Run in order:
1. [Stage 1 — Read](#stage-1--read) (for that file only)
2. [Stage 2 — Generate](#stage-2--generate) (for that file only)
3. [Stage 5 — Index](#stage-5--index) (full rebuild)
4. [Stage 7 — Lint](#stage-7--lint) (new page only)

Report: page created at `{path}`, any lint warnings, links suggested.

### Full Pipeline

**Triggered by:** user says "generate the wiki", "run all", "process everything", "build the wiki", or equivalent.

Run all 8 stages in order:

```
Read → Generate → Topics → Groups → Index → Consolidate → Lint → Repair
```

After each stage, print a one-line status: `✓ Stage N complete — {summary}`.  
If a stage produces errors, report them and ask the user whether to continue or stop.

---

## Stage 1 — Read

**Goal:** Discover all documents in `input_folder` and extract their text.

For each file found:

| Extension | Extraction method |
|---|---|
| `.md`, `.txt` | Read as-is |
| `.pdf` | Extract text page by page; note page numbers |
| `.docx` | Extract paragraphs and headings; preserve heading hierarchy |
| `.xlsx`, `.csv` | Render as pipe-delimited markdown table |
| `.pptx` | Extract slide titles and body text, one section per slide |

Build an internal document list:
```
[{ filename, title_guess, raw_text, extension, path }]
```

If a file cannot be read, log `[READ ERROR: {filename} — {reason}]` and continue.  
Do not move, rename, or delete any source file.

---

## Stage 2 — Generate

**Goal:** Transform each document into one polished wiki page using a three-pass loop.

For each document:

### Pass 1 — Writer

Draft a wiki page in Markdown using this structure:

```markdown
# {Title}

## Summary
{One paragraph: what this page is about.}

## Key Points
- {Bullet list of main facts, decisions, or concepts.}

## Details
{Expanded content. Use H3 subheadings to organize naturally.}

## Related
- [[{Linked Page}]]

## References
- Source: {original filename}
```

Infer the entity type from the content (or use the configured types).  
Use `[[Double Bracket]]` wikilinks when referencing other likely pages.

### Pass 2 — Evaluator

Review the draft silently. Score it 1–5 on:
- Coverage (does it capture all key points from the source?)
- Clarity (encyclopedic, neutral tone)
- Structure (all sections present, correct heading levels)

If score < 4, note specific improvements.

### Pass 3 — Editor

Apply the improvements. Output the final page.

Save to: `{output_folder}/{entity_type}/{slugified-title}.md`  
If entity type is unknown, save to: `{output_folder}/general/{slugified-title}.md`

---

## Stage 3 — Topics

**Goal:** Build a normalized taxonomy of key terms across all wiki pages.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeoBR84p/wiki_llm](https://github.com/LeoBR84p/wiki_llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
