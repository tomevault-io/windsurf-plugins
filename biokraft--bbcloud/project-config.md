---
trigger: always_on
description: Guidance for AI coding agents working in this repository. `CLAUDE.md` is a symlink to this file,
---

# AGENTS.md

Guidance for AI coding agents working in this repository. `CLAUDE.md` is a symlink to this file,
because Claude Code reads that name.

## What this is

`bb` is a Bitbucket Cloud REST API CLI, written in Rust. It is an independent rewrite of an
unmaintained PHP client of the same name; see `NOTICE` for attribution. The rewrite is why several
hard rules below exist — the shell-invocation and cleartext-credential rules in particular encode
mistakes this implementation must not repeat.

**Auth context you must not get wrong:** Atlassian removed Bitbucket Cloud app passwords on
2026-07-28. Authentication is HTTP Basic with the **Atlassian account email** as the username
and an **API token** as the password. Never add, document, or suggest app passwords — a test
(`tests/cli.rs::help_does_not_mention_app_passwords`) fails the build if help text mentions them.

## Build and check

Cargo is installed via rustup and may not be on `PATH`. If `cargo` is not found:

```bash
export PATH="$HOME/.cargo/bin:$PATH"
```

All three of these must exit 0 before any change is done:

```bash
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test --all
```

Toolchain is pinned to 1.97 in `rust-toolchain.toml`; MSRV is 1.88. Do not lower the pin —
current `keyring`, `reqwest`, and `wiremock` all require newer than 1.75.

## Development workflow

**Branches.** Start every change on a branch off `main`, named for what it is: `feat/…`,
`fix/…`, `docs/…`, `ci/…`. Never commit to `main` directly — it is protected and requires a pull
request with passing checks.

**Before opening a pull request**, run these locally and get all three green:

```bash
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo test
```

One caveat, because it bites: CI additionally sets `RUSTFLAGS: -D warnings` for the whole
workflow, so a warning clippy tolerates locally can still turn CI red. If CI fails on a warning
you did not see locally, that is why.

**The pull request.** CI runs five jobs, and all must pass before merge: `test (macos-latest)`
and `test (ubuntu-latest)` cover the two supported platforms; `msrv 1.88` guards the version
floor the README promises, and needs `RUSTUP_TOOLCHAIN` set because otherwise `rust-toolchain.toml`'s
1.97 pin would override the action's toolchain choice; `coverage` uploads to Codecov and now
fails the build if the upload itself fails, because a silently skipped upload once left the
badge reading "unknown" for weeks; `publish dry run` catches crates.io packaging problems before
a real release depends on them.

**Commit messages are functional, not cosmetic.** release-plz parses them to build the changelog
and pick the next version: `feat:` earns a minor bump, `fix:` a patch, a `!` suffix or a
`BREAKING CHANGE:` trailer a major, and `test:`/`chore:`/`ci:` are skipped from the changelog
entirely. Get the prefix wrong and you get a wrong version number, so it matters more here than
in a repo where the changelog is written by hand.

**Releases are automated end to end. Never do any of these by hand:** edit `version` in
`Cargo.toml`, write `CHANGELOG.md`, or create a git tag. Merging to `main` makes release-plz open
a release PR carrying the version bump and generated changelog; merging *that* PR tags the
commit, publishes to crates.io, builds the four target binaries with their checksums, and updates
the Homebrew formula in `biokraft/homebrew-tap`.

Two prohibitions specific to this repository's history:

- **Never force-push `main`.** It was force-pushed once, while the repository was still private
  and being prepared; that period is over. Published tags point into the current history, and
  other people's clones descend from it.
- **Never `git push --tags`.** Tags come only from release-plz.

The `docs/` tree — design specs and execution records — lives on a separate remote outside this
repository, which is why you will not find it here.

## Hard rules

These are enforced by lints or tests. Breaking one breaks the build.

- **No `.unwrap()` / `.expect()` / `panic!` in `src/**` outside `#[cfg(test)]`.** `Cargo.toml`
  sets `clippy::unwrap_used = "deny"` and `expect_used = "deny"` package-wide. Test code may
  unwrap freely: `#[cfg(test)] mod tests` blocks inside `src/` carry a narrowly scoped
  `#[allow(clippy::unwrap_used)]`; files under `tests/` carry it crate-level.
- **No `#[allow(dead_code)]` anywhere in `src/`.** There is a library target, so `pub` items are
  reachable by definition and never warn. If something you add warns as dead, that is a signal it
  should not exist yet.
- **`#![forbid(unsafe_code)]`** in both `src/lib.rs` and `src/main.rs`.
- **Never invoke a shell.** All subprocess work goes through `std::process::Command` with an
  explicit argument vector (see `src/git.rs`), or through `open::that_detached` for URLs. No
  `sh -c`, no `format!` assembling a command line. The PHP version's `exec()` on a git-remote-derived
  string was a remote code execution bug; do not reintroduce the shape.
- **`reqwest`'s TLS is rustls only.** It is declared `default-features = false, features = ["json", "rustls-tls"]`.
  No OpenSSL in the HTTP path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biokraft/bbcloud](https://github.com/biokraft/bbcloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
