---
trigger: always_on
description: Instructions for AI agents working in this repository.
---

# Clayers Agent Guide

Instructions for AI agents working in this repository.

## Development Workflow (MANDATORY)

**Spec first, code second.** Every change to clayers follows this order:

1. **Update the spec.** Before writing any code, update the relevant
   spec files in `clayers/clayers/` to describe what you are building.
   Add or modify prose, terminology, organization, relations, and
   LLM descriptions. If the feature is new, add a plan element.

2. **Validate the spec.** Run `cargo run -p clayers -- validate clayers/clayers/`
   to catch structural errors. Fix hashes, check drift.

3. **Implement the code.** Write the Rust (or other) code that realizes
   what the spec describes.

4. **Map spec to code.** Add or update artifact mappings linking spec
   nodes to the implementing code. Fix hashes on both sides.

5. **Iterate on quality.** After each change, check and improve:

   | Metric | Command | What to look for |
   |--------|---------|------------------|
   | **Coverage** | `cargo run -p clayers -- artifact --coverage clayers/clayers/` | Unmapped nodes, partial coverage, uncovered code ranges |
   | **Connectivity** | `cargo run -p clayers -- connectivity clayers/clayers/` | Isolated nodes, missing relations, low density |
   | **Drift** | `cargo run -p clayers -- artifact --drift clayers/clayers/` | Spec or artifact hashes out of sync |
   | **Comprehension** | `cargo run -p clayers -- schema schemas/` | Namespace declarations, layer completeness |
   | **Validation** | `cargo run -p clayers -- validate clayers/clayers/` | Structural errors, broken references |

   Iterate until coverage is full, connectivity has no isolated nodes,
   drift is clean, and validation passes.

6. **Commit.** Spec changes and code changes go in the same commit
   when they are part of the same logical unit.

**Why spec-first?** The spec is the source of truth. Writing the spec
first forces you to think through the design before coding. Artifact
mappings then provide machine-verifiable traceability between what was
specified and what was built. Drift detection catches divergence
automatically.

## Quick Reference

```bash
# Validate a spec
cargo run -p clayers -- validate clayers/clayers/

# Check for drift (read-only, exit 0=clean, 1=drift)
cargo run -p clayers -- artifact --drift clayers/clayers/

# Coverage analysis
cargo run -p clayers -- artifact --coverage clayers/clayers/

# Coverage for a specific code path
cargo run -p clayers -- artifact --coverage clayers/clayers/ --code-path src/

# Connectivity analysis
cargo run -p clayers -- connectivity clayers/clayers/

# Export schemas as RNC
cargo run -p clayers -- schema schemas/

# Export a single layer
cargo run -p clayers -- schema schemas/ --layer pr

# Fix spec-side hashes (after editing prose/terminology/etc.)
cargo run -p clayers -- artifact --fix-node-hash clayers/clayers/

# Fix artifact-side hashes (after editing code)
cargo run -p clayers -- artifact --fix-artifact-hash clayers/clayers/

# XPath query (XPath first, then path)
cargo run -p clayers -- query '//trm:term/trm:name' clayers/clayers/ --text
```

All commands accept a directory (searches recursively) or a single XML file.

## Directory Layout

```
clayers/
  AGENTS.md              # This file
  crates/                # Rust workspace
    clayers/             # CLI binary
    clayers-spec/        # Spec-aware logic (validate, drift, coverage, etc.)
    clayers-xml/         # Domain-agnostic XML utilities (C14N, RNC, catalog)
  schemas/               # XSD 1.1 schemas (one per layer)
    catalog.xml          # OASIS XML Catalog (namespace-to-file mapping)
    spec.xsd             # Root element, annotation markers
    index.xsd            # File manifest
    revision.xsd         # Named snapshots
    prose.xsd            # DITA-style writing elements
    terminology.xsd      # Controlled vocabulary
    organization.xsd     # Topic typing (concept, task, reference)
    relation.xsd         # Typed semantic links
    decision.xsd         # Decision records
    source.xsd           # External references and citations
    plan.xsd             # Implementation plans and acceptance criteria
    artifact.xsd         # Code traceability + drift detection
    llm.xsd              # Machine-readable descriptions
  clayers/              # Specification instances (knowledge base)
    clayers/             # Self-referential spec (describes the format)
      index.xml          # Manifest: lists all files in the spec
      revision.xml       # Revision layer ("draft-1")
      overview.xml       # Layers, vocabulary, file structure
      validation.xml     # Combined documents, cross-layer integrity
      traceability.xml   # Artifact mapping, drift, hash tooling
      schema.xml         # XSD design, extensibility
      descriptions.xml   # LLM layer descriptions
  examples/              # Example specifications
    payment-processing/
  clayers-harness.py     # Empirical harness for spec quality measurement
  experiments/           # Harness experiment configurations
```

## Concepts

**Spec**: a set of XML files sharing an index. Each file is a
`<spec:clayers>` document mixing content from multiple layers.

**Layer**: a semantic namespace (prose, terminology, organization, relation,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CognitiveLayers/clayers](https://github.com/CognitiveLayers/clayers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
