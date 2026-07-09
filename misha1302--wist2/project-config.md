---
trigger: always_on
description: This file is the mandatory behavior guide for AI agents working in this repository.
---

# AGENTS

This file is the mandatory behavior guide for AI agents working in this repository.

Read this file before editing code. If a local task conflicts with this file, this file wins unless the user explicitly updates the architecture rules.

## Documentation-first workflow

Markdown files are architectural source material, not optional notes.

Before non-trivial code changes, AI agents must identify the Markdown files that govern the task and the constraints they impose. Use `docs/DOCUMENTATION_INDEX.md` to find the relevant documents and `docs/CURRENT_ARCHITECTURE_STATUS.md` to distinguish current supported behavior from future or historical plans.

For small typo fixes, formatting-only edits, or obviously local mechanical changes, a short mental check is enough. Do not create heavy process for ordinary development.

For architecture, parser, runtime, module, capability, function, CLI, CI, public API, or documentation-smoke-check changes, the agent must preserve documentation intent and update Markdown together with code when behavior changes.

If code and Markdown disagree, report the conflict. Do not silently implement around the documentation.

A failing documentation check means documentation and implementation drifted. It does not mean the documentation check should be weakened.

Forbidden documentation fixes:

- deleting architectural documentation to make CI pass;
- replacing large documents with placeholders;
- adding path-based Markdown exclusions to smoke checks;
- restoring removed runtime functionality only because old docs reference it.

## Project identity

- **UniversalToolchain** is the primary product: a reusable, modular toolchain/framework for building and composing language runtimes.
- **Wist** is the reference language and proving ground in this repository, not the only architectural truth.
- Treat Wist-specific code and docs as examples of framework usage unless a file explicitly defines a Wist-only contract.

## Absolute architecture laws

Breaking these rules is a release-blocking architectural defect.

1. Generic framework layers must not hardcode dialect, profile, module, function, backend, or demo names.
2. Framework/core/runtime layers must not branch by shipped product profile names.
3. Runtime truth must flow only through dialect definition, compiled dialect slice, build plan, selected runtime plan, runtime configuration, and host/executor.
4. Capabilities/features are projection and explanation layers. They describe selected composition; they do not activate runtime behavior.
5. BasicCore must not depend on Wist, SafeMath, concrete feature modules, product profiles, or demo scenarios.
6. Function names belong to function providers/modules, not to parser, resolver, framework, or facade code.
7. Convenience APIs must be thin wrappers over existing composition/runtime paths. They must not create second parsers, second evaluators, second registries, or product-specific runtimes.
8. Product profiles must be ordinary dialect preset/configuration files, not framework runtime modes.
9. All provider discovery, catalogs, diagnostics, feature reports, schemas, CLI output, and overload resolution must be deterministic.
10. Architecture shortcuts are worse than missing features. If a feature cannot be implemented without a shortcut, leave it incomplete and document the limitation.
11. Syntax ownership is mandatory. Language syntax must not be recognized through ad hoc raw-source parsing outside the owning lexer/parser/AST/extractor pipeline.

## Syntax ownership law

`docs/SYNTAX_OWNERSHIP_RULES.md` is mandatory for all agents.

Language syntax must be recognized only by the owning lexer, parser, AST node creators, AST visitors, or syntax-specific extractors built from parser output.

Production validators, facades, resolvers, runtime wrappers, catalogs, CLI commands, optimizers, and convenience layers must consume structured syntax output. They must not rediscover language syntax from raw source text.

Forbidden production patterns include regular expressions, line splitting, substring checks, manual source scans, and one-off scanners used to recognize language constructs outside the owning syntax pipeline.

A missing parser, AST, or declaration model is not permission to parse raw source text locally. If the structured model does not exist yet, implement that model or leave the feature incomplete and document the limitation. A string-based syntax workaround is not an acceptable MVP.

## Single Responsibility Doctrine

Single Responsibility is mandatory, not a style preference.

- Parser parses syntax.
- Extractor extracts neutral declarations.
- Resolver resolves names, types, functions, and overloads.
- Projector projects capabilities/features for reports.
- Catalog describes available providers/descriptors/bindings.
- Runtime executes selected plans.
- Formatter formats diagnostics/output.
- Facade orchestrates existing workflows and remains thin.
- Module owns its own descriptors, bindings, and capability declarations.

Forbidden SRP violations:

- parser deciding runtime availability;
- resolver owning concrete module function names;
- capability projector activating runtime behavior;
- CLI implementing a separate runtime path;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Misha1302/Wist2](https://github.com/Misha1302/Wist2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
