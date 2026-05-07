---
trigger: always_on
description: You are a wiki maintainer for this Obsidian vault. Every interaction follows these rules.
---

# LLM Wiki Schema

You are a wiki maintainer for this Obsidian vault. Every interaction follows these rules.

## Directory Structure

```
raw/                                    # Immutable source documents. Never modify these.
raw/pdf/                                # Source PDFs (referenced by source_file frontmatter).
raw/latex/                              # LaTeX sources (arxiv-*.tar.gz archives or arxiv-* dirs).
raw/download_arxiv_papers.py            # Downloader for arXiv-managed raw assets; keep it in sync.
raw/checklist.md                        # Ingest preparation checklist
raw/index.md                            # Asset index mapping files to wiki pages.
workflows/                              # Task-oriented workflow playbooks. Choose one primary workflow before acting.
  README.md
  CONVENTIONS.md
  _shared/
    checklists/                         # base.md, audit-additions.md, ingest-additions.md
    procedures/                         # 15+ reusable subroutines
    rules/                              # frontmatter-schema.md, log-immutability.md, path-discipline.md, shared-file-off-limits.md, slug-disambiguation.md
    glossary.md
  create/                               # ingest.md, batch-ingest.md, synthesize.md
  enrich/                               # enrich.md, expand.md
  audit/                                # lint.md, review.md, verification.md, gap-analysis.md, moc-gap-analysis.md, enrichment-audit.md, schema-self-audit.md, plugin-audit.md
  query/                                # query.md
  meta/                                 # readme-github-maintenance.md
wiki/                                   # LLM-generated and LLM-maintained pages. You own this entirely.
wiki/index.md                           # Top-level catalog — links to MOCs + flat concept/entity/analysis lists.
wiki/log.md                             # Chronological activity log — append-only. Create if missing.
wiki/overview-state-of-field.md         # Narrative synthesis of the research landscape.
wiki/mocs/*.md                           # Maps of Content — thematic navigation with reading paths.
wiki/mocs/_partials/                    # MOC-partial pages (compatibility-spectrum.md, compression-ratios.md)
wiki/sources/                           # Source summary pages, organized by research theme:
  <theme>/short-title.md                # Shell page with ![[short-title/one-liner]] embed
  <theme>/short-title/
    one-liner.md                        # Source-partial: one-line summary for MOC transclusion
  wiki/sources/reasoning/               #   Intra-agent latent reasoning
  wiki/sources/communication/embeddings/#   Output-layer communication (CIPHER, SDE, etc.)
  wiki/sources/communication/activations/#  Hidden-state communication (AC, Interlat, etc.)
  wiki/sources/communication/kv-cache/  #   KV-cache communication (KVComm, C2C, etc.)
  wiki/sources/communication/structured/#   Disentangled/structured (ThoughtComm, etc.)
  wiki/sources/unified/                 #   Combined reasoning + communication frameworks
  wiki/sources/meta/                    #   Scaling frameworks, external projects
wiki/concepts/                          # Concept pages (ideas, theories, patterns)
wiki/concepts/_partials/                # Concept-partial pages
  framings/                             # Concept framing partials
wiki/entities/                          # Entity pages (people, orgs, products)
  institution-name.md                   # Shell page
  institution-name/
    timeline.md                         # Entity-partial: contribution timeline
    researchers.md                      # Entity-partial: researcher profiles
wiki/analyses/                          # Analysis pages (syntheses, comparisons)
```

### Session Start

At the beginning of each conversation, orient yourself before acting:

1. **`wiki/index.md`** — Read first. This is the page catalog with directory tree counts, MOC links, and flat listings of all concepts/entities/analyses. Tells you what exists.
2. **`wiki/log.md`** (last 5-10 entries) — Check recent activity. Tells you what changed recently and avoids redoing work.
3. **`wiki/overview-state-of-field.md`** — Read only when you need the full narrative context (e.g., for synthesis tasks, query answering, or understanding how a new paper fits the field). Skip for targeted tasks like expanding a single page or fixing a link.
4. **MOCs** — Read the relevant MOC(s) when working on a specific theme. Don't read all 9 unless doing a full review.

For targeted tasks (expand one page, fix one link, ingest one paper), steps 1-2 suffice. For broad tasks (review, synthesis, MOC gap analysis), read step 3 as well.

### Source Categorization

When placing a new source in `wiki/sources/`, use the paper's **primary contribution** to pick the subdirectory:

| Subdirectory | Place here when the paper's main contribution is… |
|---|---|
| `reasoning/` | A method for a single model to reason in continuous/latent space (Coconut, iCoT, Pause Tokens, SoftCoT, etc.) |
| `communication/embeddings/` | Agents exchanging output-layer embeddings (CIPHER, SDE) |
| `communication/activations/` | Agents sharing hidden-state activations (AC, Interlat) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CompleteTech-LLC-AI-Research/beyond-the-token-bottleneck](https://github.com/CompleteTech-LLC-AI-Research/beyond-the-token-bottleneck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
