---
trigger: always_on
description: - Keep AGENTS.md and README.md in lockstep with any meaningful behavior or UX change; prune stale details to keep both compact.
---

# Repository Guidelines

## Maintenance & Documentation

- Keep AGENTS.md and README.md in lockstep with any meaningful behavior or UX change; prune stale details to keep both compact.
- Keep BENCHMARK.md aligned with benchmark suites, metrics, and snapshot tables. README should carry only the simplified benchmark summary and one link out to the full benchmark doc.
- Keep ARCHITECTURE.md aligned with the actual code structure. README should keep only the simplified architecture diagram and link out to the fuller developer-oriented architecture doc.
- Keep the architecture diagram in README.md aligned with the actual code at all times. Direction is always code -> diagram: update the diagram after any factual architecture change, never the other way around.
- Prioritize clear UX and documentation for a global audience (not just native English readers); keep CLI/report text transparent.
- Write all user-facing text for a global audience. Prefer short, plain English over dense prose, idioms, or culture-specific phrasing.
- When any doc, README section, benchmark note, or user-facing explanation makes claims about external models, tools, metrics, benchmarks, or research, cite primary sources inline near the exact claim and do not make the wording stronger than the source supports.
- In summaries, prefer one clear link to the deeper document instead of repeating the same link multiple times in nearby text.
- Keep README.md easy to scan and easy to understand. If something can be explained simply, write it simply.
- Keep README.md limited to the text users actually need. Push deeper methodology, edge cases, long rationale, and benchmark detail into separate docs instead of bloating the README.
- Treat README.md like product-facing documentation: short, clear, practical. Treat detailed reference material the same way as code structure: move it into focused files when depth is required.
- When you need external library/tool details, fetch official docs via the Context7 tool instead of ad-hoc searches.

## Tool Purpose & Problem Domain

**duplicalis** is a CLI tool for detecting duplicate and near-duplicate React components in large codebases. It addresses a common problem in component libraries: Developer A creates ComponentA, then Developer B creates ComponentB that is 80–95% similar in behavior, structure, and styling but not textually identical.

### Primary Goals

- Identify duplicate or near-duplicate React components using semantic analysis (embeddings + AST).
- Surface components that are almost identical and could be unified via props/configuration.
- Detect copy-paste patterns where reuse of the original would be preferable.

### Duplication Patterns Detected

The tool labels similarity matches with specific duplication classes:

| Label                  | Description                                                                      |
| :--------------------- | :------------------------------------------------------------------------------- |
| `prop-parameterizable` | Components differ mainly by prop values/sets; could be unified via props.        |
| `copy-paste-variant`   | Very high semantic + textual similarity; looks like copy with small edits.       |
| `logic-duplicate`      | Internal logic (hooks, handlers) is similar even if JSX/styles differ.           |
| `style-duplicate`      | Styles are nearly identical even if component code differs.                      |
| `forked-clone`         | High similarity but larger uneven differences; suggest canonical implementation. |
| `wrapper-duplicate`    | Many thin wrappers around the same base component.                               |

### Key Design Decisions

- **Imports treated as low-signal**: Import statements are normalized/summarized; they don't dominate similarity scores.
- **Component as primary unit**: One component = one chunk. Multi-component files are handled separately.
- **Semantic representation over raw text**: AST-based extraction preserves structure while ignoring irrelevant whitespace/comments.
- **Parser mode is extension-aware and decorator-aware**: Rust-backed SWC parsing keeps `.ts` files out of JSX mode to avoid angle-bracket TypeScript syntax being misread as JSX; `.tsx/.jsx/.js` keep JSX enabled, and parser decorators stay on so MobX-style fields and other decorated classes do not break scans.
- **Parser walk is single-pass**: Style imports and component metadata are collected in one SWC AST walk, and component source slices come from normalized node spans instead of repeated line splitting.
- **Parsed analysis is cached persistently**: Parsed component metadata plus semantic representations are stored in a file-aware cache and invalidated when source files or dependent stylesheets change.
- **Path-agnostic embeddings**: File-system paths are excluded from the embedded representation so similarity scores reflect code/style only, not folder layout.
- **Pluggable embedding backend**: Local model by default; remote API opt-in via env vars.
- **Remote trust boundary is explicit**: Remote mode sends component representations to the configured embeddings endpoint; local mode keeps analysis on-box.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pfrankov/duplicalis](https://github.com/pfrankov/duplicalis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
