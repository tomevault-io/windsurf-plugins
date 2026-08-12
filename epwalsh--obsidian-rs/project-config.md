---
trigger: always_on
description: This file provides guidance to coding agents when working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working in this repository.

## Project Overview

`obsidian.rs` is a Rust library and CLI for working with Obsidian vaults. It is structured as a Cargo workspace with sub-crates for various features:
- `obsidian-core` (crate name: `obsidian-rs-core`; library name: `obsidian_core`): core API used by the other sub-crates.
- `obsidian-cli` (crate name: `obsidian-rs-cli`; binary name: `obsidian`): command-line interface exposing `search`, `note`, `tags`, and `check` commands. The `note` subcommand supports `backlinks`, `extract`, `merge`, `patch`, `rename`, and `update`. `extract` can move a named section or explicit span into a new note, keeps the source section heading when extracting by section, and defaults the source replacement to a wiki link to the new note. The `check` command reports duplicate IDs or aliases, broken links, and stranded notes, while ignoring `README.md`-style notes for stranded-note reporting.
- `obsidian-mcp` (crate name: `obsidian-rs-mcp`; binary name: `obsidian-mcp`): MCP (Model Context Protocol) server over STDIO transport. Exposes vault operations as MCP tools: `read_note`, `list_notes`, `list_backlinks`, `write_note`, `extract_to_note`, `append_to_note`, `patch_note`, `update_note`, `search_notes`, `rename_note`, `list_tags`, `search_tags`, `check_vault`. Vault path resolved in order: `--vault <PATH>` CLI arg, then `OBSIDIAN_VAULT` env var, then `open_from_cwd()`. Uses the `rmcp` crate with `tokio` for async handling of blocking vault I/O. `check_vault` reports duplicate IDs or aliases, broken links, and stranded notes.
- `obsidian-lsp` (crate name: `obsidian-rs-lsp`; binary name: `obsidian-lsp`): Language Server Protocol server over STDIO transport. Resolves the vault with the same precedence as `obsidian-mcp` and currently provides initialization, startup work-done progress while indexing the cached vault, full-document sync for open buffers, cached vault state through `obsidian_core::Vault::open_cached()` with open-buffer overlays, watched Markdown file and workspace file-operation refreshes, health diagnostics for broken links, duplicate IDs or aliases, stranded notes, and trailing whitespace, document formatting that trims trailing whitespace and normalizes parseable YAML frontmatter, hover metadata for note links and tags, document links with resolve support, document symbols for note structure, workspace symbols for vault-wide note/tag/heading search, backlinks-based references, go-to-definition for note links, heading anchors and nested sub-anchors, completion for wiki/markdown note links and tags, create-note quick fixes for broken note links, extract-to-note execute-command support with span- and heading-based code action previews that derive default heading-extract paths from heading ancestry, duplicate ID/alias quick fixes, wiki/markdown link conversion refactors, wiki-link missing-heading quick fixes, filename-first note rename via `textDocument/prepareRename` / `textDocument/rename` with backlink updates, and tag references/definition/rename for inline and frontmatter tags.

## Workspace Structure

- `Cargo.toml` — workspace root
- `obsidian-core/` — the core library crate
  - `src/lib.rs` — library entry point
  - `src/note.rs` — defines the `Note` struct; `content` is `Option<String>` (not loaded by default); `links` and `tags` are always pre-computed; `from_path()` omits content, `from_path_with_content()` retains it; `write()` requires content, `write_frontmatter()` reads body from disk. Filename-derived default IDs are normalized to lowercase ASCII kebab-case with Unicode transliteration via `default_note_id_for_path()`. `tags: Vec<LocatedTag>` holds all tags — frontmatter tags have `location: Location::Frontmatter`, inline body tags have `location: Location::Inline(InlineLocation)`
  - `src/link.rs` — parsing markdown/wiki/embedded links
  - `src/search.rs` — `find_note_paths()` for recursively finding `.md` files (public)
  - `src/health.rs` — `VaultHealthReport`, `DuplicateId`, `DuplicateAlias`, `BrokenLink`, `StrandedNote`, `NoteRef` types returned by `Vault::check()`, plus reusable `check_notes()` / `backlinks_from()` helpers for cached note sets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epwalsh/obsidian.rs](https://github.com/epwalsh/obsidian.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
