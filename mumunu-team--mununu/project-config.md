---
trigger: always_on
description: <!-- last reviewed: 2026-05-16 -->
---

<!-- last reviewed: 2026-05-16 -->

# CLAUDE.md — Project Instructions for Claude Code

> **How to read this file.** This file is loaded into context for every Claude Code session in this repo. Sections tagged `[RULE]` are binding constraints, not suggestions — violating them blocks merge. Sections tagged `[REFERENCE]` and `[CONTEXT]` are background. When a task touches a subsystem documented under `docs/`, follow the link rather than guessing from prior knowledge.

## Contents

1. [Project Overview](#project-overview) `[CONTEXT]`
2. [Workspace Structure](#workspace-structure) `[REFERENCE]`
3. [CI Requirements](#ci-requirements) `[RULE]`
4. [Git Identity](#git-identity) `[RULE]`
5. [Surface Parity](#surface-parity) `[RULE]`
6. [Soundness Guarantees](#soundness-guarantees) `[RULE]`
7. [Abstraction Guidelines](#abstraction-guidelines) `[RULE]`
8. [Documentation Traceability](#documentation-traceability) `[RULE]`
9. [Claims Integrity](#claims-integrity) `[RULE]`
10. [Reproducers Before Fixes](#reproducers-before-fixes) `[RULE]`
11. [Adapter / Emitter Capability Use](#adapter--emitter-capability-use) `[RULE]`
12. [Git Operations & Destructive Commands](#git-operations--destructive-commands) `[RULE]`
13. [Code Reuse, Dead Code, Performance, Security](#code-reuse-dead-code-performance-security) `[RULE]`
14. [Test the Layer the Bug Lives In](#test-the-layer-the-bug-lives-in) `[RULE]`
15. [Available Agents and Skills](#available-agents-and-skills) `[REFERENCE]`
16. [Private Files Policy](#private-files-policy) `[RULE]`
17. [Reasoning & Recommendation Honesty](#reasoning--recommendation-honesty) `[RULE]`
18. [Engine Specificity in Proposals](#engine-specificity-in-proposals) `[RULE]`
19. [Reference Docs Index](#reference-docs-index) `[REFERENCE]`

---

## Project Overview

Mununu is a formal verification tool for analyzing and synthesizing controllers for reactive systems modeled as Compositional Labeled Transition Systems (CLTS). It includes mu-calculus formula evaluation, controller synthesis, LTL pattern support, state variable abstraction, and a DSL (CTXDSL) for specifying concurrent systems.

See `docs/architecture/` for the three-layer model (extraction → adaptation → verification).

## Workspace Structure

Cargo workspace, three crates:

```
crates/
├── mununu-core/      (lib: verification engine, adapters, composition, IR, shared types)
├── mununu-cli/       (bin: `mununu` — CLI + HTTP API server)
└── mununu-extract/   (bin: `mununu-extract` — tree-sitter AST extraction)
```

Edition 2024. Toolchain pinned by `rust-toolchain.toml`; see [`docs/toolchain.md`](docs/toolchain.md) for the bump procedure and current clippy-lint compatibility notes.

## CI Requirements

The gate is `make ci` — the same command runs locally, in pre-commit hooks, and in GitHub Actions. If a contributor cannot reproduce a CI failure with `make ci`, the contract is broken.

```bash
make ci    # = cargo fmt --check + cargo clippy -D warnings + cargo test --workspace
```

For CI-exact reproduction inside the pinned dev container, see [`docs/dev-container.md`](docs/dev-container.md). For finer-grained cargo invocations (per-crate, specific tests, benches, server, extractor), see [`docs/build-recipes.md`](docs/build-recipes.md). Install pre-commit hooks via `./scripts/setup-hooks.sh` — the hook runs `cargo fmt --check`, `cargo clippy`, and `cargo test`.

The `security-audit` CI job runs `cargo audit`. The `dependency-check` job is non-blocking.

### SVA-verification e2e validation (slang) `[REFERENCE]` (added 2026-06-27)

The SVA front-end (`mununu sv extract-sva` / `mununu sv verify-auto`) shells out to **slang** (SystemVerilog elaboration), **sv2v**, and **yosys**. Per the subprocess-tools-are-not-bundled policy, these are NOT in `mununu-dev`, so the slang-gated end-to-end tests are `#[ignore]`d and do not run in `make ci`. Their mechanisms are covered by non-ignored unit tests; the `#[ignore]` tests validate the full chain when the tools are present.

**Run them in the `mununu-sva` image** (`docker/Dockerfile.sva` — extends `mununu-dev` with ONE tag-pinned public oss-cad-suite release — yosys + the yosys-slang plugin, slang, Verilator, btormc, pono, cvc5 — plus sv2v. The pins are asserted against the binaries at build time and exposed as image labels; `docker/README.md` shows how to read them. The tag is the SAME one rosf and monono pin, so the lift, the synth-check and the bitstream come from one compiler):

```bash
docker build -f docker/Dockerfile.dev -t mununu-dev .     # once (or on toolchain bump)
docker build -f docker/Dockerfile.sva -t mununu-sva .     # once
docker volume create mununu-target                        # once (warm cargo cache)
docker run --rm -v "$(pwd)":/work -v mununu-target:/cargo-target \
  mununu-sva cargo test -p mununu-core --lib --all-features e2e_ -- --ignored --nocapture
```

**For the WHOLE sweep, use `make e2e`** (`scripts/e2e-sweep.sh`), which runs one test per
process. A test that ABORTS rather than fails — a stack overflow, or a panic while unwinding
an exhausted BDD arena — does not unwind, so in a single shared libtest process it kills the
run: no summary line, no failure list, and every later test silently skipped. That is how the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mumunu-team/mununu](https://github.com/Mumunu-team/mununu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
