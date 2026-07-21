---
trigger: always_on
description: - Repo: https://github.com/utooland/utoo
---

# Repository Guidelines

- Repo: https://github.com/utooland/utoo

## Project Overview

Utoo is a unified frontend toolchain with a Rust core and TypeScript bindings. It has two main products:
- **Package Manager (PM)** — the `utoo`/`ut` CLI, implemented in `crates/pm`
- **Bundler (Pack)** — `@utoo/pack`, built on Turbopack, with Rust core in `crates/pack-core` and Node.js bindings via NAPI-RS in `crates/pack-napi`

The `next.js/` directory is a **git submodule** (`utooland/next.js`) providing Turbopack crates. Many Rust crates depend on paths within it. Initialize with `git submodule update --init --recursive`.

## Project Structure & Module Organization

### Rust Crates (`crates/`)

| Crate | Purpose |
|-------|---------|
| `pm` | Package manager CLI (`utoo`/`ut`). Modules: `cmd/` (CLI commands), `service/` (business logic), `util/` (registry, cache, linker), `helper/` (workspace, deps) |
| `pack-core` | Bundler core built on Turbopack |
| `pack-api` | API layer for bundling operations (ProjectContainer, entrypoints) |
| `pack-napi` | NAPI-RS bindings exposing bundler to Node.js |
| `pack-schema` | Configuration schema definitions |
| `pack-tests` | Snapshot-based bundler integration tests |
| `pack-cli` | Rust CLI wrapper for the bundler |
| `ruborist` | Dependency resolution helper for PM |
| `utoo-wasm` | WebAssembly bindings for `@utoo/web` |

### Crate Layout & Publishing Policy

- **Exactly two crates are published to crates.io**: `utoo-pm` and `utoo-ruborist` (see `cargo-publish.yml`). Everything a published crate depends on must itself be on crates.io — never add an internal-only (`publish = false`) crate to their dependency trees. `pack-*` crates are internal-only (shipped to npm via `pack-napi`); `utoo-wasm` ships to npm as `@utoo/web`.
- **Modules over crates**: shared domains (spec parsing, npm-flavored semver, HTTP/net policy, package cache/tar store) live as modules inside `utoo-ruborist`, not as separate crates. Promote a module to a crate (`utoo-npm-args`, `utoo-semver`, `utoo-net`, `utoo-pm-store`) only when a consumer independent of pm/ruborist appears — promotion adds it to the crates.io publish chain (topological order, before its dependents).
- **WASM visibility rule**: `utoo-wasm` must never (transitively) compile pm-only code. Native-only functionality stays behind the `http-tarball` / `native-git` features, which `utoo-wasm` does not enable.
- **Façade rule**: downstream crates (`pm`, `utoo-wasm`) consume `utoo-ruborist` only via the curated re-export modules in its `lib.rs` (`graph`, `manifest`, `lock`, `registry`, `runtime`, `workspace`, `builder`, `semver`, `progress`, `compat`, `git`, `http`) plus the genuinely public modules `spec`, `service`, `util`. Never import through `utoo_ruborist::resolver::` / `::model::` / `::traits::` paths — add a re-export to the façade instead. Keep the façade minimal: remove re-exports that lose their last consumer.

### TypeScript Packages (`packages/`)

| Package | Purpose |
|---------|---------|
| `@utoo/pack` | Main bundler library wrapping pack-napi, includes Webpack compatibility layer |
| `@utoo/pack-cli` | CLI for bundler (`up`/`utoopack` commands) |
| `@utoo/pack-shared` | Shared types and utilities |
| `@utoo/web` | Browser-compatible toolchain via WASM |

### Key Data Flow

- **Pack**: User config → `@utoo/pack` (TS) → `pack-napi` (NAPI bridge) → `pack-api` → `pack-core` (Turbopack) → bundled output
- **PM**: CLI args → `crates/pm/src/cmd/` (command parsing) → `crates/pm/src/service/` (business logic) → `crates/pm/src/util/` (registry, cache, linker)
- **Web**: `@utoo/web` is an independent TS project that uses `utoo-wasm` for browser-compatible toolchain. Demo workflow:
  1. `ut build:local --workspace @utoo/web`
  2. `ut start --workspace utooweb-demo`

## Build, Test, and Development Commands

- Install deps: `npm install` (or bootstrap via global `utoo`/`ut` or locally built `target/release` binary)
- Build all packages: `npm run build` (via Turbo)
- Build specific package: `npx turbo run build --filter=@utoo/pack`
- Watch mode: `npm run dev`
- Build Rust crates: `cargo build`
- Faster local release build (no LTO): `cargo build --profile release-local`

### Testing

- PM unit tests: `cargo test -p utoo-pm`
- Bundler snapshot tests: `cargo test -p pack-tests`
- Update snapshots: `UPDATE=1 cargo test -p pack-tests`
- PM end-to-end tests: `./e2e/utoo-pm.sh`
- All JS tests: `npm run test` (via Turbo)

Bundler snapshot tests live in `crates/pack-tests/tests/snapshot/`. Each test case has an `input/` dir and an `output/` dir with expected results. Set `UPDATE=1` to regenerate snapshots.

### Formatting & Linting

- Format JS/JSON: `npm run biome` (Biome, double quotes, 2-space indent)
- Format Rust: `cargo fmt`
- Format TOML: `tombi format` (or `npx tombi format`)
- Spellcheck: `typos`

The pre-push hook runs: `cargo fmt --check`, `tombi format --check`, `npx biome ci`, `typos`.

## Coding Style & Conventions

- **Commits**: Conventional Commits with scopes — `feat(pm):`, `feat(pack):`, `fix(wasm):`, etc.
- **Rust edition**: 2024, nightly toolchain (version pinned in `rust-toolchain.toml`)
- **JS formatting**: Biome with double quotes, 2-space indent
- **Monorepo**: npm workspaces (JS) + Cargo workspace (Rust), orchestrated by Turborepo

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [utooland/utoo](https://github.com/utooland/utoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
