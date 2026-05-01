---
trigger: always_on
description: You are operating on a knowledge base built on the LLM Wiki pattern (Karpathy 2026), with four optimizations: an atom layer, topic-branch organization, two-layer Lint, and parallel-compile naming locks.
---

# CLAUDE.md — Schema for LLMs operating this repo

You are operating on a knowledge base built on the LLM Wiki pattern (Karpathy 2026), with four optimizations: an atom layer, topic-branch organization, two-layer Lint, and parallel-compile naming locks.

This file is the formal spec — read it before touching anything. Mental model, operations, file formats, lifecycle rules, and what you must never do.

---

## Mental model

Three storage layers, one navigation layer, one log layer:

```
raw/                  sources you may read but never write
atoms/                knowledge atoms, organized by topic-branch
  <branch-1>/         one folder per topic
  <branch-2>/         each contains atoms (source of truth)
  ...
wiki/                 compiled pages from atoms, derived cache
index.md              auto-generated wiki navigation
log.md                append-only change history
```

Atoms are immutable. Wiki is rebuildable from atoms. If a wiki page is wrong, fix the underlying atom and recompile, never patch the wiki.

Each branch folder under `atoms/` holds the atoms for that topic, plus an optional `_archive/` for superseded atoms.

---

## Atom format (spec)

### Frontmatter (YAML, required)

```yaml
---
id: <branch>/<descriptive-slug>
type: explanation | opinion | tutorial | myth-busting | case-study | comparison
depth: beginner | intermediate | advanced
source_type: post | reply | thread | transcript | article | note | screenshot | audio
source_ids: []
reuse_score: high | medium | low
tags: []
created: YYYY-MM-DD
---
```

| Field | Notes |
|-------|-------|
| `id` | Format `<branch>/<slug>`. Slug all lowercase, hyphens only. Must be unique within branch. |
| `type` | What kind of knowledge this atom carries. Pick one. |
| `depth` | Audience level. Used in gap analysis. |
| `source_type` | Where the raw came from. Extend the enum if your sources differ. |
| `source_ids` | Stable identifiers (URLs, paths, post IDs). Atoms without source attribution are not auditable. |
| `reuse_score` | `high` = standalone-publishable, `medium` = needs companions, `low` = niche. |
| `tags` | Cross-cutting concerns. Used to surface related atoms across branches. |
| `created` | ISO date. Used for chronological ordering and stale-detection. |

Optional fields you may add: `confidence` (high/medium/low), `superseded_by` (id of replacement atom), `archived` (boolean).

### Filename

Pattern: `YYYY-MM-DD-<descriptive-slug>.md`

- Date prefix gives natural chronological order in `ls`.
- Slug all lowercase, hyphens only — no underscores, no spaces, no uppercase.
- Slug should be 3–6 words describing the core claim.

### Body

- One core claim per atom. If two independent claims share a paragraph, split into two atoms.
- Refine, don't copy. Strip filler from the source; preserve the author's voice and stance.
- Cite source at the end if you want traceability beyond `source_ids`.

### Lifecycle (immutable)

Atoms are immutable. Do not edit after creation.

When knowledge evolves:
1. Create a new atom with the updated claim.
2. Add `superseded_by: <new-atom-id>` to the old atom's frontmatter.
3. Move the old atom to `atoms/<branch>/_archive/`.
4. Recompile any wiki page that referenced the old atom.

Never delete an atom outright. Use `_archive/`. Final deletion is a separate, conscious decision.

See `atoms/_template.md` for a copyable starter.

---

## Wiki page format (spec)

### Filename

Pattern: `<branch>-<topic-slug>.md`

- Branch prefix lets `gen-index.sh` group pages.
- All lowercase, hyphens only.
- Lives flat in `wiki/`, not in subfolders.

### First line

Must be `# Title`. `gen-index.sh` reads this for the index entry. Lint flags violations.

### Wiki links

Pattern: `[[slug]]` or `[[slug|display text]]`

- The slug must equal an existing wiki filename without `.md`.
- Lint flags ghost links and orphan pages.

### Body structure

```markdown
# Page Title

Opening paragraph: why this matters, common misconception, what you'll get.

## Section
Integrated content from multiple atoms, written as coherent prose.
First mention of a related concept gets a [[wiki-link]]; subsequent
mentions in the same page don't repeat it.

## Section
Continue.

---

**See also**
- [[related-page]] — one-line description

---
*Compiled from atoms: branch/atom-slug-1, branch/atom-slug-2, ...*
```

### Length

- Target 1500–2500 words per page.
- Past 2500 words: consider splitting.
- Below 800 words: consider merging or staying at atom level.

### Temporal markers

In time-sensitive claims, use one of:
- Specific date: `as of 2026-04`, `截至 2026-04`
- Version number: `v3.5`, `Claude 3.5 Sonnet`

Avoid bare `currently` / `latest` / `now` / `目前` / `現在` in time-sensitive contexts. Lint regex flags only `<temporal word> <version/date>` combinations to avoid false-positive flooding from rhetorical use.

See `wiki/_template.md` for a copyable starter.

---

## Branch design (spec)

### When to add a branch (all four required)

1. **Independence** — the topic doesn't fit cleanly under any existing branch.
2. **Scale** — you expect 5+ atoms in this branch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cablate/llm-atomic-wiki](https://github.com/cablate/llm-atomic-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
