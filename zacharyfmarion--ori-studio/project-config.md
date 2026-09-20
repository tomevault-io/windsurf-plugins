---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

**Ori Studio** is a workspace for origami design and analysis: a React/Tauri
frontend over a family of Rust and WebAssembly engines. The GitHub repository is
`zacharyfmarion/ori-studio`; some local clones and internal package names still
carry the older `treemaker-rs` / `@treemaker/*` naming, which is historical and
does **not** mean the product is TreeMaker.

The product is built largely from ports of existing community origami tools,
each of which is a distinct upstream with its own parity obligations:

- **Oriedita** — the crease-pattern editing kernel, and the bulk of the app's
  functionality (`crates/oristudio-cp*`).
- **TreeMaker 5.0.1** (Robert J. Lang) — turns a tree structure into a crease
  pattern (`crates/treemaker-*`). One tool within the app, not the app.
- **Box Pleating Studio** — box-pleating design kernel (`crates/oristudio-bp*`).
- **Flat-Folder** (Jason S. Ku) — flat-foldability and layer ordering
  (`crates/treemaker-flatfold`).

Original functionality is built on top of the ports — reference images beside
crease patterns, crease-pattern detection from images, the origami simulator —
but compatibility with the upstream tools and their file formats is a
priority.

When naming things in new code and docs, use **Ori Studio** for the product, and
an upstream's name only when referring to that specific tool, port, or vendored
source.

The top-level `README.md` is user-facing. Keep architecture notes, porting
discipline, implementation plans, and agent workflow details in developer docs
instead of turning the README into an engineering index.

## Repository layout

```text
crates/
  # Oriedita-derived crease-pattern editing (the core of the app)
  oristudio-cp/                   # Oriedita-compatible CP editing kernel
  oristudio-cp-wasm/              # wasm-bindgen bridge for the CP kernel
  oristudio-cp-compiler/          # Constraint-aware CP compiler core
  # Crease-pattern detection from images
  oristudio-cp-detect/            # Detection core types + oracle fixture plumbing
  oristudio-cp-detect-wasm/       # wasm-bindgen bridge for browser detection
  oristudio-cp-detect-inspector/  # Local API server for inspecting detect stages
  oristudio-cp-eval/              # Evaluation metrics for detection + compiler benchmarks
  # Box Pleating Studio port
  oristudio-bp/                   # BP-compatible headless kernel
  oristudio-bp-wasm/              # wasm-bindgen bridge for the BP kernel
  # TreeMaker 5.0.1 port
  treemaker-core/                 # Engine, file I/O, optimizers, geometry, CP generation
  treemaker-cli/                  # Headless command-line interface
  treemaker-wasm/                 # wasm-bindgen bridge for browser and Node
  # Shared / research
  treemaker-fold/                 # Generic FOLD data structures and geometry helpers
  treemaker-flatfold/             # Flat-foldability and layer-order solver (Flat-Folder port)
  oracle-tests/                   # Parity + fixture tests against every vendored oracle
apps/
  web/                            # React + Vite shared web frontend
  tauri/                          # Tauri v2 desktop shell wrapping apps/web
  cp-detect-architecture-inspector/  # Dev tool for CP-detection model architecture
packages/
  origami-simulator/              # TS origami simulator (port of Ghassaei's Origami Simulator)
tests/
  fixtures/                       # Shared model fixtures
  corpus/                         # External corpus harness notes; no private corpus files
tools/
  oracle/                         # C++ TreeMaker oracle build support
  oriedita-oracle/                # Oriedita (Java) parity oracle
  bp-studio-oracle/               # Box Pleating Studio parity oracle
  flat-folder-oracle/             # Flat-Folder parity oracle
third_party/
  oriedita/                       # Vendored Oriedita reference source
  treemaker-5.0.1/                # Vendored TreeMaker reference source
  box-pleating-studio/            # Vendored Box Pleating Studio reference source
  flat-folder/                    # Vendored Flat-Folder reference source
scripts/                          # Release, worktree setup, CP-detect tooling
research/                         # Investigation write-ups
implementation-plans/             # Per-feature plans (see "Implementation plans")
```

Note the `oristudio-*` / `treemaker-*` crate split is by **upstream lineage**,
not by importance: `treemaker-*` crates are the TreeMaker port plus some shared
FOLD/geometry code that predates the rename.

## Key architectural rules

### Porting discipline

Each ported subsystem has its own upstream, and that upstream — not our own
prior behavior — is the canonical behavioral reference:

| Subsystem | Vendored reference | Oracle |
| --- | --- | --- |
| CP editing (`oristudio-cp*`) | `third_party/oriedita` | `tools/oriedita-oracle` |
| TreeMaker (`treemaker-*`) | `third_party/treemaker-5.0.1` | `tools/oracle` (C++) |
| Box pleating (`oristudio-bp*`) | `third_party/box-pleating-studio` | `tools/bp-studio-oracle` |
| Flat folding (`treemaker-flatfold`) | `third_party/flat-folder` | `tools/flat-folder-oracle` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zacharyfmarion/ori-studio](https://github.com/zacharyfmarion/ori-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
