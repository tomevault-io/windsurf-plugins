---
trigger: always_on
description: Read at start of every session before any wiki work.
---

# Wiki Operating Schema

Read at start of every session before any wiki work.

## Purpose

LLM-maintained research wiki for an academic researcher. Configure the research topics below for your domain. Default topics:
- Learning theory (decolonial, mycelial, challenge-based, boundary crossing)
- Sustainability transitions + higher education
- Systems thinking + wicked problems
- Education design research (EDR)
- AI in education and SME contexts
- Lifelong Learning

Wiki feeds papers, research projects, and a future book. Not a note-taking system — a compiled, synthesised knowledge base.

## Configuration

Set these in your local copy before using the skills:

```
WIKI_ROOT: /path/to/your/wiki/
RAW_ROOT:  /path/to/your/raw/
WIKI_LANGUAGE: en
RESEARCHER_NAME: Your Name
INSTITUTION: Your Institution
```

## Language Policy

**All wiki pages in English.** Raw sources may be in any language. Summarise non-English sources in English. Quote key terms in the original language where no clean English equivalent exists, with translation in parentheses.

## Directory Layout

```
wiki/
├── CLAUDE.md          ← this file (operating schema)
├── index.md           ← master catalog of all wiki pages
├── log.md             ← append-only session log
├── overview.md        ← high-level synthesis of entire research landscape
├── concepts/          ← one page per theoretical concept
├── thinkers/          ← key scholars: position, key works, debates
├── themes/            ← synthetic pages that map directly to papers/book chapters
├── tensions/          ← explicit contradictions, open questions, unresolved debates
├── sources/           ← one summary page per ingested source
├── outputs/           ← paper outlines, chapter structures, writing plans
└── fleeting/          ← FUNGI notes: atomic, claim-shaped notes at any maturity stage

raw/
├── articles/          ← source PDFs, clipped markdown articles
├── books/             ← book chapters, excerpts
├── podcasts/          ← podcast notes, transcripts
└── legacy/            ← archived old notes (read-only, never modify)
```

**Raw sources immutable.** Never edit `raw/`. Read from it; write only to `wiki/`.

## Page Formats

### concepts/[concept-name].md
```markdown
---
type: concept
title: [[Full Concept Name]]
aliases: [other names for this concept]
related_concepts: 
- "[[concept-a]]"
- "[[concept-b]]"
key_thinkers:
- "[[thinkers/thinker-a]]"
themes:
- "[[themes/theme-a]]"
sources:
- "[[sources/source-a]]"
last_updated: YYYY-MM-DD
---

# [Concept Name]

## Definition
[2–4 sentence working definition. Synthesized, not copied.]

## Core Elements
[Numbered or bulleted breakdown of key components]

## Theoretical Lineage
[Where does this concept come from? Who developed it? How has it evolved?]

## Relevance to This Research
[Why does this concept matter for the researcher's work specifically?]

## In Tension With
[Concepts or positions this concept conflicts with — link to tensions/ pages]

## Key Sources
[Annotated list of 3–6 most important sources for this concept]
```

### thinkers/[lastname-firstname].md
```markdown
---
type: thinker
name: Full Name
institution: Current or last known institution
field: [discipline, subdiscipline]
related_concepts:
- "[[concepts/concept-a]]"
key_thinkers:
- "[[thinkers/thinker-b]]"
sources:
- "[[sources/slug]]"
themes:
- "[[themes/theme-a]]"
last_updated: YYYY-MM-DD
---

# [Full Name]

## Position in Brief
[2–3 sentences: what is this thinker's central argument or contribution?]

## Key Works
[Annotated list of most important works. Mark ingested works with source link.]

## Core Concepts
[Link to concept pages this thinker developed or heavily influenced]

## Relation to This Research
[How does this thinker's work connect to the research agenda?]

## Debates and Critiques
[Who disagrees with them and why? What are the limits of their work?]
```

### themes/[theme-name].md
```markdown
---
type: theme
title: [Theme Name]
related_themes:
- "[[themes/theme-b]]"
key_concepts:
- "[[concepts/concept-a]]"
key_thinkers:
- "[[thinkers/thinker-a]]"
feeds_into: "Paper title or Book: Chapter X"
source_count: N
last_updated: YYYY-MM-DD
---

# [Theme Name]

## Research Question / Thesis Direction
[The central question or argument this theme is building toward]

## Current Synthesis
[The substantive synthesis — most important section. What do we know? What is the argument?]

## Key Evidence and Sources
[Annotated list of sources that support the synthesis]

## Gaps and What's Missing
[What would strengthen this? What sources are needed?]

## Connections to Other Themes
[How this theme intersects with others]
```

### tensions/[tension-name].md
```markdown
---
type: tension
title: [Tension Name]
concepts_involved:
- "[[concepts/concept-a]]"
- "[[concepts/concept-b]]"
thinkers_involved:
- "[[thinkers/thinker-a]]"
themes_involved:
- "[[themes/theme-a]]"
status: open | partially_resolved | resolved
last_updated: YYYY-MM-DD
---

# [Tension Name]

## The Tension
[Plain-language statement of the contradiction or unresolved question]

## Position A
[Statement of one side, with key sources]

## Position B
[Statement of the other side, with key sources]

## Current Assessment
[Where does the researcher's thinking sit? What would resolve this?]

## Why This Matters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FBoschman/claude-wiki-research-skills](https://github.com/FBoschman/claude-wiki-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
