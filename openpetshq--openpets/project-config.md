---
trigger: always_on
description: A full codemap is available at `codemap.md` in the project root.
---

## Repository Map

A full codemap is available at `codemap.md` in the project root.

Before working on any task, read `codemap.md` to understand:
- Project architecture and entry points
- Directory responsibilities and design patterns
- Data flow and integration points between modules

For deep work on a specific folder, also read that folder's `codemap.md`.

## Documentation Map

`docs/` holds the maintained, conceptual documentation — the narrative layer on
top of the codemaps. It explains concepts and contracts and points at the files
that own each behavior (it deliberately avoids pasting code, which rots). Start
at `docs/README.md`, then read the doc for the area you're touching:

- **`docs/architecture.md`** — system overview, runtime topology, package spine,
  end-to-end flows, cross-cutting invariants, glossary. Read first.
- **`docs/desktop.md`** — the Electron app: process model, tray/Control Center,
  windows, app state, lifecycle, security, logging, CSP.
- **`docs/ipc.md`** — local IPC protocol + `@open-pets/client`: discovery,
  transports, the lease model, request surface, security.
- **`docs/pets.md`** — pet model, reactions→animations→speech, installation,
  Codex pets, motion.
- **`docs/catalog.md`** — pet/plugin catalog contracts (v3/v2), pagination,
  search, R2 ZIP hosting, and how the app consumes them.
- **`docs/agent-integrations.md`** — Claude/MCP/OpenCode/Cursor/Pi + the CLI.
- **`docs/plugins.md`** — plugin platform: manifest, permissions, runtime,
  sandbox, install paths, packaging/publishing, troubleshooting.
- **`docs/official-plugins.md`** — companion-first direction, official lineup,
  bundling/enabled defaults, right-click action strategy.
- **`docs/sdk.md`** — public SDK v3 contract + the deterministic test harness.
- **`docs/i18n.md`** — translations across host UI, reaction speech, and plugins.
- **`docs/development.md`** — DX: layout, command surface, dev modes, releases.
- **`docs/testing-and-validation.md`** — tests, contracts, release validators,
  catalog verification, and what "production-valid" means before shipping.

When you change behavior, update the matching `docs/*.md` in the same change.
Ongoing improvement ideas / known issues are tracked in the root `improvements.md`.

## Tests Must Protect Behavior

Tests are evidence of a user-visible behavior, public contract, or a plausible
regression—not a record of the implementation that happened to be written.

- Before adding a test, state the bug or contract it would catch. If there is
  no concrete answer, do not add it.
- Prefer a small assertion of the observable outcome over exact internal calls,
  helper sequencing, incidental data shapes, generated asset/source mappings,
  arbitrary versions/counts, or full wording snapshots.
- Do not test private implementation details merely to increase coverage.
  Test assets only when their format or integrity is itself a shipped contract.
- Keep one behavior-focused purpose per test. Remove no-op assertions,
  duplicate coverage, and brittle snapshots/regexes that fail on harmless
  refactors or copy changes.
- When fixing a bug, add the narrowest regression test that fails without the
  fix. When reviewing existing tests, delete or rewrite tests that do not
  protect a plausible failure mode.

## Catalog Direction

Catalog v2 is legacy and exists only for old app versions/fallback compatibility.
For new work, migrations, and Control Center UI, do not optimize for v2 behavior.
Use catalog v3 (`thumbnail`, `spritesheet`, paginated pages, and search index) as the source of truth.

See `docs/catalog.md` for the v3/v2 contracts, pagination/search, ZIP hosting, and how the app consumes them.

## Forward-Only Product Direction

Move the current app forward; do not keep legacy compatibility code, duplicate
paths, stale shims, or old behavior in current runtime code unless it is required
so older released app versions can still open/use versioned catalogs or existing
published data. Prefer clean migrations, versioned catalog/data boundaries, and
removing obsolete code over preserving backwards-compatible branches. The bar is:
old app versions should not break catastrophically, but the current app should
not carry legacy bloat for deprecated plugin/catalog behavior.

## Plugin Docs

Before changing plugin platform code, official plugins, plugin catalog generation, plugin packaging, plugin runtime behavior, or plugin-facing UI, read:
- `docs/plugins.md` for the current plugin platform architecture, manifest/runtime rules, local development workflow, publishing commands, and troubleshooting notes.
- `docs/official-plugins.md` for the companion-first plugin direction, current official plugin lineup, bundling defaults, and right-click plugin action strategy.

When plugin work is finished, update these docs if behavior, commands, manifests, plugin IDs, default bundled/enabled status, catalog workflow, permissions, or the planned plugin lineup changed. Do not leave plugin docs stale after implementation.

For plugin release/catalog work, run the release validator before shipping:
- `pnpm plugins:package`
- `pnpm plugins:validate-release`
- after deploy/R2 upload, `pnpm plugins:validate-live`

The validator exists to catch production-breaking plugin mistakes: unresolved

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenPetsHQ/openpets](https://github.com/OpenPetsHQ/openpets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
