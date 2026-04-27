---
trigger: always_on
description: <!-- vigiles:sha256:36233841867c0b53 compiled from CLAUDE.md.spec.ts -->
---

<!-- vigiles:sha256:36233841867c0b53 compiled from CLAUDE.md.spec.ts -->

# CLAUDE.md

## Positioning

vigiles compiles `.spec.ts` files to instruction files (CLAUDE.md, AGENTS.md, or any markdown target). The spec is the source of truth. The markdown is a build artifact. Nobody else does this — other tools lint markdown after the fact. vigiles eliminates the problem at the source.

The linter cross-referencing engine is the core moat: `enforce("@typescript-eslint/no-floating-promises")` verifies the rule exists AND is enabled in your linter config. Same for ESLint, Ruff, Clippy, Pylint, RuboCop, and Stylelint. No other tool resolves rules against 6 linter APIs.

`generate-types` is the second moat: scans all 6 linter APIs, package.json, and project files to emit a `.d.ts` with type unions. The TS compiler then PROVES references are valid at authoring time — typos become type errors, not runtime surprises.

vigiles does NOT do architectural linting. Use ast-grep, Dependency Cruiser, Steiger, or eslint-plugin-boundaries for that. vigiles can reference their rules via `enforce()`.

## Architecture

Two rule types in specs:

- `enforce()` — delegated to external tool (linter, ast-grep, dependency-cruiser). vigiles verifies the rule exists and is enabled.
- `guidance()` — prose only, compiles to `**Guidance only**` in markdown.

Architectural linting (file pairing, import boundaries, AST patterns) belongs in external tools — reference them via `enforce()`.

Template literal types ensure linter names (`eslint/`, `ruff/`, etc.) are type-safe. Branded types (`VerifiedPath`, `VerifiedCmd`, `VerifiedRef`) distinguish verified references from raw strings.

Compilation: spec.ts → compiler reads spec, validates references (file paths via existsSync, npm scripts via package.json, linter rules via linter APIs), generates markdown with SHA-256 integrity hash.

Core modules: `src/spec.ts` (types + builders), `src/compile.ts` (compiler), `src/linters.ts` (6-linter cross-referencing engine), `src/generate-types.ts` (type generator), `src/proofs.ts` (proof algorithms for self-evolving specs), `src/evolve.ts` (evolution engine).

## Key Files

- `src/spec.ts` — Type system and builder functions (enforce, guidance, claude, skill, file, cmd, ref)
- `src/compile.ts` — Compiler: spec → markdown with SHA-256 hash, linter verification, reference validation
- `src/linters.ts` — Linter cross-referencing engine (ESLint, Stylelint, Ruff, Clippy, Pylint, RuboCop)
- `src/generate-types.ts` — Type generator: scans linters/package.json/filesystem → emits .d.ts
- `src/cli.ts` — CLI: init, compile, audit (3 primary commands + generate-types plumbing)
- `src/inline.ts` — Inline-mode parser: `<!-- vigiles:enforce ... -->` comments in markdown for gradual adoption
- `src/action.ts` — GitHub Action wrapper
- `src/spec.test.ts` — Spec + compiler test suite (node:test)
- `src/validate.test.ts` — Validation test suite (node:test)
- `src/cli.test.ts` — CLI integration + E2E test suite (node:test)
- `src/integrity.ts` — Integrity check: SHA-256 hash verification for compiled markdown (detects hand-edits)
- `src/sidecar.ts` — Per-spec sidecar manifests at .vigiles/<target>.inputs.json, used by session audit
- `src/sidecar.test.ts` — Tests for sidecar manifests, per-file hashes, and integrity check
- `src/coverage.ts` — Spec coverage analysis: linter rule coverage + npm script coverage with configurable thresholds
- `src/coverage.test.ts` — Coverage test suite (node:test)
- `src/session.ts` — Post-session audit: git diff analysis against spec surface area
- `src/session.test.ts` — Session audit test suite (node:test)
- `src/hash.ts` — Shared SHA256Hash branded type and assertNever exhaustive check helper
- `src/test-utils.ts` — Shared test utilities: makeTmpDir, makeSpec, cleanupTmpDir, initGitRepo
- `src/types.ts` — Shared types: RulesConfig, VigilesConfig, FreshnessMode, CoverageThresholds
- `src/proofs.ts` — Deterministic proof algorithms (monotonicity lattice, NCD, Bloom filter, Merkle DAG, fixed-point, property testing)
- `src/evolve.ts` — Evolution engine: mutation operators, fitness function, proof-gated selection
- `src/proofs.test.ts` — Proof system + evolution engine tests (node:test)
- `CLAUDE.md.spec.ts` — This file — the source of truth for CLAUDE.md
- `examples/SKILL.md.spec.ts` — Example SKILL.md spec
- `research/adoption-strategy.md` — Adoption strategy: zero-config setup, progressive enforcement, agent workflows
- `research/competitive-landscape.md` — Competitive landscape: rule-porter, rulesync, vibe-cli, Ruler
- `research/executable-specs.md` — Design doc: executable spec system
- `research/feature-ideas.md` — Feature ideas: plugin API, custom rules, exhaustive coverage
- `research/ai-code-quality.md` — Research: AI code quality patterns
- `research/self-evolving-specs.md` — Design doc: self-evolving spec system (proofs, Merkle history, evolution engine)
- `research/code-search-for-agents.md` — Research: code search approaches (grep vs embeddings vs AST-grep)
- `research/runtime-enforcement.md` — Research: spec-derived runtime enforcement via hooks, skill contracts, session audit
- `research/architecture-platform.md` — Research: architecture-aware agent platform (FSD/DDD/hexagonal presets, meta-validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zernie/vigiles](https://github.com/zernie/vigiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
