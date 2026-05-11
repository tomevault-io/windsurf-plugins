---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`mogen` (brand: **MoGen**) is a Rust CLI that compiles a compact declarative DSL (`.mog` files)
into glTF 2.0 `.glb` assets. It is designed as the deterministic backend of an LLM-driven 3D
generation pipeline: an LLM writes high-level structured scenes, `mogen` expands them into real
geometry. Primary engine target is Godot 4.x, but glTF output must remain spec-compliant.

The desktop GUI is **MoGen Studio** (crate `mogen-studio`, binary `mogen-studio`).

## Commands

```sh
./scripts/build-release.sh                  # cargo build --release --workspace
./scripts/run-tests.sh                      # cargo test --workspace
./scripts/run-mogen.sh <subcommand> …       # cargo run --release --bin mogen -- …
./scripts/run-studio.sh                     # cargo run --release -p mogen-studio

# one package / one test
cargo test -p mogen-dsl
cargo test -p mogen-geom csg::tests::difference_basic -- --exact

# common CLI flows
./scripts/run-mogen.sh build    examples/chair.mog --out chair.glb
./scripts/run-mogen.sh check    examples/chair.mog [--json]      # validate; exits non-zero on errors
./scripts/run-mogen.sh parse    examples/chair.mog               # dump AST
./scripts/run-mogen.sh dump-scene examples/chair.mog --json      # dump lowered SceneGraph
./scripts/run-mogen.sh inspect  chair.glb                        # read back + summarize a GLB
./scripts/run-mogen.sh generate "a wooden stool" --out stool.glb     # Gemini-driven; needs GEMINI_API_KEY
./scripts/run-mogen.sh modify   examples/chair.mog "make legs taller" # LLM edit of an existing .mog
./scripts/run-mogen.sh bench    --prompts benches/prompts.txt         # ≥80% success gate

# GUI
./scripts/run-studio.sh                     # MoGen Studio desktop app
```

`generate`/`modify`/`bench` read `GEMINI_API_KEY` from env (or take `--api-key`). `generate` and
`modify` embed a `// mogen-generate seed=…` header so rebuilds are reproducible.

## Architecture

Cargo workspace under `crates/`. The compile pipeline is a strict layering; keep cross-crate
dependencies pointing in one direction:

```
mogen-dsl  ──parse──►  AST  ──validate_ast──►  lower  ──►  mogen-core::SceneGraph
                                                              │
                                                              ├──validate_graph──►
                                                              │
                                                              └──mogen-export──►  .glb
```

- **mogen-core** — pure data: `SceneGraph` (arena of `SceneNode` with parent/child ids),
  `Transform` (glam-based TRS), `Mesh`, `Material`, `Connector` (pos + quat + tag + optional
  radius), `Joint`/`Clip`/`Track` for animation, `Skin` for skinning, `Diagnostic`/`Severity`/
  `Span` for error reporting, and `Aabb` helpers. No I/O, no parsing.
- **mogen-dsl** — pest grammar (`grammar.pest`), AST (`ast.rs`), parser, and the lowering
  pipeline that turns AST → `SceneGraph`. Lowering is split across files by concern:
  `module.rs` (module declarations + `use` expansion with `$param` substitution, recursion
  detection, expansion cache), `lower.rs` (geometry/materials/transforms/CSG/mirror/array),
  `attach.rs` (connector frame alignment), `anim_lower.rs` (joints, clips, procedural
  templates), `skin_lower.rs` (skeletons, bones, automatic weight binding). Every AST node
  preserves pest spans — diagnostics depend on this.
- **mogen-validate** — two-phase validator. `validate_ast` runs on the parsed AST (unknown
  kinds, missing/typo attrs, unknown references). `validate_graph` runs on the lowered
  `SceneGraph` (topology, weights summing to 1, skeleton-root ancestry, etc.). Both produce
  `Diagnostic` values; `render_human` uses `codespan-reporting`, `render_json` emits the
  line-delimited format the LLM repair loop consumes.
- **mogen-geom** — primitives (`box`, `cylinder`, `cone`, `sphere`, `capsule`, `torus`,
  `prism`, `pyramid`, `disc`, `icosphere`, `rounded_box`, `plane`, `quad`), CSG via `csgrs`
  (`union`/`difference`/`intersect` with many-arg variants), mesh transforms, and cleanup
  (vertex welding, degenerate-tri cull, normal recomputation). CSG ops call `clean_csg_output`
  to give the exporter a watertight mesh.
- **mogen-anim** — procedural animation templates (`spin`, `open_close`, `wave`, `flap`,
  `idle`) that build `Clip`s. v1 emits glTF node-transform tracks only; skinning lives in
  `mogen-core::Skin` + exporter and is driven by the same joint nodes.
- **mogen-export** — hand-rolled GLB writer (JSON chunk + BIN chunk). Uses `serde_json` for
  the JSON side; buffer packing is manual via `to_le_bytes`. Writes PBR materials, animation
  channels, and skins (`skins[]`, `JOINTS_0`/`WEIGHTS_0` accessors, `node.skin` refs). The
  `asset.generator` field in the output GLB is `"MoGen"`. `options.rs` defines
  `ExportOptions` (`include_animations`, `include_textures`, `merge_sibling_meshes`) consumed
  by `write_glb_with_options`; `merge.rs` is an optional pre-export pass that CSG-unions
  same-material, non-skinned sibling leaf meshes into one node (preserves hierarchy,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krazyjakee/MoGen](https://github.com/krazyjakee/MoGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
