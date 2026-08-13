---
trigger: always_on
description: Satsuma is a domain-specific language for source-to-target data mapping. The repository contains the language specification, a canonical example corpus, a tree-sitter parser, a multi-command CLI for structural extraction and validation, and a VS Code syntax highlighting extension.
---

# AGENTS.md

## Project Summary

Satsuma is a domain-specific language for source-to-target data mapping. The repository contains the language specification, a canonical example corpus, a tree-sitter parser, a multi-command CLI for structural extraction and validation, and a VS Code syntax highlighting extension.

All tooling is parser-backed. Downstream tools should be built on the tree-sitter CST and stable AST conventions rather than ad hoc text processing.

## Repository Layout

Current per-package test counts and the CLI command count are tracked in [`test-stats.json`](test-stats.json).

- `docs/developer/SATSUMA-V2-SPEC.md`: authoritative language specification (v2)
- `docs/product-owner/PROJECT-OVERVIEW.md`: product vision, motivation, and roadmap
- `SATSUMA-CLI.md`: CLI command reference
- `AI-AGENT-REFERENCE.md`: compact grammar and agent-oriented Satsuma guidance (v2) — also available via `satsuma agent-reference`
- `HOW-DO-I.md`: question-based index to all guides and conventions
- `docs/product-owner/ROADMAP.md`: deferred work items, ideas, and convention docs still to write
- `examples/`: canonical `.stm` examples and fixtures (v2 syntax)
- `archive/v1/`: archived v1 specification and examples — for historical reference only, not for new work
- `archive/features/`: delivered feature specs — for historical reference only. A spec moves here once its feature ships
- `features/`: active feature plans. See [ROADMAP.md](docs/product-owner/ROADMAP.md) for which are active and which have shipped — that page is the cross-feature view, and a spec's own `Status:` line is authoritative for its own feature
- `useful-prompts/`: self-contained system prompts for web LLMs (Excel-to-Satsuma, Satsuma-to-Excel)
- `skills/`: Agent Skills following the [agentskills.io](https://agentskills.io) standard (Excel-to-Satsuma conversion skill, Satsuma-to-Excel export skill)
- `scripts/`: utility scripts used during development
- `tooling/tree-sitter-satsuma/`: tree-sitter grammar, generated parser artifacts, and queries
- `tooling/satsuma-core/`: **the shared library every other package builds on** — parsing, extraction, validation, formatting, NL `@ref` resolution, and the single definition of coverage semantics. Logic that more than one consumer needs belongs here; see [Core vs Consumer Packages](#core-vs-consumer-packages)
- `tooling/satsuma-scenario-gen/`: **test-only** generator of semantic Satsuma scenarios shared by every package's property suites — builds scenarios as plain data, renders them to source, and states the ground truth that follows by construction. Must never depend on `@satsuma/core` (core's tests depend on it, so the reverse edge would be a cycle); the adapters that drive production pipelines live in each consumer's test tree
- `tooling/satsuma-cli/`: TypeScript CLI tool for workspace extraction, validation, and structural analysis
- `tooling/satsuma-lsp/`: editor-agnostic Language Server (semantic tokens, diagnostics, go-to-definition, find-references, completions, hover, rename, code lens, folding, document symbols); runnable standalone via `npx satsuma-lsp --stdio`
- `tooling/satsuma-viz-model/`: the VizModel protocol contract — the payload shape the LSP produces and the viz component consumes, defined once so neither can drift
- `tooling/satsuma-viz-backend/`: workspace indexing and VizModel assembly shared by the LSP and browser hosts; also computes the field coverage the payload carries (ADR-042)
- `tooling/satsuma-viz/`: the `satsuma-viz` Lit web component — overview graph and per-mapping detail view, embedded in the VS Code webview and the site playground
- `tooling/satsuma-viz-harness/`: Playwright harness for the viz component — headless Chromium regression suite, see [Viz harness Playwright tests](#viz-harness-playwright-tests)
- `tooling/vscode-satsuma/`: VS Code extension (LSP client, commands, webview panels) and TextMate grammar; delegates language intelligence to `satsuma-lsp`
- `tooling/integration-tests/`: **test-only** cross-consumer parity sweeps (CLI vs. the VizModel both the webview and the LSP consume) — the one place both sides are reachable without either package taking on a dependency its own architecture forbids

## Platform Lineage Entry Point

When reasoning about lineage across a multi-file data platform, look for a **platform entry point file** that uses `import` with namespace-qualified names to pull definitions from across the platform. This is the canonical entry point for platform-wide lineage traversal.

```satsuma
// platform.stm — the entry point
import { crm::customers, crm::orders } from "crm/pipeline.stm"
import { billing::invoices } from "billing/pipeline.stm"
import { warehouse::inventory } from "warehouse/ingest.stm"
```

Use `satsuma lineage --from <schema> platform.stm` to trace data flow through the platform. See `archive/features/15-namespaces/PRD.md` for the full namespace specification.

## Source of Truth

When making changes or answering questions about syntax, semantics, or supported constructs:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EqualExperts/satsuma-lang](https://github.com/EqualExperts/satsuma-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
