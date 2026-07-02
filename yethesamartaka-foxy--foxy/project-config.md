---
trigger: always_on
description: This repo is a Rust edition 2024, version 1.96 desktop app named `Foxy` (Arma 3 mod updater). It uses egui/eframe for UI and **Turso** (pure-Rust, async-native, SQLite-compatible engine) for core storage, accessed through the `src/core/db/` seam.
---

﻿# AGENTS.md
## Foxy repo guide for Codex, Antigravity, Gemini 3, Claude

This repo is a Rust edition 2024, version 1.96 desktop app named `Foxy` (Arma 3 mod updater). It uses egui/eframe for UI and **Turso** (pure-Rust, async-native, SQLite-compatible engine) for core storage, accessed through the `src/core/db/` seam.

Keep this file as the compact root router. Put detailed conventions in `conventions/` or nested `AGENTS.md` files so agents load them only when relevant. When workflows, commands, schemas, or conventions change, update the matching agent docs in the same change.

---

## Always follow
- Make small, reviewable changes that compile and pass the relevant checks.
- Match existing style and local patterns; avoid large refactors unless explicitly requested.
- Prefer existing helpers and clear code over new abstractions. Add an abstraction only when it removes real duplication or matches an established local pattern.
- Keep files focused; split growing modules into well-named directory modules before they become hard to review.
- Do not reformat unrelated code, rename unrelated items, or change dependencies unless the task requires it.
- Do not log secrets, tokens, or user file paths.
- Never use em dashes (—) or en dashes (–) in any text (code, comments, docs, UI strings, commit messages, changelog). Use a plain hyphen `-` instead.
- Do not edit runtime `database.db`, logs, caches, backups, temp patch artifacts, or user config files unless explicitly requested.
- Use `rg`/`rg --files` for discovery, inspect the nearest `mod.rs`, and confirm behavior in code before changing it.
- Check for nested `AGENTS.md` files before editing a subtree; more-specific instructions apply to files under that directory.
- When editing locale or other non-ASCII text, do not paste Unicode through PowerShell here-strings or other paths that may replace characters with literal `?`. Use UTF-8-safe edits and audit changed strings for `?` corruption before handoff.

---

## Quick map
- App starts in `src/main.rs`; GUI flow is `src/ui/window.rs` -> `src/ui/launcher.rs` -> `src/ui/app/mod.rs` (`Foxy`).
- CLI flow is `src/cli/args.rs` -> `src/cli/mod.rs` -> `src/cli/commands/`; output contracts live in `src/cli/output.rs` and `src/cli/exit_codes.rs`.
- UI code is split between state/behavior in `src/ui/app/`, screens in `src/ui/views/`, shared types in `src/ui/types/`, and shared presentation helpers in `src/ui/palette.rs`, `src/ui/fonts.rs`, `src/ui/i18n.rs`, and `src/ui/app/ui_helpers/`.
- Core code lives in `src/core/`: API/sync orchestration in `src/core/api/`, the Turso DB seam in `src/core/db/` (engine init/bootstrap in `src/core/tasks/db_turso.rs`), query/domain models in `src/core/models/`, task pipelines in `src/core/tasks/`, and shared utilities in `src/core/utils/`.
- The authoritative schema is the folded bootstrap file `sql/turso_schema.sql`; `migrations/` holds the historical SQLite migrations (no longer applied). Data/manifest references live in `sql/` and `examples/`.
- Server-side repository generator lives in `foxy-server-backend-cli/`; standalone helper tools live in `tools/`.
- Shareable repo-maintained agent skills live in `skills/`; Claude Code project-skill entrypoints live in `.claude/skills/` and should point back to the repo-maintained source.
- Runtime data defaults to `%APPDATA%\Foxy` on Windows and can be overridden with `FOXY_CONFIG_DIR` or CLI `--config-dir`.

---

## Load when needed
- UI layout, app state, widgets, palette, fonts, or egui interaction: `conventions/UI_CONVENTIONS.md` and `src/ui/AGENTS.md`.
- Agentic GUI automation, semantic UI snapshots, screenshots, FPS probes, simulated scroll/click/key input, or Codex/Claude desktop driver work: `skills/foxy-gui-driver/SKILL.md`, then `conventions/UI_CONVENTIONS.md` and `src/ui/AGENTS.md` if UI code changes are needed.
- User-facing text, locales, pluralization, RTL, i18n checker changes, or exact-English fallback cleanup: `skills/foxy-locale-translator/SKILL.md` and `conventions/i18n_CONVENTIONS.md`.
- Accessibility, keyboard flow, focus, status/error clarity, contrast, or CLI readability: `conventions/ACCESSIBILITY_CONVENTIONS.md`.
- CLI commands, flags, output contracts, `--json`, `--dry-run`, or destructive actions: `conventions/CLI_CONVENTIONS.md`.
- Core, the Turso data layer (the `src/core/db/` seam, `tasks/db_turso.rs`), schema, transactions, filesystem/network safety, or sync tasks: `conventions/CORE_CONVENTIONS.md` and `src/core/AGENTS.md`.
- Sync algorithm, quick scan, remote refresh, tree hashing, download queue, delta patch, pending updates, or sync performance: `conventions/SYNC_ALGO_CONVENTION.md`.
- Performance budgets, speed-of-light ratios, `SOL` log lines, perf baselines, or regression analysis: `conventions/SPEED_OF_LIGHT.md`.
- Tests, validation commands, pure helper coverage, or regression tests: `conventions/TESTING_CONVENTIONS.md`.
- Config examples, manifest examples, generated repository JSON, or sample fixtures: `conventions/EXAMPLES_CONVENTIONS.md`.
- Changelog entries: `conventions/CHANGELOG_CONVENTIONS.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YetheSamartaka-Foxy/Foxy](https://github.com/YetheSamartaka-Foxy/Foxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
