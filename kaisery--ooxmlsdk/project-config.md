---
trigger: always_on
description: This file is the entrypoint for agents working in this repository. Keep it
---

# Repository Guidelines

This file is the entrypoint for agents working in this repository. Keep it
short, stable, and map-like. Treat deeper docs as the source of truth.

Read first:

1. `README.md` for public API and feature surface
2. `ARCHITECTURE.md` for workspace layout, data flow, test buckets, and upstreams
3. `docs/tests/` for fixture taxonomy and coverage matrices when touching tests
4. `docs/specs/` for user-facing format notes when touching behavior or docs

## Project Stage

The current phase of this repository is focused on rewriting upstream
`Open-XML-SDK` and LibreOffice logic in Rust while reusing as much upstream
test coverage and fixture evidence as possible.

Priority by subsystem:

- `ooxmlsdk`: use `Open-XML-SDK` as the primary reference and LibreOffice as a secondary reference
- `ooxmlsdk-pdf`: use LibreOffice as the primary reference

Until the implementation reaches a higher maturity level:

- prefer translating upstream behavior into idiomatic Rust over inventing new behavior
- prefer upstream tests, fixtures, and source code over local guesswork
- for `ooxmlsdk`, check `Open-XML-SDK` first and use LibreOffice mainly for supplemental evidence
- for `ooxmlsdk-pdf`, check LibreOffice first
- when behavior is unclear, inspect upstream first instead of inferring a new rule
- do not broaden the feature surface or add novel logic unless the task explicitly requires it
- treat unexplained behavioral differences from upstream as bugs or gaps to investigate, not as opportunities to design a new model

## Working Style

- Start from local evidence. Use `rg` / `rg --files` first.
- Read only the files needed for the task.
- Keep summaries diff-based rather than conversation-based.
- Do not paste broad search output or large generated snippets unless asked.
- Run commands from the repository root.
- Do not fix clippy or compiler warnings by adding `#[allow(...)]` or `#![allow(...)]`; remove dead code, tighten cfgs, or improve the implementation instead.
- Cargo generation, format, test, clippy, and bench commands must run sequentially in the default `target/` directory; do not set `CARGO_TARGET_DIR`.
- This repository has long Cargo build/test times. After starting any Cargo command, let it run to completion and wait for the final result.
- Never start a second Cargo command while another Cargo command is still running, even for a quick verification, retry, status check, or no-op probe.
- While a Cargo command is running, do not launch other repository commands just to inspect progress. Do not poll with extra Cargo invocations. If output is quiet for a while, keep waiting.
- After starting a Cargo command, wait for the background command to return before doing any other repository work. Do not interrupt yourself to inspect files, diffs, or partial results while that Cargo command is still running; only look at the result after the command exits.
- For long-running Cargo commands, use the longest practical command wait interval available. If the command returns a background session, poll with long intervals, preferably several minutes, so the terminal does not emit frequent intermediate wait messages.
- Do not send progress commentary while waiting for Cargo. Report only after the Cargo command exits, unless the user explicitly asks for a status update.
- If Cargo reports a target lock or another Cargo process is already running, do not probe processes or start competing commands; just wait for the active Cargo command to finish and then continue.
- If Cargo reports a target lock, wait for Cargo rather than probing processes.
- Never create ad hoc Cargo projects, temporary manifests, or throwaway crates
  for inspection, reproduction, debugging, or code execution. This ban applies
  everywhere: `/tmp`, other workspace directories, sibling checkouts, nested
  repository subdirectories, and any other path.
- Do not work around the ban by generating temporary Cargo files, helper
  projects, or one-off crates outside the repository. Analyze with existing
  repository commands, checked-in fixtures, and existing tests only.
- If inspecting runtime output requires instrumentation, prefer an existing
  test or existing repository binary. Ask the user before adding temporary
  debugging code.

## Commands

- `cargo test -p ooxmlsdk-build test_gen -- --ignored --nocapture`: regenerate `sdk_data/` and runtime generated code from checked-in `data/` and package schemas.
- `cargo test -p ooxmlsdk-test`: fast integration lane for common runtime and package behavior.
- `cargo test --workspace`: default full test lane.
- `cargo test --workspace --no-default-features`: no-default-features lane.
- `cargo test --workspace --no-default-features --features parts`: parts lane without validators or MCE-specific behavior.
- `cargo test --workspace --no-default-features --features flat-opc`: Flat OPC lane without validators or MCE-specific behavior.
- `cargo test --workspace --no-default-features --features mce`: MCE lane without validators or Flat OPC-specific behavior.
- `cargo test -p ooxmlsdk-test --features validators`: validator-focused lane.
- `cargo fmt --all`: format.
- `cargo clippy --workspace --all-targets -- -D warnings`: default clippy lane.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaiserY/ooxmlsdk](https://github.com/KaiserY/ooxmlsdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
