---
trigger: always_on
description: Convert PDF game rulebooks into a Traditional Chinese Markdown documentation site.
---

# game-doc-template

Convert PDF game rulebooks into a Traditional Chinese Markdown documentation site.

## Immutable Laws

<law>

**Law 1: Communication**

- Concise, actionable responses
- No unnecessary explanations
- No summary files unless explicitly requested

**Law 2: Skill Discovery**

- MUST check available skills before starting work
- Invoke applicable skills for specialized knowledge
- If ANY skill relates to the task, MUST use Skill tool to delegate

**Law 3: Convention Consultation**

- When task relates to documentation formatting or translation style, apply established project conventions
- MUST maintain consistency with existing patterns

**Law 4: Parallel Processing**

- MUST use Task tool for independent operations
- Batch file searches and reads with agents

**Law 5: Reflexive Learning**

- Important discoveries -> remind user: `/reflect`

**Law 6: Traditional Chinese Only**

- All user-facing outputs — translated content, conversational replies, and every other interaction with the user — must be Traditional Chinese.
- Translation target language is fixed to zh-TW (Taiwan usage).
- Simplified Chinese is not allowed.
- Mainland China-specific wording is not allowed.
- Terminology must remain consistent.

**Law 7: Terminology Consistency**

- Must follow term mappings in `glossary.json`.
- New terms must be added to the glossary before use.
- If `proper_nouns.mode != keep_original`, proper nouns appearing 2 or more times in corpus must be treated as managed terms in glossary workflow.
- Preserve source meaning and avoid over-localization.
- Proper noun policy (person/place/org/brand/product names) is user-configurable during `/init-doc`; do not hardcode a single rule.
- Terminology workflow must reuse `.claude/skills/terminology-management/SKILL.md`.
- `/init-doc`, `/translate`, and `/bilingual-translate` must run terminology read/consistency checks first. The deprecated `/super-translate` entry forwards to `/translate`.

**Law 8: zh-TW Writing Conventions**

- MUST use Traditional Chinese punctuation per `.claude/rules/docs-conventions.md` in all user-facing Chinese text
- MUST avoid Mainland China-specific and Hong Kong-specific wording; prefer Taiwan usage

**Law 9: User Consultation for Complex Terms**

- For rare characters, puns, or culturally nuanced terms, MUST consult user before finalizing terminology decisions when ambiguity affects meaning or tone

</law>

## Quick Reference

### Slash Skills

| Command               | Description                                                           |
| --------------------- | --------------------------------------------------------------------- |
| `/new-project`        | Create a new project from template and set up a private GitHub repo   |
| `/init-doc`           | Initialize, verify, then auto-route to the selected full translation mode |
| `/chapter-split`      | Split extracted Markdown into semantic docs pages and regenerate nav  |
| `/translate`          | Translate with reusable context; whole-book completion builds the final site |
| `/super-translate`    | Deprecated compatibility entry; forwards to `/translate`             |
| `/md-review`          | Check Markdown structure and style compliance for docs or drafts      |
| `/bilingual-translate` | Single-pass bilingual translate: Chinese primary + English blockquote (no review loop) |
| `/terminology-management` | Glossary-driven terminology creation, edit, validation, and enforcement |
| `/check-consistency`  | Validate terminology consistency                                      |
| `/term-decision`      | Make terminology decisions and batch replace                          |
| `/check-completeness` | Check for missing rule content                                        |
| `/fix-ref`            | Convert printed page references into internal Markdown links          |
| `/final-proofread`    | Final quality sweep: frontmatter, content, page refs, search verify   |

### Tech Stack

- **Frontend**: Astro 5 + Starlight + starlight-auto-sidebar (bun/npm)
- **Search**: Pagefind + custom zh-TW segmentation layer (`docs/search/`, see its README)
- **Scripts**: Python 3.11+ (uv)
- **PDF Processing**: markitdown, pymupdf, opendataloader-pdf (default engine, requires Java 11+)

### Key Paths

| Path                                             | Description                                        |
| ------------------------------------------------ | -------------------------------------------------- |
| `docs/`                                          | Astro documentation site                           |
| `docs/src/content/docs/`                         | Markdown content                                   |
| `docs/search/`                                   | zh-TW search enhancement (build post-process)      |
| `scripts/`                                       | Python processing scripts                          |
| `data/pdfs/`                                     | Source PDF files                                   |
| `data/markdown/`                                 | Extracted Markdown                                 |
| `data/markdown/images/`                          | Extracted images                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wayne930242/game-doc-template](https://github.com/wayne930242/game-doc-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
