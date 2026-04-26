---
trigger: always_on
description: This file applies to the entire FlowScope monorepo.
---

# AGENTS.md

## Scope

This file applies to the entire FlowScope monorepo.

## Repo Overview

FlowScope is a Rust + TypeScript monorepo.
Key areas:
- `crates/` Rust workspace (core engine, wasm, CLI, export).
- `packages/` TypeScript packages (`@pondpilot/flowscope-core`, `@pondpilot/flowscope-react`).
- `app/` demo web app (Vite + React).
- `vscode/` VS Code extension + `vscode/webview-ui`.

## Cursor/Copilot Rules

- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` found.

## Tooling Defaults

- Use `just` as the task runner (see `justfile`).
- Use `yarn` for Node workspaces.
- Use `cargo` for Rust workspace.
- Node.js 18+ and Rust 1.82+ expected (see `README.md`).

## Build Commands

- `just build` (WASM + TypeScript packages).
- `just build-rust` (Rust workspace debug build).
- `just build-rust-release` (Rust workspace release build).
- `just build-cli` (CLI release build).
- `just build-cli-serve` (CLI with embedded web UI, release build).
- `just build-cli-serve-debug` (CLI with embedded web UI, debug build).
- `just build-wasm` (runs `./scripts/build-rust.sh`).
- `just build-ts` (runs `yarn build:ts`).
- `just run` (build + dev server).

### CLI Serve Mode Build Order

The CLI serve feature embeds the web app using rust-embed at compile time. This requires the app to be built first:

1. `cd app && yarn build` - Build frontend assets to `app/dist/`
2. `cargo build -p flowscope-cli --features serve` - Compile CLI with embedded assets

The `just build-cli-serve` target handles this dependency automatically.

## Dev Commands

- `just dev` (Vite dev server at `http://localhost:5173`).
- `yarn dev` (from `app/` if you want direct Vite usage).
- `just cli -- <args>` (run CLI in debug mode).
- `just cli-release -- <args>` (run CLI in release mode).

## Lint, Format, Typecheck

- `just lint` (Rust + TypeScript lint).
- `just lint-rust` (`cargo clippy --workspace -- -D warnings`).
- `just lint-ts` (`yarn workspaces run lint`).
- `just lint-fix` (`yarn workspaces run lint:fix`).
- `just fmt` (Rust + TS formatting).
- `just fmt-rust` (`cargo fmt --all`).
- `just fmt-check-rust` (`cargo fmt --all -- --check`).
- `just fmt-ts` (runs Prettier across workspaces).
- `just typecheck` (`yarn workspaces run typecheck`).

## Test Commands

- `just test` (Rust + TS tests).
- `just test-rust` (`cargo test --workspace`).
- `just test-rust-release` (`cargo test --workspace --release`).
- `just test-ts` (`yarn workspaces run test`).
- `just test-core` (`cargo test -p flowscope-core`).
- `just test-cli` (`cargo test -p flowscope-cli`).
- `just test-cli-serve` (CLI tests with serve feature, builds app first).
- `just test-lineage` (`cargo test -p flowscope-core --test lineage_engine`).
- `just test-lineage-verbose` (same test with `--nocapture`).
- `just check-schema` (Rust schema guard + TS schema compatibility).
- `just coverage` (generate HTML coverage report in `coverage/`, requires `cargo-llvm-cov`).
- `just coverage-lcov` (generate LCOV file at `lcov.info` for CI/Codecov).
- `just coverage-summary` (print coverage summary to stdout).

## Workspace Utilities

- `just install` (install Node dependencies).
- `just setup` (install deps + tools + hooks + build).
- `just install-rust-tools` (installs `wasm-pack` and `cargo-watch`).
- `just install-hooks` (install `prek` hooks).
- `just clean` (cargo clean + remove node_modules).
- `just update-schema` (regenerate API schema snapshot).
- `just check` (fmt check + lint + typecheck + schema checks).
- `just check-all` (Rust + TS + schema compatibility).
- `just watch` (rebuild Rust workspace on changes).
- `just watch-test` (run Rust tests on changes).
- `just watch-lineage` (run lineage tests on changes).

## Package-Level Scripts

- `packages/core`: `yarn test`, `yarn test:watch`, `yarn lint`, `yarn typecheck`.
- `packages/react`: `yarn test`, `yarn test:watch`, `yarn lint`, `yarn typecheck`.
- `app`: `yarn dev`, `yarn build`, `yarn preview`, `yarn lint`, `yarn typecheck`.
- `vscode`: `npm run build`, `npm run build:webview`, `npm run watch`, `npm run typecheck`.
- `vscode/webview-ui`: `yarn build`, `yarn dev`, `yarn lint`, `yarn typecheck`.

## Single Test Tips

- Rust lineage tests: `just test-lineage-filter PATTERN`.
- Rust lineage tests (manual): `cargo test -p flowscope-core --test lineage_engine PATTERN`.
- Schema compatibility: `yarn workspace @pondpilot/flowscope-core test schema-compat.test.ts --silent`.
- For other Rust crates, use `cargo test -p <crate>` then filter with a test name if needed.

## Code Style (Rust)

- Follow standard Rust conventions (see `CONTRIBUTING.md`).
- Format with `cargo fmt` before committing.
- Lint with `cargo clippy` and fix warnings.
- Workspace uses Rust 2021 edition (see `Cargo.toml`).
- Avoid unused variables; if intentionally unused, prefix with `_`.
- Keep modules small and focused; `flowscope-core` uses a layered analyzer.

## Error Handling (Rust)

- Use `ParseError` for fatal parsing failures returned via `Result<T, ParseError>`.
- Use `Issue` for non-fatal analysis problems (collected and returned alongside results).
- `flowscope-cli` uses `anyhow::Result` with `Context` for CLI errors.
- `flowscope-export` and analyzer input use `thiserror::Error` for structured errors.

## Code Style (TypeScript)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pondpilot/flowscope](https://github.com/pondpilot/flowscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
