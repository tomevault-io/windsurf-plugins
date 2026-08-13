---
trigger: always_on
description: rust-doctor is one Rust crate (edition 2024, rustc 1.95 or later): a local-first
---

# AGENTS.md

rust-doctor is one Rust crate (edition 2024, rustc 1.95 or later): a local-first
CLI that inspects a trusted Cargo workspace with curated Clippy lints and native
detectors, then scores it out of 100. `src/lib.rs` exposes
`inspect(InspectRequest) -> Report`, `src/main.rs` is the CLI on top of it, and
`npm/rust-doctor/` is a Node launcher for the released binary.

The catalog holds 62 rules across five producers, and a rule's id prefix names
its producer: `clippy::*` (37 curated lints, `Producer::Clippy`),
`rust_doctor::source::*` (2, `SourceKernel`, error stage `source`),
`rust_doctor::cargo::*` (11, `CargoHealth`, stage `dependencies`, which judges
the manifests and `.cargo/config.toml`), `rust_doctor::structure::*` (9,
`Structure`, stage `structure`) and `rust_doctor::repo::*` (3, `Repo`, stage
`repo`, the only pass that reads outside the Cargo model, enumerating through
`git ls-files`). `validate_catalog`
refuses any other prefix, and a pass that fails degrades to a complete report
carrying a `ReportError` at its stage with the authoritative flag dropped.

## Trust boundary

Inspecting a workspace runs `cargo clippy` inside it, and Cargo executes that
workspace's `build.rs` files and procedural macros. Inspect trusted local paths
only. Never scan a path taken from an issue, a bug report, or any source outside
this repository. Clippy is the only pass that compiles anything: the four
native producers parse source text, read manifests or ask git what it tracks,
and build nothing.

The tool never reaches the network, never uploads, never emits telemetry. Keep
it that way: no HTTP client, no analytics dependency, no phone-home. `--json`
reports stay workspace-relative, with no absolute path, no environment variable,
and no user data.

## Commands

| Goal | Command |
|---|---|
| Build | `cargo build --release` |
| Test | `cargo test` |
| Lint, must be clean | `cargo clippy --all-targets --no-deps -- -D warnings` |
| Node launcher tests | `cd npm/rust-doctor && bun test tests` |
| Packed launcher smoke | `cd npm/rust-doctor && bun run smoke:packed` |

Use `bun` under `npm/rust-doctor/`, never `npm` or `pnpm`. There is no CI: the
lint and test commands above are the only gate, so run them before calling a
change complete.

## Running the tool on this repository

The CLI opens a scope menu when stdin and stdout are both terminals. Pass
`--yes` for any scripted or agent-driven run:

```bash
cargo run --release -- . --yes --verbose
```

## Invariants the tests enforce

- **The crate passes its own rules.** Production code carries no `unwrap`,
  `expect`, `panic!`, or `dbg!`: use `?`, `ok_or(...)?`, `unwrap_or`, or
  `match`. `tests/score_credibility_packs.rs` scans this repository with the
  concurrency pack and fails on any hit.
- **No catalogued Clippy rule is `deny` by default**
  (`no_catalogued_clippy_rule_is_denied_by_default`). A `deny` rule cannot be
  switched off: dropping its `-W` restores Clippy's refusal and turns a scan
  into a compilation failure. `clippy::async_yields_async` and
  `clippy::unused_io_amount` were rejected for that reason.
- **The published catalog matches the shipped policy**
  (`the_published_catalog_matches_the_shipped_policy`). Editing
  `src/policy/catalog.rs` means `tests/corpus.json` has to be regenerated with
  it.
- **The score ranks by the rate the corpus adjudicated**
  (`the_noise_the_score_ranks_by_matches_the_adjudicated_rate`). `CORPUS_NOISE`
  in `src/policy/catalog.rs` mirrors the measured rates of `tests/corpus.json`,
  because the report ranks what to fix first by what repairing each rule is
  expected to be worth: its cost to the score discounted by its measured noise.
  Re-adjudicating a rule means moving both. The rate ranks, it never penalizes:
  what a rule costs the score is what it reported.
- **Two populations, two rates, no verdict crossing between them**
  (`each_population_publishes_its_own_rate_from_its_own_sites`). Every reviewed
  site carries a `population`: `healthy` says what a rule costs on code nobody
  wants disturbed, `agent` what it is worth on the code this tool exists for.
  Each rate is derived from its own sites against its own observations, and a
  Clippy rule can never carry an `agent` rate, since Clippy is switched off on
  untrusted code. `CORPUS_NOISE` mirrors the healthy rates today; switching that
  reference is a product decision, not a consequence of a number.
- **The JSON report is versioned.** Any change to the report shape bumps
  `SCHEMA_VERSION` in `src/report.rs`, currently 14, and the frozen v7 archive
  keeps projecting: `project_v11_wire_to_v7` in `tests/support/mod.rs` strips
  the members added since, which is what proves no historical field ever
  disappeared or changed type.
- **Dependencies are pinned exactly** (`= 1.8.5`, not `^1.8`) in `Cargo.toml`,
  and `Cargo.lock` is committed. The `missing_lockfile` detector requires it for
  a binary crate.
- **Structural rules default to warning, never error.** The
  `rust_doctor::structure::*` rules live in `src/structure/`, run on the same
  file set the source kernel enumerates, and report a clone family as one
  diagnostic whose `related` array names every member beyond the first. Their

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arthjean/rust-doctor](https://github.com/arthjean/rust-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
