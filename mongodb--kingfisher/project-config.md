---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Project Overview

Kingfisher is an open-source secret scanner and live secret validator written in Rust by MongoDB. It detects, validates, and helps remediate leaked API keys, tokens, and credentials across code repositories, git history, and integrated platforms.

Key capabilities:
- Secret detection with 942 built-in rules (820 standalone detectors + 122 dependent rules; 484 standalone detectors include live validation as of 2026-04-24)
- Live credential validation against provider APIs
- Direct secret revocation from CLI
- Blast radius mapping (AWS, GCP, Azure, GitHub, GitLab, Slack)
- Output formats: TOON, JSON, SARIF, interactive HTML
- Platform integrations: GitHub, GitLab, Azure Repos, Bitbucket, Gitea, Hugging Face, S3, GCS, Docker, Jira, Confluence, Slack, Microsoft Teams, Postman

## Scope
- Applies to the entire repository rooted at this file.
- If a deeper `AGENTS.md` exists in a subdirectory, that file takes precedence for its subtree.

## Repository Structure
- `src/`: main binary source
- `src/cli/commands/`: CLI command implementations
- `src/matcher/`: pattern matching engine
- `src/scanner/`: core scanning logic
- `src/parser/`: language-aware context verification (lightweight lexers, `tl` for HTML, `cssparser` for CSS)
- `src/reporter/`: TOON/JSON/SARIF/HTML report generation
- `src/access_map/`: access mapping analysis
- `crates/kingfisher-core/`: shared types and core logic
- `crates/kingfisher-rules/`: rule loading and rule data
- `crates/kingfisher-rules/data/rules/`: YAML detection rules
- `crates/kingfisher-scanner/`: embeddable high-level scanning API
- `crates/kingfisher-scanner/src/validation/`: shared typed and raw credential validators
- `tests/`: integration/e2e tests
- `testdata/`: test fixtures
- `docs/`: user and developer docs
- `docs/viewer/`: static hosted/local report viewer assets
- `docs-site/`: MkDocs documentation sources, overrides, and generated site output
- `vendor/vectorscan-rs/`: vendored vectorscan bindings

## Toolchain and Environment
- Shell assumptions in build scripts: `bash` with `set -eu -o pipefail`
- Workspace minimum Rust version: `1.94` (`Cargo.toml`)
- `make check-rust` enforces `>= 1.94.1` for build targets
- Windows Makefile targets (`windows-x64`, `windows-arm64`) expect an MSYS2 environment with `pacman` available.
- Prefer `rg` and `rg --files` for fast code/file search.

## Quick Commands
- Development build: `cargo build`
- Release build: `cargo build --release`
- Tests (preferred wrapper): `make tests`
- Tests (direct): `cargo test --workspace --all-targets`
- Nextest (if installed): `cargo nextest run --workspace --all-targets`
- Format: `cargo fmt --all`
- Lint: `cargo clippy --workspace --all-targets -- -D warnings`
- Clean: `make clean`

## Build Targets (Makefile)
- Host convenience:
  - `make linux`
  - `make darwin`
  - `make windows` (Windows host; builds `windows-x64` and `windows-arm64`)
- Explicit platform archives:
  - `make linux-x64`
  - `make linux-arm64`
  - `make darwin-x64`
  - `make darwin-arm64`
  - `make windows-x64` (Windows host only; MSYS2/MinGW flow)
  - `make windows-arm64` (Windows host only; MSYS2 clangarm64 flow)
- Ubuntu bare-metal (Zig/cargo-zigbuild flow):
  - `make ubuntu-x64`
  - `make ubuntu-arm64`

## Code Style
- Rust formatting is defined in `rustfmt.toml` (`max_width = 100`, 4-space indentation, Unix newlines, reordered imports).
- Keep edits minimal and scoped; preserve existing conventions in touched files.
- Prefer clear, maintainable fixes over broad refactors unless requested.

## Architecture Notes
- Rules are YAML-driven and loaded from `crates/kingfisher-rules/data/rules/`.
- Allocator feature flags are in root `Cargo.toml`:
  - `use-mimalloc` (default)
  - `use-jemalloc`
  - `system-alloc`
- Validation modules live in `crates/kingfisher-scanner/src/validation/`; optional validation feature sets are defined in `crates/kingfisher-scanner/Cargo.toml` (e.g., `validation-raw`, `validation-aws`, `validation-gcp`, `validation-database`, `validation-all`).

## Validation and Revocation Policy
- Default rule: define validation logic in rule YAML (`validation:` block), especially `Http` or `Grpc`, not Rust code.
- Typed validators are first-class schema variants (`AWS`, `AzureStorage`, `Coinbase`, `GCP`, `MongoDB`, `MySQL`, `Postgres`, `Jdbc`, `JWT`) for stable, reusable validation families.
- Raw validators use `validation: { type: Raw, content: <name> }` and are the ad-hoc exception path for provider-specific or protocol-specific validation that cannot be expressed reliably in YAML alone. Implement them in `crates/kingfisher-scanner/src/validation/raw.rs`.
- Treat Rust validation additions as rare; prefer extending YAML-based validation first.
- If a Rust exception path is required, prefer adding a raw validator before introducing a new typed validator. Add a new typed validator only when it represents a reusable schema-level validation family.
- Do not convert existing typed validators to `Raw` just for consistency.
- For rules that include validation, add a `revocation:` section whenever the third-party API safely supports revocation.

## Common Development Tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb/kingfisher](https://github.com/mongodb/kingfisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
