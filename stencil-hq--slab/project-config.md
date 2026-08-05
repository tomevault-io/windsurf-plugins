---
trigger: always_on
description: Use this file to navigate Slab safely. Prefer a small, targeted check over a
---

# Repository Guidelines

Use this file to navigate Slab safely. Prefer a small, targeted check over a
workspace-wide rebuild unless a change crosses compiler, kernel, or generated-artifact boundaries.

## Project Overview

Slab is a design language for agents. Rust compiles `.slab` source to **SLIR**
(Protobuf in a raw-Snappy envelope); one hand-maintained Rust kernel evaluates
layout, responsive conditions, animation, editing, hit-testing, focus, and
events. Thin native, terminal, browser/WASM, and static-rendering drivers paint
the same kernel output.

**Core invariant:** there is one solver in `crates/slab-kernel/`. Do not add
platform-specific layout or interaction behavior to a driver when it belongs in
the kernel.

## Architecture & Data Flow

```text
.slab source
  -> crates/slab-syntax       lex, parse, format, diagnostics
  -> crates/slab-compile      expand components/imports, embed assets, lower
  -> crates/slab-slir         encode/decode/dump the binary document
  -> crates/slab-kernel       instantiate, lay out, dispatch, produce Frame/Scene
  -> slab-cli | slab-tui | slab-native | slab-wasm | clients/web
```

- `crates/slab-cli/src/main.rs` is the reference command surface: `check`,
  `build`, `dump`, `fmt`, `render`, `conformance`, `drive`, `lsp`, and `gen`.
- The kernel accepts decoded SLIR plus environment, params, and retained state;
  clients translate host input into kernel events and paint returned frames.
- `slab-wasm` exposes compiler and renderer paths to JavaScript. `clients/web/`
  renders its kernel frames as custom elements; `site/` is the live WASM
  playground.
- `packages/dslab/` speaks the newline-delimited Slab Drive Protocol (SDP),
  either over TCP or a spawned `slab drive` process.
- Determinism is intentional: kernel arithmetic, output quantization, and
  ordering must stay host-independent so native and WASM conformance goldens
  remain byte-identical.

## Key Directories

| Path | Purpose |
| --- | --- |
| `crates/slab-syntax/` | Lexer, parser, AST spans, formatter, diagnostics. |
| `crates/slab-compile/` | Semantic compilation, expansion, SLIR lowering, static render and code generation. |
| `crates/slab-slir/` | Normative SLIR structures, binary reader/writer, canonical dump. |
| `crates/slab-kernel/` | Shared deterministic runtime: layout, scene, input, editing, animation. |
| `crates/slab-cli/` | Native CLI, conformance runner, Drive Protocol server. |
| `crates/slab-{tui,native,lsp,wasm}/` | Terminal, wgpu, language-server, and WASM host adapters. |
| `clients/web/` | `@stencil-hq/wslab`: `SlabElement`, frame decode, DOM/canvas painter, browser WASM glue. |
| `packages/slab/` | `@stencil-hq/slab` WASM-backed npm CLI. |
| `packages/dslab/` | `@stencil-hq/dslab` typed SDP client and `dslab` CLI. |
| `site/` | CodeMirror playground, preview, inspector, and design-mode UI. |
| `conformance/` | Shared cases, traces, manifest, and byte-exact expected outputs. |
| `spec/` | Normative language, SLIR, frame API, and platform-support specifications. |
| `tree-sitter-slab/` | Editor grammar and corpus/highlight tests. |
| `scripts/`, `tools/` | Packaging, site/dev servers, conformance, and browser E2E runners. |

## Development Commands

```sh
# Bootstrap
bun install

# Usual validation layers
just check          # rustfmt, clippy -D warnings, Biome, tree-sitter checks
just test           # cargo test --workspace
just conformance    # native and WASM cases against checked-in goldens
just freshness      # regenerate in a temp snapshot and reject drift
just ci             # check + test + conformance + freshness

# Generation and packages
just gen             # refresh all committed derived artifacts
just pack            # build @stencil-hq/wslab, slab, and dslab distributions
bun scripts/pack-e2e.ts
just editors         # build the VSCode .vsix and Zed .tar.gz into out/editors

# Browser/site work
just site            # bundle site/dist
just dev             # local site server with live reload
just dev-wasm        # refresh the playground WASM compiler after a WASM change
just web-e2e         # Playwright web-component integration tests
```

Use focused commands while iterating:

```sh
cargo test -p slab-kernel
cargo test -p slab-compile
cargo run -q -p slab-cli -- check examples/10-settings.slab
cargo run -q -p slab-cli -- render examples/10-settings.slab -o /tmp/settings.png
cargo run -q -p slab-cli -- dump path/to/document.slir
bun test packages/dslab/test/drive.test.ts
```

## Code Conventions & Common Patterns

### Architecture patterns

- Keep compiler, binary-format, runtime, and host concerns in their existing
  layers. A syntax or semantic change normally flows through `slab-syntax` and
  `slab-compile`; a layout or input behavior change belongs in `slab-kernel`.
- Preserve the shared-kernel model. Drivers should adapt input/output, not
  independently solve layout, focus, animation, or text editing.
- Treat frame/scene output as a deterministic contract. Avoid unordered output,
  host-dependent metrics, lossy numeric changes, and platform-only fallbacks.
- State is retained by kernel instances. Apply parameter/environment/input
  changes through the existing instance and dispatch APIs rather than recreating
  ad-hoc state in a renderer.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stencil-hq/slab](https://github.com/stencil-hq/slab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
