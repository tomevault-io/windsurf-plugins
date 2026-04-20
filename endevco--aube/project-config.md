---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Aube?

Aube is a fast Node.js package manager written in Rust. It mirrors pnpm's CLI surface and isolated symlink layout so users can swap it in, but it owns its own on-disk state: the global store lives in `~/.aube-store/`, the per-project virtual store is `node_modules/.aube/`, and the canonical lockfile is `aube-lock.yaml`. Aube reads and writes several lockfile formats — `aube-lock.yaml`, `pnpm-lock.yaml` (v9), `package-lock.json`, `npm-shrinkwrap.json`, `yarn.lock`, and `bun.lock` — and preserves whatever kind is already on disk, so a project with only `pnpm-lock.yaml` keeps getting `pnpm-lock.yaml` updated. `aube-lock.yaml` is the default only when no lockfile exists yet. If a project already has a `node_modules` from another package manager, aube leaves it alone and installs into its own tree alongside — it never reaches into `.pnpm/` or `~/.pnpm-store/`. Lifecycle scripts are skipped by default for security.

## Common Commands

```bash
cargo build                    # Build the `aube` binary
cargo test                     # Run unit tests
cargo clippy --all-targets -- -D warnings  # Lint (CI enforces zero warnings)
cargo fmt --check              # Check formatting

# BATS integration tests (requires Node.js 22 and GNU parallel).
# `cargo build` first, then run the mise task — it shards across cores
# via `bats --jobs`, so prefer it over raw `./test/bats/bin/bats`.
mise run test:bats                       # Run the full suite in parallel
mise run test:bats test/install.bats     # Run a single file (or several)
```

## Commit Messages (REQUIRED)

All commit messages and PR titles MUST follow the aube convention: short,
imperative subjects with a lowercase area prefix. Aube uses **area-based**
prefixes (subsystem / crate names), *not* strict conventional-commits
`feat:` / `fix:` / `chore:` verbs — a commit touching the resolver starts
with `resolver:`, not `fix(resolver):`. Reserve `feat:` / `fix:` for the
rare case where no subsystem prefix fits.

**Format:** `<area>(<subscope>): <description>` (subscope is optional)

**Areas:**
- Crates: `cli`, `resolver`, `registry`, `lockfile`, `store`, `linker`,
  `manifest`, `scripts`, `workspace`, `settings`
- Commands / features: `install`, `add`, `publish`, `dlx`, `audit`,
  `login`, `logout`, `deprecate`, `unpublish`, `fetch`, `import`,
  `catalog`
- Infrastructure: `ci`, `docs`, `bench`, `test`, `packaging`, `release`,
  `chore`
- Generic fallbacks (only when nothing else fits): `feat`, `fix`

**Subscopes:** narrow the area when it adds useful context, e.g.
`lockfile(yarn)`, `lockfile(npm)`, `lockfile(bun)`, `bench(hermetic)`,
`ci(release-plz)`, `ci(ppa-publish)`, `docs(home)`, `docs(installation)`,
`fix(dlx)`.

**Description style:**
- Lowercase after the colon
- Imperative mood ("add feature", not "added feature")
- Concise but descriptive — for most commits the subject line is the
  whole message

**Examples (from recent history):**
- `cli: add aubr/aubx multicall shims for run and dlx`
- `resolver: preserve npm-alias as folder name on fresh resolve`
- `store: swap CAS hash from SHA-512 to BLAKE3`
- `lockfile(yarn): add yarn berry (v2+) parse + write support`
- `bench(hermetic): warm registry with every PM, pin via .npmrc`
- `docs(home): plain-English speed copy + link stats to benchmarks`
- `ci(release-plz): grant contents:write to upload-assets caller`
- `chore: release v1.0.0-beta.4`

**Do not:**
- Prefix commits or PR titles with `[codex]`, agent names, or tool
  branding.
- Include PR numbers in local commit messages; GitHub adds those
  automatically when squash-merging.
- Use `feat(<area>):` / `fix(<area>):` when a subsystem prefix exists —
  prefer `<area>:` directly.

## User support channels

Aube does not use GitHub Issues — the repo has issues disabled. All bug
reports, questions, and feature requests go through
[GitHub Discussions](https://github.com/endevco/aube/discussions). When
writing docs, error messages, or any user-facing copy that points people
somewhere to get help, link Discussions and do **not** write phrases like
"file an issue" or "open an issue."

## Architecture

Cargo workspace with 9 crates under `crates/`. The binary entry point is `crates/aube/src/main.rs`.

**Install pipeline flow:** CLI (`aube`) -> resolve deps (`aube-resolver`) -> fetch from registry (`aube-registry`) -> import tarballs into global CAS (`aube-store`) -> link into `node_modules` (`aube-linker`)

Key crates:
- **aube** — Clap-based CLI, command implementations, auto-install state tracking (`state.rs`)
- **aube-resolver** — BFS dependency resolver with semver satisfaction and packument caching
- **aube-registry** — HTTP client for npm registry (abbreviated packument format, tarball downloads)
- **aube-lockfile** — Read/write support for `aube-lock.yaml`, `pnpm-lock.yaml` (v9), `package-lock.json`, `npm-shrinkwrap.json`, `yarn.lock`, and `bun.lock`. The install path preserves the existing lockfile kind via `detect_existing_lockfile_kind` (precedence: aube > pnpm > bun > yarn > npm-shrinkwrap > npm)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [endevco/aube](https://github.com/endevco/aube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
