---
trigger: always_on
description: This file provides instructions for contributing to the `rust-project-goals` repository.  It is intended to be self-contained: everything needed to work effectively in this repository is documented here or cited with a precise path.
---

# Instructions for rust-project-goals work

This file provides instructions for contributing to the `rust-project-goals` repository.  It is intended to be self-contained: everything needed to work effectively in this repository is documented here or cited with a precise path.

## Repository overview

This repository hosts the Rust project's **goal proposals** -- a bottom-up process where contributors propose goals, Rust teams review them, and accepted goals are tracked to completion.  The repository is published as an mdBook site at <https://rust-lang.github.io/rust-project-goals/>.

There are two kinds of content:

- **Goal documents** (`src/<milestone>/<name>.md`) -- individual proposals with metadata, motivation, work items, and team asks.

- **Roadmap documents** (`src/<milestone>/roadmap-<theme>.md`) -- narrative pages that group related goals under a unifying theme.

Milestone directories follow the pattern `YYYY` or `YYYYhN` (e.g., `2024h2`, `2025h1`, `2026`).  Starting with 2026, goals are annual rather than semiannual.

## Repository structure

```
src/                    mdBook source
  SUMMARY.md            Book table of contents (manually maintained)
  TEMPLATE.md           Template for new goal proposals
  ROADMAP_TEMPLATE.md   Template for new roadmap documents
  2026/                 Current goal period
    README.md           Overview page (uses preprocessor directives)
    *.md                Goal and roadmap documents
  2025h2/, 2025h1/, ... Prior periods
  about/, how_to/       Explanatory pages
  admin/                Admin/operational docs
crates/
  rust-project-goals/       Core library (parsing, formatting, GitHub API)
  rust-project-goals-cli/   CLI tool (`cargo rpg`)
  mdbook-goals/             mdBook preprocessor
  rust-project-goals-json/  External JSON API types
templates/              Handlebars templates for generated content
book.toml               mdBook and preprocessor configuration
rust-project-goals.toml Configuration for valid team ask types
justfile                Task runner (just check, just serve, just build)
```

## Essential commands

A Cargo alias is defined in `.cargo/config.toml`:

```
cargo rpg <command>   # shorthand for: cargo run -q --bin rust-project-goals-cli -- <command>
```

The commands you need most often:

| Command                   | Action                                                  |
|---------------------------|---------------------------------------------------------|
| `cargo check --workspace` | Verify all Rust code compiles                           |
| `cargo rpg check`         | Validate all goal and roadmap documents parse correctly |
| `cargo test --workspace`  | Run unit tests                                          |
| `just check`              | Same as `cargo rpg check`                               |
| `just serve`              | Build the book and serve it locally (needs `GH_TOKEN`)  |
| `just build`              | Build the mdBook site (needs `GH_TOKEN`)                |

**Before submitting any change**, at minimum run:

1. `cargo check --workspace`
2. `cargo rpg check`

Both of these run in CI (via `.github/workflows/compile.yml` and `.github/workflows/check.yml`).

Note: `cargo rpg check` validates all milestone directories.  A failure in a preexisting file (not one you touched) is not your problem, but you should be aware of it and not mistake it for a regression you introduced.

## Goal document format

Goal documents live at `src/<milestone>/<name>.md`.  The canonical template is `src/TEMPLATE.md`.  Here is the structure:

### Title

The first line must be a level-1 heading (`#`).  This becomes the goal's title.

### Metadata table

Immediately after the title, a two-column markdown table with the header `| Metadata | |`.  The rows below are recognized.  Rows marked "parsed" are extracted by the parser and validated; rows marked "conventional" are used by humans and the preprocessor but the parser does not reject typos in their names.

| Row name                | Required | Kind         | Notes                                                                           |
|-------------------------|----------|--------------|---------------------------------------------------------------------------------|
| `Point of contact`      | Yes      | Parsed       | Single GitHub `@username`                                                       |
| `Status`                | Yes      | Parsed       | `Proposed`, `Invited`, `Accepted`, `Proposed for mentorship`, or `Not accepted` |
| `Short title`           | No       | Parsed       | Alternate short title for tables; defaults to the `#` heading                   |
| `Tracking issue`        | Note [1] | Parsed       | Must contain `rust-project-goals#NNN` if present                                |
| `Other tracking issues` | No       | Conventional | For issues in other repos; use `org/repo#NNN` format                            |
| `Zulip channel`         | No       | Conventional | Link to Zulip stream                                                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rust-lang/rust-project-goals](https://github.com/rust-lang/rust-project-goals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
