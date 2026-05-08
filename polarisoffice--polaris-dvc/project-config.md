---
trigger: always_on
description: Read this before doing any non-trivial work in this repo. It describes
---

# CLAUDE.md — AI Agent working notes for polaris_dvc

Read this before doing any non-trivial work in this repo. It describes
the invariants that catch you out if you don't know them, the commands
that actually run the test matrix, and the exact places to edit when
you're wiring a new rule from upstream DVC.

This file is for AI agents (and humans doing AI-style drive-by edits).
It is intentionally denser than the user-facing `README.md`.

## What this project is, in one paragraph

polaris_dvc is a pure-Rust reimplementation of [hancom-io/dvc](https://github.com/hancom-io/dvc),
a Windows-only C++ DLL that validates HWPX (OWPML) documents against
JSON rule specs. The port keeps byte-level compatibility goals with
upstream — same rule-file schema (`sample/jsonFullSpec.json`), same
numeric error codes (`JID_*` from `Source/JsonModel.h`), same output
JSON field names and ordering — while running on macOS, Linux, and
`wasm32-unknown-unknown`. Upstream sources are vendored read-only at
`third_party/dvc-upstream/`; never edit them.

## Workspace layout

```
polaris_dvc/
├── crates/
│   ├── polaris-dvc-core/    rule engine, error codes, output model, Report
│   ├── polaris-dvc-hwpx/    pure-Rust HWPX (OWPML) parser (zip + quick-xml)
│   ├── polaris-dvc-format/  sniff() + parse() dispatch; reserves HWP5 slot
│   ├── polaris-dvc-cli/     `polaris-dvc` binary (DVC-compatible flags)
│   └── polaris-dvc-wasm/    wasm-bindgen shim (single `validate` entry)
├── tools/gen-jids/              regenerates jid_registry.rs from JsonModel.h
├── schemas/jsonFullSpec.json    upstream rule schema sample (unchanged copy)
├── testdata/golden/<nn>_.../    (doc.hwpx, spec.json, expected.json) triples
├── third_party/dvc-upstream/    READ-ONLY upstream snapshot, pinned commit
├── scripts/                     push.sh (PAT-aware git push wrapper)
└── docs/                        parity-roadmap, jid-registry, golden-tests, …
```

All five crates share dependencies via `[workspace.dependencies]` in the
root `Cargo.toml`. MSRV is pinned to 1.82 in `rust-toolchain.toml`.

## The three invariants you must not violate

### 1. JID values come from `JsonModel.h`, never from you

The 217 error codes live in `crates/polaris-dvc-core/src/jid_registry.rs`.
That file is **generated**, not hand-edited. It mirrors every
`#define JID_* N` in `third_party/dvc-upstream/Source/JsonModel.h`.

A drift test (`crates/polaris-dvc-core/tests/jid_registry_drift.rs`)
re-parses the upstream header on every `cargo test` and fails if the
committed registry's numeric values don't match. Bypass with
`POLARIS_ALLOW_JID_DRIFT=1` only while mid-edit.

The curated `jid` submodule in `crates/polaris-dvc-core/src/error_codes.rs`
exposes short-name aliases (`jid::TABLE_BGFILL_TYPE` →
`jid_registry::JID_TABLE_BGFILL_TYPE`). **Engine code uses the alias; the
alias refers to the registry.** Never hardcode an integer for an
ErrorCode — always go through `jid::`.

To regenerate the registry (only after `third_party/dvc-upstream/`
changes):

```sh
cargo run --manifest-path tools/gen-jids/Cargo.toml
```

### 2. `testdata/golden/` is authoritative for engine output

Every case directory holds `doc.hwpx`, `spec.json`, `expected.json`. The
`doc.hwpx` bytes are *reproducible from the in-Rust fixture template* in
`crates/polaris-dvc-core/tests/support/mod.rs`. On every `cargo
test` run, `tests/golden.rs` rebuilds each fixture and asserts:

- the freshly built `doc.hwpx` bytes equal the committed file (byte-exact),
- the engine's output JSON equals the committed `expected.json`.

If you change the fixture template or an engine checker, regenerate:

```sh
POLARIS_REGEN_FIXTURES=1 cargo test -p polaris-dvc-core --test golden
```

Review the diff before committing. Orphan directories (case dir with no
matching `Case` entry in `golden.rs`) fail a separate test — when
renaming a case, delete the old directory.

As of the XML-output change (Phase 8), each case directory also holds
`expected.xml`. The harness regenerates both JSON and XML expected
files from the same engine run — they stay semantically identical.

### 3. Output JSON shape matches upstream DVCOutputJson

The `ViolationRecord` struct in `crates/polaris-dvc-core/src/output.rs`
field-maps 1:1 to upstream `DVCOutputJson.cpp` writes. Field **order
matters** (serde_json uses `preserve_order`): upstream emits
`CharIDRef, ParaPrIDRef, errorText, PageNo, LineNo, ErrorCode, TableID,
IsInTable, IsInTableInTable, TableRow, TableCol, UseStyle, IsInShape,
UseHyperlink`. Conditional fields (`TableID`, `UseStyle`, …) are
included or omitted based on `OutputOption` — golden tests render under
`AllOption` so every field is present.

If you add a field, it must be because upstream emits it. Don't add
convenience fields.

## Daily commands

```sh
# Format + lint + test everything except WASM (which has a separate build):
cargo fmt --all
cargo clippy --workspace --exclude polaris-dvc-wasm --all-targets -- -D warnings
cargo test --workspace --exclude polaris-dvc-wasm

# WASM build (matches CI):
wasm-pack build crates/polaris-dvc-wasm --target web

# Regenerate golden fixtures after intentional engine/template change:
POLARIS_REGEN_FIXTURES=1 cargo test -p polaris-dvc-core --test golden

# Run the CLI directly:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PolarisOffice/polaris_dvc](https://github.com/PolarisOffice/polaris_dvc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
