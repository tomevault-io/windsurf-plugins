---
trigger: always_on
description: This file is the repository-wide operating contract for coding agents. It
---

# AGENTS.md - Pingclair

This file is the repository-wide operating contract for coding agents. It
applies regardless of which agent, editor, or development environment is
used. Keep it actionable: failure history, reproductions, and subsystem
archaeology belong in `docs/guardrails/`, not here. `CLAUDE.md` spells the
same rules out in more depth and adds the cross-crate picture; where the two
overlap, this file wins.

## 🧰 Tools first

Use the repository's intended tools before falling back to slower generic
workflows. If a common development tool required by this repository is
missing, install it rather than silently replacing the workflow with a worse
one.

### Canonical command interface

`just` owns repository workflows. CI runs the same recipes a developer runs
locally, so a green push is the same evidence as a green edit loop. Do not
duplicate long Cargo command lines in prose when a `just` recipe owns the
behavior.

The canonical gate is `just ci`, which runs formatting, Clippy, cargo-shear,
repository lint, documentation lint, the full nextest suite, and a benchmark
smoke run. Focused recipes:

```text
just fmt              # format Rust sources in place
just fmt-check        # fail when formatting differs
just clippy           # workspace clippy, -D warnings
just test -p <crate>  # nextest for one crate (accepts nextest args)
just lint             # fmt-check + clippy + shear + repo-lint + docs-lint
just check            # lint + test
just ci               # check + bench-smoke; the exact CI gate
just shear            # unused-dependency check
just repo-lint        # mechanical repository invariants
just docs-lint        # codespell + markdownlint
just bench / bench-smoke
just h3               # the three HTTP/3 verification scripts
just disk / cache-report
just install          # install the pinned local tooling
```

### General CLI

- Prefer `rg` over `grep -R`, `fd` over `find`, `bat` over `cat`, `jq` for
  JSON, and `gsed` when GNU `sed` behavior is required.
- Respect `.gitignore`. Do not search generated directories such as
  `target/`, persistent build caches, benchmark result caches, or vendored
  trees unless the task specifically concerns them.
- Avoid relying on macOS BSD command behavior in scripts intended for Linux
  CI, and remember that zsh does not word-split unquoted variables.
- Be patient with long Rust builds; never kill cargo or rustc by PID. The
  Cargo lock is expected to make builds slow.

### Development environment

Installed and preferred tools include `rg`, `fd`, `bat`, `jq`, `gsed`,
`cargo-nextest`, `cargo-watch`, and `just`. When a task is inefficient with
the available tools, prefer a specialized tool and install it when
appropriate: Homebrew for macOS system tools, `cargo install` for Rust
CLIs, npm for Node CLIs, official installers when required. `cargo-watch`
is for continuous checking during development.

## 🦀 Rust toolchain is exact

CI validation uses Rust **1.97.1**; the workspace declares
`rust-version = "1.97"`. Use `cargo +1.97.1` for formatting checks, Clippy,
formal builds, CI-parity tests, and release validation — `+1.97.1` is not
decoration. Different compilers produce different inference, warnings, and
rustfmt line breaking; all-green locally followed by all-red in CI has
happened in both directions (newer-than-CI on 2026-07-29, an older toolchain
in the release image on 2026-08-02).

## 🧪 Local tests use nextest

`cargo-nextest` is the default local and CI test runner. Use `just test`
instead of `cargo test` while iterating; `cargo test` is reserved for
doctests, Cargo-harness-specific behavior, and reproducing a failure known
only in that harness.

When only one crate or test binary changed, run it first:

```bash
just test -p pingclair-proxy
cargo +1.97.1 nextest run -p pingclair --test integration --no-fail-fast
```

Run the full suite before handoff when shared crates, configuration, or
policy code changed. When Rust documentation examples change, run the
relevant doctests explicitly (`cargo +1.97.1 test --doc`), because nextest
does not run them.

## 💾 Persistent build caches

Expensive Rust builds must reuse persistent build state. Keep primary Cargo
target directories out of disposable containers and short-lived build
directories. Use stable architecture-specific caches:

```text
~/.cache/pingclair-build/
├── macos-aarch64/
├── linux-aarch64/
└── linux-x86_64/
```

Use `sccache` when available for repeated compatible builds, and do not copy
target trees between incompatible architectures. Before concluding that a
full rebuild is required, check whether the target dir, target triple,
toolchain, feature set, profile, `RUSTFLAGS`, linker, build environment, or
cache path changed — a path change alone can turn a warm build cold.

### Build-cache disk budget

Observe cache size with `just disk`. The normal budget is about **80 GiB**;
crossing **100 GiB** is a stop condition unless the artifacts are deliberately
required for an active benchmark, comparison, or release. Do not reflexively
run `cargo clean`: measure what is large, identify the architecture and
profile, preserve caches required by current work, and remove stale trees
selectively. Do not delete Cargo registry or git caches merely because a
target tree is large.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dorianverlaine/pingclair](https://github.com/dorianverlaine/pingclair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
