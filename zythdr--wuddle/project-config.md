---
trigger: always_on
description: > This file provides architectural context, product direction, invariants, and development guidance for AI assistants working on Wuddle.
---

# Wuddle — Development Context

> This file provides architectural context, product direction, invariants, and development guidance for AI assistants working on Wuddle.

## Product Vision

Wuddle is a native desktop launcher and manager for legacy World of Warcraft clients.

It aims to provide:

- A GitAddonsManager-compatible addon manager with safer installation flows and additional collection-management features.
- DLL mod installation, updating, repair, removal, and enable/disable controls.
- Separate game profiles, each with its own WoW directory, tracked projects, database, and launch configuration.
- Client-aware support for Vanilla 1.12.1, TBC 2.4.3, and WotLK 3.3.5.
- Straightforward game launching through Auto, Lutris, Wine, or Custom commands.
- Optional secure WotLK 3.3.5 auto-login through Awesome WotLK.
- Useful diagnostics that users can safely attach to GitHub issues.
- A polished, self-explanatory interface for users who may not be comfortable managing Git repositories, DLLs, Wine commands, or addon folders manually.

The addon-management goal is effectively “GitAddonsManager plus additional features and a friendlier UX.” When Wuddle and GAM support equivalent behavior, Wuddle should remain compatible with GAM’s filesystem layouts. Wuddle-specific features may add metadata outside GAM-managed worktrees when GAM has no equivalent concept.

## Product Principles

- Preserve user data and existing installations.
- Prefer transactional operations that can be cancelled or rolled back safely.
- Keep profiles completely isolated from one another.
- Never expose credentials, tokens, command arguments, or private paths in logs.
- Keep advanced behavior understandable through clear labels, dialogs, tooltips, and actionable errors.
- Support Linux and Windows deliberately; do not treat either platform as an afterthought.
- Prefer compatibility with existing user setups over forcing migrations or reinstalls.
- Avoid server-specific assumptions. Wuddle manages legacy WoW clients, not one particular private server.
- Prefer a modular architecture where it improves cohesion, replaceability, testability, and isolation between unrelated features.

## Architecture

Wuddle consists of three independent Rust crates. There is no root Cargo workspace.

### Modularity Direction

Wuddle should remain a modular monolith. Shared application coordination may stay centralized, but substantial features should be implemented behind focused engine and frontend boundaries so they can be repaired, replaced, disabled, or removed without destabilizing unrelated behavior.

- Keep `Engine`, `App`, and `Message` as coordinators rather than eliminating them solely for modularity.
- Avoid continuing to grow broad files such as `lib.rs`, `app/mod.rs`, and `service.rs` when new behavior has a clear feature or domain boundary.
- Prefer focused engine modules for domain, security, compatibility, and filesystem behavior.
- Prefer focused frontend modules for feature state, update handling, service adapters, dialogs, and reusable views where the feature is substantial enough to justify them.
- Extract existing monolithic areas incrementally when related behavior is already being changed. Do not perform risky repository-wide rewrites purely to reduce line counts.
- Modularize around cohesive responsibilities, independent testing, and change isolation—not arbitrary file-size targets or layers of abstraction.
- Keep trivial shared behavior close to its caller when extracting it would only add indirection.

### Crate Structure

```text
wuddle-engine/                 Core Rust library; no UI dependencies
├── Cargo.toml                 Engine features and dependencies
└── src/
    ├── lib.rs                 Engine API, scanning, updates, staging, conflicts, rollback
    ├── db.rs                  SQLite schema and migrations
    ├── model.rs               Repository, release, asset, and installation types
    ├── install.rs             Archive extraction and final file deployment
    ├── direct.rs              Direct archive URL support
    ├── gam_compat.rs          GitAddonsManager discovery and layout compatibility
    ├── diagnostics.rs         Privacy-safe diagnostic event interface
    ├── auto_login.rs          Optional secure auto-login domain and vault logic
    └── forge/
        ├── mod.rs             Forge detection and dispatch
        ├── github.rs          GitHub API support
        ├── gitlab.rs          GitLab API support
        ├── gitea.rs           Gitea and Codeberg API support
        └── git_sync.rs        Git clone, update, branch, and remote handling

wuddle-iced/                   Active native desktop application built with Iced
├── Cargo.toml                 Application version, features, and dependencies
├── assets/                    Fonts, icons, and bundled visual assets
└── src/
    ├── main.rs                Startup, storage, single-window setup, diagnostics, Iced setup
    ├── app/mod.rs             Main application state, subscriptions, and view composition
    ├── message.rs             Application messages and events
    ├── service.rs             Asynchronous bridge to wuddle-engine

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZythDr/Wuddle](https://github.com/ZythDr/Wuddle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
