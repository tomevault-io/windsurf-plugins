---
trigger: always_on
description: **firefox-scripts** installs and keeps up to date the helper scripts that let Firefox-family
---

# Repository Guidelines

## Scope

**firefox-scripts** installs and keeps up to date the helper scripts that let Firefox-family
browsers (Firefox stable/Nightly/Developer Edition, Waterfox, Zen, LibreWolf, Floorp) run legacy
(non-WebExtension) extensions. Three parts:

1. **Native C installer** (`installer/`) — detects a running browser, serves an embedded web UI over
   `127.0.0.1:8777`, and copies two packages: **fx-folder** (`core/fx-folder/`) to the browser
   install dir, **utils** (`core/chrome/utils/`) to `ProfD/chrome/utils/`.
2. **In-browser updater** (`core/chrome/utils/updater/`) — daily hash-manifest check; self-updates
   and opens the updater tab (chrome-privileged page shipped in `updater-ui.zip`).
3. **Updater tab UI** (`tools/publish/remote-ui/` → `updater-ui.zip`) — the tab's HTML/JS/CSS and
   logos; `scriptsUpdater.sys.mjs` (utils.zip) keeps it current before opening the tab.

Architecture deep dive: `docs/DEVELOPING.md` (structure, installer/updater flow, publishing) and
`docs/auto-updater.md`.

## Critical Rules

- **Never hand-edit generated files.** They are gitignored build products, regenerated on demand
  from their sources (see the Generated files section).
- **Do not publish or upload** unless the user explicitly asks. Use `snapshot:*` for offline
  validation.
- **Never merge a PR without the user's explicit approval** — open PRs for review and wait.
- **Never expose the GitHub token.** It lives only in an untracked root `.env` under the fixed
  `GITHUB_TOKEN_VAR` name — do not commit, log, or rename that variable.
- **Do not introduce Python**; build and asset tooling uses Node.js.
- Prefer the smallest change that solves the requested problem. Avoid unrelated refactoring,
  formatting, renaming, dependency updates, or architectural changes.
- Do not base work on `.local` files/dirs; they are local drafts and gitignored.
- Preserve upstream provenance in `core/`: files outside `updater/` come from
  xiaoxiaoflood/firefox-scripts (MPL-2.0); `updater/` is custom (MIT). Avoid unrelated changes to
  upstream-derived files.
- Read the relevant `docs/`, plus the decision log `docs/decisions/index.md`, before architectural
  or design changes.
- Keep `docs/ci-inventory.md` synchronized when changing workflow names, triggers, path filters,
  gates, scheduled jobs, or watchdog behavior. Keep workflow job names and required/advisory status
  semantics accurate.

## Architecture invariants

Full design notes live in `docs/` (`DEVELOPING.md`, `auto-updater.md`, `status-logic.md`). These
facts most often cause bugs:

- **Single source of truth** is `config/installer.conf`: it generates `installer/src/_config.h` (C),
  `core/chrome/utils/updater/updater-config.sys.mjs` (updater), and feeds `tools/publish/paths.js`.
  The generated-file set itself is registered in `tools/publish/generatedRegistry.mjs` — the one
  list the generators, the publish hashes and the zip re-adds read; the registry ↔ hash-inputs tests
  (`test/unit/generatedRegistry.test.mjs`) fail when the wiring is missing (ADR 0008).
- **Hash-based detection.** Per-package SHA-256 manifest (`hashes.json`): for each file
  `sha256(rel_path + '\n') + sha256(file_bytes)`, files sorted case-insensitively; a missing file
  contributes only its path. Equal → Up to date; differs with at least one file present → Update
  available; zero files present → Not installed. JS ref `tools/publish/hashUtils.mjs`; C twin
  `compute_directory_sha256` in `installer/src/detect_browser.c`; cross-checked by
  `installer/test/test_hash.mjs`.
- **The C installer does zero network I/O** — the browser tab fetches the zips, manifest and release
  lists from CORS-enabled hosts and POSTs the bytes to the local server.
- **Updater flow:** daily check of utils + fx-folder → keep `updater-ui.zip` current → open one
  trusted tab → verify, extract, copy; admin-protected dirs use a freshly downloaded standalone
  helper binary (one elevation prompt; cancel is a distinct exit code). A **remote test/dev build**
  whose own manifest is unreachable falls back to the stable channel's manifest and auto-migrates;
  local snapshots keep the silent exit. Full flow: `docs/auto-updater.md`; status semantics:
  `docs/status-logic.md`; the dead-channel decision: ADR 0026.
- **Publishing:** `--mode=prod` → `latest` release + `gh-pages` (branch `main` only, CI-only — runs
  the full cross-OS binary matrix); `--mode=dev` → disposable `dev-build-<id>` branch (branch-only;
  `--note` adds an RC-style prerelease page), `-dev` artifact names, served via jsDelivr. Requires a
  clean worktree; real runs need `GITHUB_TOKEN_VAR`.
- **Gotchas:** the build date is DERIVED, not hand-stamped (ADR 0036): per-binary git dates from the
  same input set the publish hash uses; the conf has no date key anymore. Waterfox skips
  `BootstrapLoader.js` in `config.js`. Per-package skip prefs
  `extensions.firefox-scripts.skippedHash.<pkg>`; one daily gate pref `lastScriptsCheckDate` (ADR
  0012 — written by the up-to-date check and by the shown updater tab). `versionInfo.json` is
  obsolete (excluded from zips; installed copies cleaned by `installer/src/obsolete_files.h`).

## Decision records


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onemen/firefox-scripts](https://github.com/onemen/firefox-scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
