---
trigger: always_on
description: >
---


# Xplor — Structured Cognition Engine

Xplor transforms **documents**, **codebases**, and **knowledge systems** into
traversable knowledge graphs — then reasons over those graphs rather than summarizing
raw text.

Traditional analysis: read text → produce text. Useful, but shallow.

Xplor analysis: extract structure → build graph → traverse relationships → answer
questions that raw text cannot. When asked "who is the key decision-maker in this
contract chain?" — traverse the entity graph. When asked "what breaks if I change
this function?" — walk the call graph. When asked "how does this therapy technique
connect to attachment theory?" — follow the wikilinks.

---

## Three Modes

### Document Mode
**Input:** PDFs, text files
**Method:** AI entity extraction
**Output:** People, organizations, locations, concepts, events, documents + their relationships

Extract every named entity and the relationships between them. The result is a
traversable graph, not a summary. Multiple documents can be merged — entities
deduplicated, connections traced across sources.

See `references/document-mode.md` for the extraction pipeline and entity schema.

### Code Mode
**Input:** GitHub repository URL (v1) or local repository path (v2/CLI)
**Method:** GitHub API + AI analysis (v1) | AST parsing with Tree-sitter (v2)
**Output:** Functions, classes, imports, call chains, modules + dependency relationships

v1 (web) uses the GitHub API to fetch repo structure and key files, then Claude
analyses architecture. v2 (CLI) uses Tree-sitter AST parsing for deep call-chain
graphs with full function/class/import extraction.

See `references/code-mode.md` for both pipeline specifications.

### Skill Graph Mode
**Input:** ZIP of markdown files with `[[wikilinks]]` and YAML frontmatter
**Method:** Structural parsing — frontmatter extraction + wikilink resolution
**Output:** Knowledge nodes, Maps of Content, claims, techniques + semantic connections

Each markdown file becomes a node. Each `[[wikilink]]` becomes an edge, resolved
with sentence-level context. MOC nodes cluster related concepts. Quality is scored
0–100 with specific, actionable issue reporting.

See `references/skill-graph-spec.md` for the full specification.

---

## Core Framework

### The Graph Data Model

All three modes write to the same canonical schema. This enables multi-domain
fusion — a code graph and a knowledge graph merged into one queryable layer.

```
GraphNode
  id:          Namespaced — "skill:cognitive-reframing" | "func:validateToken"
  kind:        document | code | skill
  type:        person | function | moc | claim | organization | class | ...
  name:        Display name
  description: One-sentence summary (required for quality score)
  domain:      Subject area: therapy | auth | legal | trading | ...
  tags:        []string
  content:     { full, sections[], preview }   ← progressive disclosure
  source:      { filePath, lineRange, document }
  metadata:    { wordCount, aliases, inDegree, outDegree }

GraphEdge
  source:      Node id
  target:      Node id
  type:        REFERENCES | CLUSTERS | EXTENDS | CONTRADICTS |
               CALLS | IMPORTS | DEFINES | RELATED_TO | CROSS_DOMAIN
  label:       Human-readable relationship
  strength:    1–5
  context:     Sentence that contained the link (wikilink/call-site context)
```

Full schema spec: `references/graph-core.md`

### Progressive Disclosure

Knowledge retrieval operates across five levels. Claude loads only what the
current task requires — never dumping full content into context.

| Level | Name | Content Loaded | ~Tokens/node |
|-------|------|---------------|-------------|
| 0 | Index | IDs + types only | 2 |
| 1 | Descriptions | + name + one-line description | 15 |
| 2 | Links | + connection list | 30 |
| 3 | Sections | + section headings + previews | 80 |
| 4 | Full | Complete content | 200–500 |

Traversal pattern:
```
1. Load Level 0 (full graph index) — what exists?
2. Load Level 1 for matching nodes — what is relevant?
3. Load Level 2 for top matches — what connects to what?
4. Load Level 3 for confirmed relevant nodes — confirm depth
5. Load Level 4 for 3–8 most relevant — deep read
6. Assemble context pack within token budget
```

Full rules and token budgets: `references/progressive-disclosure.md`

### Skill Graph Format

A skill graph is a folder of markdown files:

```markdown
---
name: cognitive-reframing
description: >
  A CBT technique for identifying and challenging distorted thought patterns.
type: technique
domain: therapy
tags: [cbt, cognition, distortions]
extends: [thought-records]
---

# Cognitive Reframing

Cognitive reframing involves examining the evidence for and against an
automatic negative thought (see [[thought-records]]).

## When to Apply
Use when the client presents with [[cognitive-distortions]] such as
catastrophizing or black-and-white thinking.
```

**Frontmatter fields:** `name`, `description`, `type`, `domain`, `tags`,
`aliases`, `extends`, `contradicts`

**Node types:** `skill` · `moc` · `technique` · `claim` · `framework` · `exploration`

**Edge types from wikilinks:**
- Default wikilink → `REFERENCES`
- Source node is type `moc` → `CLUSTERS`
- Frontmatter `extends: [target]` → `EXTENDS`
- Frontmatter `contradicts: [target]` → `CONTRADICTS`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amlitio/file-xplor-skill](https://github.com/amlitio/file-xplor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
