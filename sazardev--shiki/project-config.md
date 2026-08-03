---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**shiki** (私記) is a TUI note-taking app written in Rust, inspired by Yazi's three-pane
layout and modal navigation. Notes are plain Markdown files with YAML frontmatter,
organized into "notebooks" (directories, each its own git repo). Full design spec,
motivation, keybindings, config format, and included themes are documented in `IDEA.md` —
read it before making architectural changes, since it's the source of truth for intended
behavior (layout, CLI commands, config schema, etc).

## Commands

```sh
cargo build --workspace              # build everything
cargo check --workspace              # fast type-check (use this while iterating)
cargo clippy --workspace --all-targets   # lint; keep this clean before considering work done
cargo fmt --all                      # format (run after editing, before checking clippy)
cargo run -p shiki-cli -- <args>     # run the binary, e.g. `-- new "titulo"`, `-- daily`, no args launches the TUI
```

Almost no automated tests yet — `panel_drawer::tests` (`shiki-tui/src/panel_drawer.rs`) are the
first, covering `drawer_hit_at`'s mouse coordinate math (a plain function of numbers, not `&App`,
specifically so it's unit-testable without constructing a full app). When adding tests for
`shiki-core`/`shiki-config` logic, put them as `#[cfg(test)]` modules in the relevant file — those
two crates have no TUI/terminal dependency, so they're the easiest to unit test. For `shiki-tui`,
prefer designing the function to not need `&App` in the first place (as `drawer_hit_at` does) over
constructing a full `App` in a test.

To exercise the CLI without touching the real user config/data, override XDG dirs (used via
`directories::ProjectDirs::from("", "", "shiki")` in `shiki-config`):

```sh
XDG_CONFIG_HOME=/tmp/shiki-test-config XDG_DATA_HOME=/tmp/shiki-test-data \
  cargo run -p shiki-cli -- notebook create personal
```

## Versioning

`[workspace.package] version` in the root `Cargo.toml` is the single version number for the whole
app — every crate inherits it via `version.workspace = true`, so there's exactly one place to bump.
The TUI status bar shows it (right-aligned in the footer, paired with the `? help` hint) via
`env!("CARGO_PKG_VERSION")` in `shiki-tui/src/status_bar.rs`, which reads shiki-tui's own
(inherited) manifest version at compile time. Cutting a release is two steps: bump the workspace
version, add a `CHANGELOG.md` entry (follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)).

**Tagging the release is automatic, not a manual `git tag && git push` step anymore.**
`.github/workflows/auto-tag.yml` watches every push to `main`; if the triggering commit's message
contains `[PUBLISH]`, it reads the version straight out of the root `Cargo.toml`, checks
`git ls-remote --tags origin` to confirm that tag doesn't already exist, and pushes it itself. This
exists because v0.7.0 was tagged locally but the `git push origin v0.7.0` step was never actually
run — the tag sat on nobody's remote, `release.yml` (which only triggers on a pushed `v*` tag)
never ran, and crates.io silently stayed on 0.5.0 for three version bumps before anyone noticed.
Pushing the tag with the default `GITHUB_TOKEN` would silently not work either: GitHub suppresses
workflow triggers caused by a token a workflow already used, specifically to prevent infinite
trigger loops, so a tag pushed that way would never fire `release.yml`. `auto-tag.yml` instead
pushes using a `RELEASE_TAG_PAT` secret (a personal access token with repo write access) so the tag
push looks like it came from a real user. **That secret doesn't exist yet** — same
not-yet-provisioned-secret situation as `AUR_SSH_PRIVATE_KEY` below; until Omar adds it,
`auto-tag.yml` still runs and fails loudly with a clear `::error::` annotation rather than silently
no-op'ing, so a missing secret can't reproduce the exact "nobody noticed" failure mode it exists to
prevent. To cut a release: bump the version, add the changelog entry, and include `[PUBLISH]`
anywhere in the commit message that lands on `main` — everything downstream (tag → build → GitHub
Release → crates.io → packaging manifests) follows automatically.

**The status bar paints no background** (`status_bar.rs` — no `.bg(...)` anywhere, spans just use
themed fg colors on the terminal's own background) and only shows what's contextually useful: the
mode label is omitted entirely in `Mode::Normal` (only INSERT/EDIT/VISUAL are worth flagging),
there's no theme-name display, and the metadata block is focus-dependent — character count of the
selected note while reading/editing one (`Focus::Notes`/`Focus::Preview` with a note selected), or
the note count in view otherwise (e.g. still browsing NOTEBOOKS). Git info comes from
`App::git_status: shiki_core::git::GitStatus` — `git::status(path, remote)` resolves `HEAD`'s
branch name and runs `repo.graph_ahead_behind` against `refs/remotes/{remote}/{branch}` (from the
last fetch — it doesn't talk to the network itself). All three counts render together when
nonzero rather than only ever showing one: `+{dirty_count}` (uncommitted files), `{UPLOAD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sazardev/shiki](https://github.com/sazardev/shiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
