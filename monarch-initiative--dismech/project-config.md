---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Disorder Mechanisms Knowledge Base (dismech)** - a LinkML-based knowledge base storing disease pathophysiology information. It combines:
1. A LinkML schema defining the data model (`src/dismech/schema/dismech.yaml`)
2. A knowledge base of disorder YAML files (`kb/disorders/*.yaml`)
3. HTML rendering for browsable disorder pages (`pages/disorders/*.html`)

## Skills

Claude Code skills are available in `.claude/skills/`:

- **dismech-terms**: Use when adding ontology term annotations (HPO phenotypes, CL cell types, GO processes, MAXO treatments). Covers term lookup with OAK, specificity guidelines, and validation.
- **dismech-references**: Use when validating/repairing evidence references. Ensures snippets match PubMed abstracts and catches AI hallucinations.

## Key Commands

```bash
# Install dependencies
just install

# Run all QC checks (validation + term validation)
just qc

# Validate all disorder YAML files against schema
just validate-all

# Validate a single disorder file
just validate kb/disorders/Asthma.yaml

# Validate ontology term references in schema (anti-hallucination check)
just validate-terms

# Run pytest tests
just pytest-all

# Run a single test
uv run pytest tests/test_data.py -k "test_name" -v

# Generate HTML pages for all disorders
uv run python -m dismech.render --all

# Generate HTML for a single disorder
uv run python -m dismech.render kb/disorders/Asthma.yaml

# Fetch and cache a reference (PMID, DOI, NCT) — NEVER create cache files manually
just fetch-reference PMID:12345678

# Validate references for a single file
just validate-references kb/disorders/Asthma.yaml

# List all available commands
just --list
```

## Architecture

### Schema (`src/dismech/schema/dismech.yaml`)
- LinkML schema defining Disease, Pathophysiology, Phenotype, EvidenceItem, etc.
- Uses ontology term bindings (HP, GO, GENO, MONDO, MAXO, etc.) with `meaning` fields
- Dynamic enums with `reachable_from` constraints for ontology validation
- Descriptor classes (PhenotypeDescriptor, CellTypeDescriptor, TreatmentDescriptor) bind entities to ontology terms

### Knowledge Base (`kb/disorders/`)
- One YAML file per disorder (55 total)
- Each file validates against the `Disease` class in the schema
- Evidence items require PMID references
- Ontology term bindings for phenotypes, cell types, biological processes, and treatments

### Ontology Configuration (`conf/oak_config.yaml`)
Maps ontology prefixes to OAK adapters for term validation:
- HP, CL, GO, MONDO, UBERON, CHEBI, GENO, HGNC → `sqlite:obo:<name>`
- MAXO (Medical Action Ontology) for treatment terms
- NCIT (NCI Thesaurus) for cancer/treatment concepts

### CURIE Prefix Casing

HGNC gene CURIEs use **lowercase** `hgnc:` prefix in this repo (e.g., `hgnc:746`, not `HGNC:746`). This is the canonical form that passes term validation. Do not flag lowercase `hgnc:` as an error in reviews.

### HTML Rendering (`src/dismech/render.py`)
- Jinja2 templates in `src/dismech/templates/`
- Generates browsable HTML pages in `pages/disorders/`
- Links ontology terms to external browsers (HPO JAX, MONDO Monarch, OLS, etc.)

### Scripts (`scripts/`)
- `add_maxo_terms.py`: Batch-add MAXO treatment terms to disorder files

### Structured-Database Sources (`src/dismech/structured_sources/`)
- Framework for ingesting structured knowledge bases (Orphanet today; OMIM /
  MONDO / HGNC pluggable) into `references_cache/` as line-oriented markdown
- Flagship: `OrphanetSource` — pre-caches all 8,823 leaf disorders from
  Orphadata XML so curators can cite `ORPHA:<code>` and quote individual rows
  (definition, prevalence, HPO phenotypes, gene-disease, xrefs)
- See "Structured-Database Reference Sources" below

### Validation Stack
- **linkml-validate**: Schema conformance checking
- **linkml-term-validator**: Validates ontology term references against authoritative sources (critical for catching AI hallucinations)
- **linkml-reference-validator**: Validates that quoted text appears in cited references

## Important Patterns

### Mechanism Modules

Mechanism modules (`kb/modules/`) define conserved pathological processes that recur across
multiple disorders (e.g., the fibrotic response). A module uses the **same schema** as a
regular dismech Disease entry — it has `pathophysiology` nodes with cell types, biological
processes, evidence, and causal edges (`downstream`).

**How conformance works:**

Individual disorder entries declare that a pathophysiology node conforms to a module node
using the `conforms_to` slot:

```yaml
# In kb/disorders/Liver_Cirrhosis.yaml
pathophysiology:
- name: Hepatic Stellate Cell Activation
  conforms_to: "fibrotic_response#Mesenchymal Cell Activation"
  cell_types:
  - preferred_term: Hepatic Stellate Cell
    term:
      id: CL:0000632
      label: hepatic stellate cell
  biological_processes:
  - preferred_term: TGF-beta Receptor Signaling
    term:
      id: GO:0007179
      label: transforming growth factor beta receptor signaling pathway
    modifier: INCREASED
```

**Key principles:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monarch-initiative/dismech](https://github.com/monarch-initiative/dismech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
