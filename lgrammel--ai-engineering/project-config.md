---
trigger: always_on
description: Validates note structure against type-specific rules: kebab-case filenames, H1 title on line 1, executive summary present, correct section ordering, no unexpected sections, H3 subsections only under Details, no links in Synonyms, and external-only URLs in External references.
---

# Agent Instructions

This repository is a lightweight knowledge workspace about [AI engineering](./concepts/ai-engineering.md) in practice. Each note captures a concept, role, or practice in a concise, definition-first format, covering what matters when building, integrating, and operating AI-powered systems on top of foundation models, and how they behave in production.

## Scope

The focus is AI engineering - the application layer of building with foundation models. ML engineering concepts (model training, data curation, architecture design) are out of scope: reference them with brief context where needed, but do not define them in depth. When an ML concept is relevant, link to it or provide a minimal description sufficient for an AI engineering audience.

## Repository structure

This is a pnpm monorepo. Content directories live at the root; tooling lives under `apps/`.

| Directory          | Contains                                                                                     | Canonical for                      |
| ------------------ | -------------------------------------------------------------------------------------------- | ---------------------------------- |
| `apps/explorer/`   | SvelteKit static site for browsing notes, graph visualization, and search                    | Knowledge base browser             |
| `apps/lint/`       | TypeScript markdown linting and validation scripts (linting only, not general build tooling) | Custom lint tooling                |
| `example-systems/` | Analyses of concrete AI systems as compositions of concepts                                  | Trust analysis of real systems     |
| `concepts/`        | Core term definitions (e.g. [LLMs](./concepts/llm.md), evals, fine-tuning)                   | Terminology and definitions        |
| `ideas/`           | Speculative/emerging ideas, optionally attributed to external sources                        | Opinion-driven or unproven ideas   |
| `threats/`         | AI agent threat descriptions (e.g. context poisoning, tool misuse)                           | Attack vectors and vulnerabilities |

Don't invent definitions in-line. If one is missing or unclear, **add or update a note** in the appropriate directory, then use it.

Treat notes as a living glossary: update entries as understanding changes.

## Writing notes

All note types (concept, idea, threat, example system) share these conventions. Type-specific rules follow in later sections.

### Naming

- **File name**: kebab-case (e.g. `ai-gateway.md`)
- **Title**: `# Term Name` on line 1, matching the primary term
- Avoid near-duplicate names (e.g. "AI Observability" vs "Observability Tools") unless scope intentionally differs

### Template

```markdown
# Term Name

A 1-2 sentence definition (executive summary).

## Details

Optional additional context: typical behaviors, scope, or how it works in practice.

## Examples

- Concrete examples if helpful.

## Synonyms

synonym1, synonym2.

## External references

- https://example.com/source
```

**Required:** title + executive summary (1-2 sentences immediately after title). The executive summary is the first paragraph after the H1 and stands alone as a useful definition of the term.

**Optional sections** (include only when they add significant value, in this order):

- `## Details` - additional context paragraphs when the executive summary alone is insufficient (scope, boundaries, how it works in practice, `Note:` clarifications). Omit this section when the executive summary fully covers the term.
- `## Examples` - concrete instances
- `## Synonyms` - plain text only, no links
- `## External references` - external URLs only; include only references you actually fetched and used

### What to avoid

- `Why it matters:` or `See also:` sections - fold relevance into the definition; use inline links instead of link lists.
- Unverified references or generic link lists (e.g. a standalone "Related concepts:" sentence).
- Prescriptive language in the main section ("should", "must", "do X", "avoid Y"). Phrase operationally as description ("common practice is...") or put guidance under `## Examples`.

### Linking and deduplication

- **One canonical note per idea.** Before creating a file, check for an existing note under a synonym; if found, update it and add the synonym under `## Synonyms`.
- **Link, don't duplicate.** Reference other notes with relative links (e.g. `[LLM](./concepts/llm.md)`) rather than restating definitions.
- **Update related notes** when changing a definition or scope.
- **Cross-link** to relevant concept, threat, and idea notes where it aids understanding.
- Keep each directory's `index.md` sorted alphabetically by visible name.

### Concise but complete

A note is "complete" when a reader can understand the term without guessing key scope details. A good note typically covers:

- What it is (core definition - in the executive summary)
- Where it applies (scope)
- Key boundary or distinction (what it's not)
- If relevant, the most important variants (kept minimal)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lgrammel/ai-engineering](https://github.com/lgrammel/ai-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
