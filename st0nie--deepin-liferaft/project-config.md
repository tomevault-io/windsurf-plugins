---
trigger: always_on
description: These instructions apply to the entire `deepin-liferaft` repository.
---

# AGENTS.md

## Scope

These instructions apply to the entire `deepin-liferaft` repository.

## Project

Memory Liferaft (`deepin-liferaft`, Chinese 内存救生筏) is a single-binary DTK 6 application. A systemd user service runs it with `--hidden`; sustained Fedora-style systemd-oomd pressure or swap conditions open a DTK force-quit dialog. Application boundaries are DDE `app-DDE-*` launch groups and `app-*.scope` scopes, both under `user@UID.service/app.slice`.

## Layout

- `main.cpp`: monitoring policy, cgroup ownership, lifecycle cleanup, and DTK UI
- `CMakeLists.txt`: build, CTest, translations, resources, and installation
- `resources.qrc`: embedded application resources
- `data/icons/`: application icon source
- `data/whitelist`: packaged whitelist defaults (package level of the three-level merge)
- `translations/`: Qt translation sources (`.ts`) for `zh_CN`, `en_US`, `ja_JP`, `ko_KR`
- `deepin-liferaft.desktop`: DDE application entry
- `deepin-liferaft.service`: systemd user service
- `deepin-liferaft.1`: command manual page
- `debian/`: Debian packaging
- `README.md`: user and operator documentation
- `LICENSE`: GPL-3.0-or-later

Do not edit or commit generated content under `obj-*/` or `build/`, `debian/deepin-liferaft/`, debhelper stamp/helper files, `debian/files`, package artifacts (`.deb`, `.buildinfo`, `.changes`, `*.tar.xz`), `deepin-liferaft-*.tar.gz` tarballs, screenshots (`*.png`), logs, local CodeGraph databases, or `.pi-subagents/`/`progress.md` scratch state.

## Safety Invariants

Memory-pressure code is safety critical. Preserve all of these invariants:

- Parse cgroup `memory.pressure` `full avg10`, not `some`.
- Treat failed or incomplete PSI, `memory.stat`, cgroup value, and `/proc/meminfo` reads as invalid samples. Never convert failure into zero usage or a trigger. The dialog's memory and swap lines follow the same rule: an invalid sample prints no numbers.
- Never freeze the cgroup containing Memory Liferaft itself.
- Read `cgroup.freeze` before freezing. Do not claim or thaw a cgroup already frozen by another component.
- Track every cgroup successfully frozen by this process.
- Resume all owned cgroups before accepting a window close, normal process exit, `SIGTERM`, or `SIGINT`. When the window is closed while it still owns frozen cgroups, confirm with the user first (`needsCloseConfirmation`, covered by `--self-test`) and keep the window open if they cancel. `SIGTERM`, `SIGINT`, and service stops thaw without asking.
- If thaw fails, retain ownership and retry. Never silently clear the recovery set.
- After `cgroup.kill`, thaw a surviving cgroup before releasing ownership.
- Keep frozen applications visible in the list even when they fall outside the normal top-ten memory list.

Do not test freeze/kill behavior on real user applications. Use a disposable systemd user cgroup or temporary fake control files.

The systemd user service sets `MemoryMin=16M` so the monitor's own cgroup keeps hard reclaim protection and can still poll PSI under extreme pressure (requires systemd 253+; older versions ignore the property). Preserve it when editing `deepin-liferaft.service`.

## Policy

Current workstation policy:

- poll interval: 1 second
- pressure: `full avg10 > 50%` for 20 seconds
- reclaim must have occurred in the previous 30 seconds
- system memory and swap trigger: both above 90%
- swap candidate: above 5% of total swap
- post-action delay: 15 seconds

Pressure candidates sort by latest `pgscan` delta, then `memory.current`. Swap candidates sort by `memory.swap.current`. The UI may freeze up to three candidates; it does not automatically kill them.

If policy values change, update `README.md`, tests, and Debian package description in the same change.

## Whitelist

Whitelisted applications never appear in the dialog and are never frozen or force quit. Entries are desktop file IDs — the same IDs parsed from cgroup unit names (for example `google-chrome` matches `app-DDE-google\x2dchrome@....service`). One ID per line; blank lines and `#` comments are ignored.

Three levels merge as one set (vim-style):

- Package defaults: `/usr/share/deepin-liferaft/whitelist` (built from `data/whitelist`)
- Administrator: `/etc/deepin-liferaft/whitelist`
- Per-user: `~/.config/deepin-liferaft/whitelist`

The packaged defaults protect the monitor itself (`deepin-liferaft`), the screen locker (`dde-lock`), and the desktop shell (`dde-shell`). The whitelist is loaded once at startup, so restart the service after editing. `parseWhitelist`, `loadWhitelist`, and the `appProcs` whitelist filter are covered by `--self-test`.

## Resource Constraints

Keep hidden mode cheap:

- Do not construct list widgets, labels, buttons, or load desktop icons before the dialog is shown.
- Do not scan all application cgroups while pressure is low and the dialog is hidden.
- Cache desktop metadata; it is static for the lifetime of the daemon.
- Do not add a dependency for parsing or logic available through Qt, libc, procfs, or cgroup v2.

Measure changes with `/proc/PID/smaps_rollup`, not RSS alone. Report PSS and private dirty memory using equal startup timing.

## UI Style

The dialog follows Deepin's design language, not macOS alert styling. Preserve these conventions:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [st0nie/deepin-liferaft](https://github.com/st0nie/deepin-liferaft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
