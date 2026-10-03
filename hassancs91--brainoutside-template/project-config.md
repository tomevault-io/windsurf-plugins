---
trigger: always_on
description: This repo is your brain: the single knowledge base all of your AI agents
---


# YOUR MIND — Schema & Operating Contract

This repo is your brain: the single knowledge base all of your AI agents
read from for context, facts, and voice. This file is the contract. Every
agent that reads or writes this repo MUST follow it. If an instruction
elsewhere conflicts with this file, this file wins.

Two skills operate on this repo:
- `.claude/skills/mind-feeder/` — the ONLY writer. Compiles new sources in.
- `.claude/skills/mind-reader/` — the consumer protocol. How agents retrieve.

> **New here?** Read `README.md` first — it tells you which parts of this
> file to edit before you feed anything in. The structure below is the
> engine's contract and should stay as-is; the *taxonomy* (§7) and the
> identity files are yours to write.

---

## 1. Layout

| Path | Layer | What lives here |
|---|---|---|
| `INDEX.md` | Index | Catalog of every entity: one line + status each. Read first, always. |
| `identity/` | Identity core | Who you are, how you write, what you believe. Small. Loaded by every consumer. |
| `projects/` | Cards | One card per project/product/series. Status, numbers, pitch, pointers. |
| `knowledge/` | Distilled notes | Atomic notes: takes, stories, lessons, facts. The retrieval workhorse. |
| `content-catalog/` | Inventory | Tables of published content per platform (for remix/repurpose queries). |
| `lenses/` | Scopes | Named default filters that consumers can invoke. |
| `raw/` | Archive | Full transcripts/posts, or pointers to them. Almost never loaded. |
| `eval/` | Quality | The falsification test. Run before trusting changes to the schema. |

## 2. Scope rule (hard)

**The mind knows ABOUT your work; it does not CONTAIN your work.**

A note is a distillation with provenance — a position, a story, a lesson, a
fact. The artefact itself (the codebase, the manuscript, the video file, the
course, the dataset) lives wherever it already lives. The project card
points at it.

Two consequences worth stating plainly:

- **Size**: if you find yourself pasting a whole document in, you want a
  `raw/` archive plus a note that links to it, not a giant note.
- **Language**: if some of your work is in another language, keep the mind
  in ONE language (whichever your agents write in) and let it hold the
  *about*-layer — what you are building, why, what you learned. A
  translated or mixed-language corpus inside the mind will poison voice
  retrieval. Point at the corpus from the project card instead.

## 3. Note frontmatter schema (knowledge/)

Every file in `knowledge/` starts with:

```yaml
---
id: take-2026-07-rag-vs-finetuning        # type-YYYY-MM-slug, unique
type: take | story | lesson | fact         # matches its folder
topics: [rag, agents]                      # from TAXONOMY below, 1-4 tags
projects: [my-library]                     # entities touched (optional)
source: yt-2026-06-rag-video                # id in content-catalog, or raw/ path
source_url: https://...                    # link to the original; null ONLY
                                           # for direct thoughts (see §5.2)
date: 2026-07                              # when this was said/published
status: current | superseded
superseded_by: null                        # id of the newer note, if superseded
visibility: public | agents-only | private
---
```

Body rules:
- **Voice is sacred**: every `take` and `story` MUST include at least one
  verbatim quote of your actual words from the source, marked as
  `> VERBATIM: "..."`. Paraphrase around it, never instead of it.
- One idea per note. If extraction finds two ideas, write two notes.
- 5–15 lines of body. Notes are retrieval units, not essays.

Type meanings:
- `take` — an opinionated position ("my angle on X").
- `story` — a personal narrative with numbers/failures/outcomes.
- `lesson` — a transferable "what I learned building/testing X".
- `fact` — a stable, citable fact about your work or results.

## 4. Visibility (hard)

- `public` — derived from published content. Safe for any consumer,
  including future audience-facing chatbots.
- `agents-only` — derived from source code, planning docs, or unpublished
  thinking. Your own agents only. Any audience-facing serving layer MUST
  filter this out server-side, before context assembly.
- `private` — do not surface in any generated output; background context only.

Default: notes from published content → `public`. Notes from repos/planning →
`agents-only`. When unsure → `agents-only`.

### Path defaults (files with no `visibility:` frontmatter)

Explicit frontmatter always wins. Files without it resolve by path:

| Path | Resolution |
|---|---|
| `identity/*`, `knowledge/*`, `projects/*` | frontmatter required; missing → `agents-only` |
| `content-catalog/*` | `public` (inventory of published content) |
| `lenses/*` | `public` |
| `raw/*` | inherits the MAX visibility of the notes/cards linking to it; reachable only via those links, never by browsing |
| `INDEX.md` | serving layers serve a generated, tier-filtered index — never the raw file |
| `PENDING.md` | workbench — you + the feeder only; never served to any consumer tier |
| `eval/*`, `*/_TEMPLATE.md`, `CLAUDE.md`, `.claude/*`, `README.md` | infrastructure, not content — never served as notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hassancs91/brainoutside-template](https://github.com/hassancs91/brainoutside-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
