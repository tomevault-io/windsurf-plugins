---
trigger: always_on
description: This repository is the Omarchy plugin target for Lacuna, with the standalone Lacuna project treated as the source reference. The current structure is intentionally small:
---

# Repository Guidelines

## Project Structure & Module Organization

This repository is the Omarchy plugin target for Lacuna, with the standalone Lacuna project treated as the source reference. The current structure is intentionally small:

- `docs/`: current project documentation, design-system specs, screenshots, and reference docs.
- `docs/plans/`: lifecycle-organized planning records. Put current work in `active/`, non-blocking ideas and drafts in `proposed/`, finished records in `completed/`, and reverted or superseded history in `archive/`; keep `docs/plans/README.md` synchronized.
- `lacuna.*/`: one top-level Omarchy plugin directory per Lacuna surface or widget. This flattened layout is required for `omarchy plugin source add` repo installs.
- `lacuna.menu/`: menu/sidebar plugin, with `menu/`, `components/`, `services/`, and `assets/`.
- `config/`: example configuration should live here, such as `settings.example.json`.

Keep plugin code self-contained under its plugin directory. Do not depend on the repository root as a runtime import path.

## Repository Exploration

This repo has a checked-in Graphify knowledge graph under `graphify-out/`. For architecture questions, file-relationship tracing, or "where does this behavior live?" investigations, query Graphify first before broad `rg`/grep-style exploration. Use `graphify query "<question>"` from the repository root, then follow up with targeted `rg`, `sed`, or file reads to verify exact implementation details before editing.

## Build, Test, and Development Commands

Use the repository check script for local validation:

- `./scripts/check.sh`: validate example JSON, manifests, vendored-file equality, optional `qmllint`/`shellcheck`, and the Python test suite.
- `python3 -m pytest`: run the test suite directly.
- `rg --files`: list tracked source-like files quickly.
- `find . -maxdepth 2 -path './lacuna.*' -print`: inspect plugin layout.
- `./scripts/dev deploy <plugin-id>`: developer-only live deploy from this checkout into `~/.config/omarchy/plugins/`, rescan, restart Omarchy shell, and verify the installed copy matches the repo.
- `omarchy plugin rescan`: ask Omarchy shell to reload installed plugins.
- `OMARCHY_PATH="$HOME/.local/share/omarchy" omarchy-shell shell summon lacuna.menu "{}"`: smoke-test the menu plugin once implemented.

For local testing, copy or symlink a plugin directory into `~/.config/omarchy/plugins/<plugin-id>/`, then rescan or restart Omarchy shell. No plugin should start a second Quickshell process.

## Live Install Verification

When changing behavior that the running Omarchy shell should exhibit, repository edits and tests are not enough. Before saying a user-visible plugin issue is fixed:

- Run `./scripts/dev deploy <plugin-id>` for each changed installed plugin, or `./scripts/dev deploy --all --only-changed` to deploy every repo plugin whose live copy differs from this checkout or is missing.
- Use `./scripts/dev deploy <plugin-id> --dry-run` to preview the deploy/rescan/restart/verify steps.
- If bypassing the helper, deploy the changed plugin into the live install at `~/.config/omarchy/plugins/<plugin-id>/` or confirm that path is a symlink to the edited repo directory.
- If using `omarchy plugin update <plugin-id>`, remember it installs from the committed source state; it will not include uncommitted repo edits. Prefer `./scripts/dev deploy` for uncommitted fixes.
- The dev helper runs `omarchy plugin rescan`, restarts Omarchy shell by default, and verifies the installed files match this checkout. Do not skip that verification.
- Only report a live shell issue as fixed after the installed copy and the running shell have been refreshed. If you only changed the repo, say it is implemented in the repo but not yet deployed live.

## Coding Style & Naming Conventions

Use QML for plugin entry points and keep roots compatible with Omarchy plugin contracts: bar widgets expose an `Item`; menu/panel surfaces implement `open(payloadJson)` and `close()`. Name plugin directories with full IDs, for example `lacuna.script-pill`. Prefer `Widget.qml` for bar-widget entry points and `Menu.qml` for the menu entry point.

Use 2-space indentation for JSON and QML unless a copied source file already has a consistent style. Store bar-widget user options in the plugin manifest schema so Omarchy Settings writes them inline to `~/.config/omarchy/shell.json`. Keep `~/.config/omarchy/lacuna/settings.json` for Lacuna runtime/app state only.

## Flyout Surface Geometry

The authoritative spec for Lacuna's seam/connector geometry is
[`docs/lacuna-design-system/02-geometry.md`](docs/lacuna-design-system/02-geometry.md).
Read it before touching any attached flyout. The load-bearing invariants:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OldJobobo/lacuna-shell](https://github.com/OldJobobo/lacuna-shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
