---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`jrs` is a Java build system written in Rust: it builds, tests, runs and packages a
single-module Java project from one `jrs.toml` manifest, resolving dependencies from
Maven Central. It shells out to `javac`, `java` and the other JDK tools — it is a
driver, not a reimplementation of the JDK. Kotlin, Scala and Groovy compile alongside
Java (`specs/JVM_LANGUAGES.md`, SPEC §7.7): their compilers are resolved from Maven
Central as isolated tool graphs, pinned in `jrs.lock`'s `[[tool]]` blocks, and
run on the project's JDK — still a driver.

`specs/INITIAL_SPEC.md` is the design document the implementation follows, and module
doc comments cite it by section (`SPEC §8.2`). Read the relevant section before changing
behaviour; **§12.1 lists the four places where the code deliberately diverges from the
spec** — those divergences are intentional, don't "fix" them back.

## Commands

```
cargo build
cargo test                      # hermetic: no network, uses a file:// repo fixture
cargo fmt --check
cargo clippy --all-targets -- -D warnings
```

CI (`.github/workflows/rust.yml`) runs exactly those four on Linux, macOS and
Windows (`fmt` on Linux only), plus the network tests on Linux, on push and PR
against `master`. Changes that only touch `*.md`, `LICENSE` or `website/` are
excluded via `paths-ignore` — they cannot break the build, so they do not run it. A `v*` tag
push runs the same checks, then `bump` writes the tag's version into `Cargo.toml`
and `Cargo.lock` and commits it to `master` (the tag itself is never moved), the
`dist` matrix builds release binaries for Linux (musl), macOS and Windows from that
commit, and `release` publishes them. The tag must point at the tip of `master`. Don't
bump the version by hand — tagging is the release process.

`.github/workflows/website.yml` builds `website/` with Bun and rsyncs it to
`getjrs.dev` on the mikr.us VPS (the `VPS_*` repository secrets) on pushes to
`master` that touch `website/` or `logo.png`. `rust.yml` also calls it after
`release`, with the version commit and `secrets: inherit`, since that commit is
pushed with the workflow token and triggers nothing itself — so the site's
version stays current. The site also serves `website/install.sh` as
`getjrs.dev/install.sh`, the `curl | sh` installer for the release binaries; it
fetches `releases/latest/download/jrs-<target>.tar.gz` and `SHA256SUMS`, so the
asset names in `dist` must stay unversioned. `website/docs/index.html` is the
user documentation at `getjrs.dev/docs/`, written from `DOCS.md` and the
specs: a change to a command, a flag or a manifest key belongs there too.
`README.md` is a short overview that links into `DOCS.md`, the full reference;
keep the details in `DOCS.md`.

Single tests and single suites:

```
cargo test --test build                             # one integration suite
cargo test --test output the_ascii_fallback         # one integration test
cargo test resolve::                                # unit tests in one module
cargo test --features network-tests --test network  # the Maven Central tests
cargo bench --bench resolution                      # SPEC §12 M5: network vs jrs vs renderer
cargo bench --bench incremental                     # SPEC §7.2: rebuild time after one edit, javac's share
```

Driving jrs against a Java project:

```
cargo run -- --manifest-path /path/to/project build
cargo run -- run -- arg1 arg2
```

A JDK 17+ must be on `PATH` or at `JAVA_HOME`. Tests that need one skip loudly via the
`require_jdk!` macro rather than failing, so a green `cargo test` on a machine without
`javac` does not mean the build pipeline was exercised — check the `SKIPPED` lines.

Set `JRS_CACHE_DIR` to relocate the shared artifact cache (`~/Library/Caches/jrs` on
macOS) when experimenting.

## Git

- **Never push.** Pushing is manual and stays the maintainer's decision — do not run
  `git push`, and do not open or merge pull requests. Committing locally when asked is
  fine; getting the commit onto a remote is not.
- **No AI attribution in commit messages.** No `Co-Authored-By: Claude`, no
  "Generated with Claude Code" trailer, no tool mention in the subject or body. Write
  the message as the change itself warrants.

## Architecture

**`ARCH.md` is the full architecture map** — module layers, the `Session` spine,
resolution, compile units, the output layer, on-disk layout, with ASCII diagrams.
Read it before a change that crosses module boundaries, and keep it in step when
one moves a boundary, a phase or a file under `target/`. What follows is the
summary.

Library-first: everything lives in `src/lib.rs` modules; `main.rs` is five lines of
`std::process::exit(jrs::cli::main())`. Every phase can be driven from a test without
spawning the CLI, and the integration tests do exactly that.

```
jrs.toml ──parse──► Manifest ──► Project (layout, source globbing)
                        │
                        └──► resolve ──► jrs.lock ──► Classpath
                                                │
Project + Classpath ──► compile ──► target/classes ──► package | runner | test
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pwittchen/jrs](https://github.com/pwittchen/jrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
