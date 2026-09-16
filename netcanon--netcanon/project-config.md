---
trigger: always_on
description: These directives govern all development in this repository.  Follow them in
---

# Netcanon — Contributor Directives

These directives govern all development in this repository.  Follow them in
every session without being asked.

**Project orientation:** if this is your first read, also skim
[`README.md`](README.md) for the quickstart and
[`ARCHITECTURE.md`](ARCHITECTURE.md) for the 4-layer design.
[`CHANGELOG.md`](CHANGELOG.md) is the authoritative current-state
shipping log; [`translator-plans.txt`](translator-plans.txt) is the
slower-changing architectural sketch (most R / GAP / Phase items
now `[SHIPPED]`).  [`tests/fixtures/real/RESULTS.md`](tests/fixtures/real/RESULTS.md)
tracks per-codec certification state.

---

## Two concerns, one FastAPI app

Netcanon handles two independent (but co-hosted) concerns:

1. **Backup** — `netcanon/collectors/` + `netcanon/api/routes/backups.py`.
   Pulls raw `running-config` (or vendor equivalent) from devices over
   SSH / NETCONF / REST and stores in `configs/<host>.<ext>`.  Mocked in
   tests at a single entry point: `get_collector`.
2. **Migration** — `netcanon/migration/`.  Translates a stored backup
   from one vendor's native config to another through a shared
   `CanonicalIntent` tree.  Per-vendor codecs under
   `netcanon/migration/codecs/` — see
   [`netcanon/migration/codecs/README.md`](netcanon/migration/codecs/README.md)
   for authorship guide.

A change to one concern rarely touches the other.  When in doubt about
where code belongs, ask: "does this fetch bytes off a device?" → backup.
"Does this translate canonical representation?" → migration.

---

## Parallel Platform Development

Netcanon ships on two platforms.  Both must always be kept at feature parity.

| Platform | Package | Entry point |
|----------|---------|-------------|
| Web (browser) | `netcanon/` | `uvicorn netcanon.main:app` |
| Desktop (Windows) | `netcanon_desktop/` | `python -m netcanon_desktop` |

**Distribution variants — NOT separate platforms.**  Docker
(`ghcr.io/netcanon/netcanon` GHCR primary + `docker.io/netcanon/netcanon`
Docker Hub mirror), `pip install netcanon`, and the Windows MSI are
all distribution methods, not platforms.  Docker and pip both produce
a **web-platform** install (the container's entrypoint is
`uvicorn netcanon.main:app` — same code path as host-installed web);
the MSI produces a **desktop-platform** install.  None of them require
their own parity test row; a feature that lands on the web platform
automatically reaches Docker + pip users on the next release tag.
The doc-sync table below has a "packaging / distribution workflow
changes" row covering edits to `Dockerfile`, the publish workflows,
and the supply-chain attestation surfaces.

**Rule:** whenever a functional feature is added or changed on one platform, the
equivalent must be implemented on the other platform in the same commit (or the
same branch, if it spans multiple commits).  Never leave one platform behind.

This applies to:
- New API endpoints
- New UI pages or views
- New device definition schema fields
- New collector strategies
- New storage backends
- New application settings that affect behaviour

---

## Platform-Specific Exceptions

The following are intentionally platform-specific and do **not** require
cross-platform equivalents:

**Desktop only**
- System tray icon (Show / Preferences… / Open configs folder / Quit menu)
- MSI installer, Start Menu shortcut, taskbar pinning
- Embedded WebView window management (hide-to-tray on close, restore on show)
- Native window chrome (title bar, window icon via `.ico`)
- `setup_desktop.py` build script and `netcanon_desktop/` package
- **Open in text editor** (`config-open-btn`, `POST /api/v1/configs/{filename}/open`) —
  calls `os.startfile()` on the local filesystem.  Only meaningful when the server runs
  on the same machine as the user.  Enabled via `Settings.open_in_editor = True` in
  `netcanon_desktop/settings.py`.  The web platform equivalent is the existing
  **View** button (`config-view-link`) which renders the file in the browser.
- **Preferences dialog** (`netcanon_desktop/preferences_dialog.py`) — operator-
  configurable paths (configs / definitions / data dir), embedded-server port,
  and toggles.  Persisted to `%APPDATA%\Netcanon\preferences.json`.  Equivalent
  to the web platform's `NETCANON_*` env-var / `.env` configuration surface;
  desktop operators have no shell-level knob, so the dialog is the equivalent
  affordance.  PySide6 widgets carry `setObjectName()` IDs following the
  `pref-dialog-<field>-<action>` convention (the desktop equivalent of
  `data-testid` since Qt has no native test-id attribute).
- **Single-instance enforcement** (`netcanon_desktop/single_instance.py`) —
  Windows named mutex (`Global\NetcanonSingleInstance_v1`) refuses to launch a
  second copy.  Without this guard the duplicate process fails to bind the
  embedded server's TCP port and surfaces as a confusing fatal-error MessageBox;
  the friendly "already running" hint is much more discoverable.  No-op on
  non-Windows platforms.

**Web only**
- Interactive Swagger API docs at `/docs` — the web browser opens this freely;
  it is accessible from the desktop too but not surfaced in the desktop UI
- `--host` / `--port` flags for public network binding — the desktop always

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netcanon/netcanon](https://github.com/netcanon/netcanon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
