---
trigger: always_on
description: A Decky Loader plugin that syncs a self-hosted RomM library into Steam as Non-Steam shortcuts. Games launch via
---

# decky-romm-sync — Decky Loader Plugin

## What This Is

A Decky Loader plugin that syncs a self-hosted RomM library into Steam as Non-Steam shortcuts. Games launch via
RetroDECK. The QAM panel handles settings, sync, downloads, and BIOS management.

## What belongs in this file

Three things, and nothing else:

1. **Traps** — where an agent would confidently do the wrong thing and have no reason to go look first.
2. **Cross-cutting invariants** — rules that span files, so no single diff shows the whole rule.
3. **Workflow** — how we work here; not derivable from the code.

Everything else is topic depth: `docs/` for humans, `.claude/rules/` for path-scoped coding conventions. A rule with a
mechanical check needs only its one-line statement here (CI carries the enforcement); a rule without one needs its full
statement here or in the rule file that owns its area, because nothing else will catch it.

## Where the details live

Each page below is the current-truth owner of its area, and most carry their own ADR trail. Read the page before working
in the area. **Do not cite ADRs from this file** — an ADR is frozen history (and may be `Proposed` or superseded, which
is invisible at the citation site), so reach it through the page that owns the topic.

- Domain vocabulary — the canonical meaning of project terms — [CONTEXT.md](CONTEXT.md). A glossary, not a spec: use its
  wording in code, issues, and PRs, and add a term there the moment it resolves in discussion.
- Steam shortcuts — appIds, artwork, launch-option writes, removal churn —
  [steam-non-steam-shortcuts.md](docs/architecture/steam-non-steam-shortcuts.md)
- Save-file sync — slots, conflict resolution, negotiate transport, version history —
  [save-file-sync-architecture.md](docs/architecture/save-file-sync-architecture.md)
- Save-sync coverage matrix — [save-sync-coverage.md](docs/architecture/save-sync-coverage.md)
- Removed-game cleanup — deletion authority, admission/leases, claims, recovery bundles —
  [removed-game-cleanup.md](docs/architecture/removed-game-cleanup.md)
- Services, adapters, wiring; connection/token and settings-persistence internals —
  [backend-architecture.md](docs/architecture/backend-architecture.md)
- SQLite schema, aggregate roots, migrations — [database-design.md](docs/architecture/database-design.md)
- Emulator and core selection — [core-emulator-selection.md](docs/architecture/core-emulator-selection.md)
- RetroArch/ES-DE config parsing — [config-source-parsers.md](docs/architecture/config-source-parsers.md)
- Steam Remote Play — [steam-remote-play.md](docs/architecture/steam-remote-play.md)
- **End-user-facing behavior and UI** — setup, configuration, syncing, save-sync, BIOS, troubleshooting —
  `docs/user-guide/`
- Dev setup, dependency management, frontend loop — `docs/contributing/`

## Path-scoped rules — `.claude/rules/`

Coding conventions live in `.claude/rules/*.md`, each carrying a `paths:` frontmatter glob. They are plain Markdown, not
a harness-specific format: an agent that supports path-scoped rules gets one loaded when a matching file is read, and
**every other agent must open the file itself**. Either way it arrives a beat late for code you are **creating** rather
than editing, so the entries below lead with what goes wrong unnoticed. Read the rule that owns an area before writing
new code in it.

- `services.md` — a new service takes **one `config: XxxServiceConfig` kwarg** (frozen, all deps inside); debug logging
  is the injected `DebugLogger`. Neither is checked.
- `python-conventions.md` — Protocol suffixes by shape, `do_<verb>` vs. `_<verb>_io`, docstrings stating the contract
  rather than the behavior, and when a subfolder is justified. **No mechanical check exists for any of it.**
- `adapters-domain.md` — adapters own I/O, domain is pure, aggregate mutations are verb-named after the event
  (`adopt_baseline`, not `update_baseline`). The field-assignment ban is checked; the naming is not.
- `romm-http.md` — an unproven 404 must never become `RommNotFoundError`, which is deletion authority downstream: the
  entity proof is the default and only the three byte-stream fetches opt out. Tests pin both directions; nothing else
  does.
- `bootstrap-wiring.md` — the `main.py` / `bootstrap/` split, and which half of `bootstrap/` new wiring belongs in.
- `callables.md` — the `{success, reason, message}` failure shape and its two carve-outs. Checked.
- `vendored-assets.md` — `_vendor/`, `native/`, `defaults/` are checksum-pinned verbatim copies. The checksums are
  checked; the reflex to fix the upstream artifact instead of the copy is not.
- `testing-backend.md` — test tiers, gate tests, vendored conformance vectors.
- `testing-frontend.md` — the `@decky/api` event harness, non-vacuous catch assertions.
- `comments.md` — an inline comment is the exception: only an outside-world fact, a road not taken, or a constraint the
  code cannot express. Re-read the comment on the line you touch — a stale one is worse than none, because it is
  believed and nothing in the toolchain contradicts it. **No mechanical check exists.**

## Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielcopper/romm-tender](https://github.com/danielcopper/romm-tender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
