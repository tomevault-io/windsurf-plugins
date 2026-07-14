---
trigger: always_on
description: This file tells an LLM agent (Claude Code, Codex, etc.) how the ICT Knowledge Library is organized and how to ingest new sources, answer queries against the wiki, and lint the wiki for consistency.
---

# AGENTS.md — Wiki Schema for LLM Maintenance

This file tells an LLM agent (Claude Code, Codex, etc.) how the ICT Knowledge Library is organized and how to ingest new sources, answer queries against the wiki, and lint the wiki for consistency.

It implements the [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern for the **ICT (Inner Circle Trader) trading methodology** domain, 2016–2026.

---

## The three layers

1. **Raw sources** — ICT mentorship materials, YouTube videos, X threads, transcripts, PDFs. Currently the library does not store raw source files; it cites them via stable IDs in `SOURCES.md`. If a future ingest workflow attaches raw files, add them under `raw/` and treat as immutable.
2. **The wiki** — the 226 concept files in `concepts/**/*.md` plus 9 root files (this AGENTS.md, README, TEMPLATE, INDEX, GLOSSARY, TIMELINE, READING-ORDER, SOURCES, CONTRIBUTING, CHANGELOG, log.md). LLM-maintained.
3. **The schema** — this file (`AGENTS.md`) plus `TEMPLATE.md` and `CONTRIBUTING.md`.

---

## Index files

- **`INDEX.md`** — content-oriented catalog of every concept file, organized by directory (`01-market-structure` through `99-glossary`). Each entry: filename + one-line summary. The LLM reads `INDEX.md` first when answering queries, then drills into specific concept files.
- **`TIMELINE.md`** — year-by-year (2016 → 2026) chronological record of which concepts were introduced or refined in which year. Used for "what changed in 2024?" type questions.
- **`GLOSSARY.md`** — alphabetical abbreviation lookup (BSL, FVG, OB, etc.).
- **`SOURCES.md`** — canonical citation registry. Stable IDs (e.g. `ICT-2022-E03`, `ROMEO-2024-CRT`).
- **`READING-ORDER.md`** — three learning tracks (Beginner / Intermediate / Advanced) for human readers.
- **`log.md`** — chronological event log of activity on the wiki.
- **`CHANGELOG.md`** — structured phase-by-phase build log (older format; `log.md` is the current going-forward log).

---

## File format

Every concept file follows `TEMPLATE.md`:

- 7 required top-matter fields: `Category`, `Aliases`, `ICT Confidence` (high / medium / community-attributed / disputed / demo-stage), `Year Introduced`, `Year Refined`, `Source IDs`, `Tags`.
- 10 required sections: `## Definition`, `## Formal Criteria`, `## Formula / Math`, `## Machine-Readable` (JSON block with `id` matching filename), `## Visual Pattern`, `## Timeframes`, `## Examples`, `## Common Mistakes`, `## Related Concepts`, `## Citations`.
- Optional: `## ICT vs Community` (only for `community-attributed`, `disputed`, or `demo-stage` files).

Filenames are `kebab-case.md`. The JSON block's `id` MUST match the filename (without `.md`).

See `TEMPLATE.md` for the canonical skeleton and `CONTRIBUTING.md` for the hard rules.

---

## Operations

### Ingest — add a new ICT source

When the user provides a new source (a 2027 ICT video, a fresh X thread, a community post worth citing):

1. **Read the source.** Extract: what concepts are introduced or refined; what year; quotes; specific timestamps if a video.
2. **Decide on a stable Source ID.** Format: `<PUBLISHER>-<YEAR>-<SLUG>`. Add it to the appropriate section of `SOURCES.md`. Stable IDs are append-only — never renumber.
3. **For each concept introduced or refined in the source:**
   - If a new concept: write a new file in the right `concepts/NN-<dir>/` directory using `TEMPLATE.md`. Set `ICT Confidence` honestly. Set `Year Introduced` and `Year Refined` correctly. Cite the new Source ID.
   - If an existing concept refined: update its `Year Refined`, append the Source ID to its top-matter `**Source IDs:**` line, append a citation to `## Citations`, and update body text only where the source genuinely changes the canonical understanding.
4. **Update `INDEX.md`** with any new concept files.
5. **Update `TIMELINE.md`** under the source's year section.
6. **If a new abbreviation is introduced**, add it to `GLOSSARY.md`.
7. **Append an entry to `log.md`** of form `## [YYYY-MM-DD] ingest | <source title>` with a 2-3 line summary of what changed.
8. **Run a quick lint** (see Lint section below) on the touched files.
9. **Commit** with a message matching the existing style (e.g. `ingest: <source title> — N files updated`).

### Query — answer a question against the wiki

When the user asks a question:

1. **Read `INDEX.md` first.** Identify candidate concept files relevant to the question.
2. **Read the candidate files** (typically 3-8 of them).
3. **Synthesize the answer.** Cite specific files inline, using the file path as link target — e.g. `per [bullish-fvg](concepts/06-fair-value-gaps/bullish-fvg.md)`.
4. **If the question reveals a gap** (a concept missing, a contradiction between files, an outdated claim), say so explicitly and recommend an ingest or lint pass.
5. **Optionally, file the answer back into the wiki.** If the answer is genuinely useful (a comparison, an analysis, a derived insight), consider creating a new concept file or appending a section to an existing one. Don't lose valuable analysis to chat history.

### Lint — wiki health check

Periodically (or when asked):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SrsBlack/ict-knowledge-library](https://github.com/SrsBlack/ict-knowledge-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
