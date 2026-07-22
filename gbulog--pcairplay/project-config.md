---
trigger: always_on
description: **The user-facing name is AirPlayPC everywhere** (rebranded 2026-07-20):
---

# AirPlayPC (repo `pcairplay`) — context for Claude Code

## What this is

**The user-facing name is AirPlayPC everywhere** (rebranded 2026-07-20):
window title, tray, MessageBoxes, shortcuts, installer, README, launcher file
names (`AirPlayPC.vbs` / `AirPlayPC.cmd`), release artifacts. Deliberately NOT
renamed: the repo/URL (`pcairplay`), `%LOCALAPPDATA%\pcairplay` (settings and
log continuity), the firewall rule names (`PCAirPlay - …`; renaming would
strand rules on existing installs), the `gbulog.pcairplay` AppUserModelID
(pinned-taskbar identity), shared function names (`*-PCAirPlay*`), mutex
names, and `pcairplay.ico` / `pcairplay.iss` file names.

A thin Windows wrapper around **UxPlay**, an open-source AirPlay mirroring
receiver. The goal: mirror an iPhone screen to a Windows PC using **native iOS
Screen Mirroring**, with **no app installed on the iPhone**.

This repo contains no protocol code. UxPlay does the hard part (mDNS
advertisement, RTSP, the FairPlay handshake, pair-verify, H.264 over RTP). What
we add is the Windows glue that is otherwise fiddly and undocumented: install,
firewall, mDNS prerequisites, sane low-latency defaults, and diagnostics.

Machine-specific status (test-machine hostnames, LAN details, in-progress
debugging state) lives in **`CLAUDE.local.md`**, untracked and per-machine —
if it exists next to this file, read it too. Keep anything personal or
machine-identifying THERE, never here: this file is public.

## Target environment

- **Home PC**: Windows, wired Ethernet.
- **iPhone**: on Wi-Fi, same router / same subnet as the PC.
- This wired-PC / Wi-Fi-phone split is the intended setup and works, provided
  both are in one broadcast domain. See *Known pitfalls*.

## Files

| File | Purpose |
|---|---|
| `uxplay-common.ps1` | Dot-sourced by every script below. Owns engine discovery, environment setup, LAN IP, rival-receiver detection, the firewall rule names, **and the engine argv** (`Build-UxPlayArgs`). **Put shared logic here.** |
| `setup.ps1` | One-time, **needs Administrator**. Installs UxPlay **1.72.1-3** (see *The 1.x / 2.x split*), checks Bonjour, opens firewall ports. `-WhatIf` previews without elevation. |
| `start-airplay.ps1` | Starts the receiver. Normal user, no elevation. `-DryRun` prints the argv without launching. |
| `airplay-ui.ps1` | WPF desktop UI over the same engine. Owns the framed view's lifecycle (opens it with the UI, closes it on exit), persists settings, displays the PIN, single-instance. `-SelfTest` builds the window, asserts layout and the generated argv, and exits — run it after any edit here. |
| `AirPlayPC.vbs` | **Preferred double-click launcher for the UI** — zero window flashes (WScript starts the powershell console hidden from birth; a `.cmd` cannot avoid flashing its own). |
| `AirPlayPC.cmd` | Fallback launcher for the UI, for machines with Windows Script Host disabled. Flashes a console briefly. |
| `frame-mirror.ps1` | iPhone-chassis "simulator look" around the live mirror window: bezel, rounded screen corners, side buttons, shadow. Attaches to the engine's video window whenever one exists; single-instance; remembers position/zoom; has its own `-SelfTest`. Normally launched and closed **by the UI** (the "iPhone frame" switch). |
| `Framed Mirror.cmd` | Double-click launcher for the framed view standalone — only needed next to a `start-airplay.ps1` console session; the UI manages the frame itself. |
| `Diagnostics.cmd` | Double-click launcher for `doctor.ps1`, so diagnostics can't be blocked by ExecutionPolicy. |
| `doctor.ps1` | Read-only diagnostics. Run this first whenever mirroring fails. Exit code 0 = clean, 1 = problems. |
| `installer/pcairplay.iss` | Inno Setup **net-installer**: ships ONLY this repo's scripts + Start Menu shortcuts, offers to run `setup.ps1` post-install (the engine download happens there), uninstall runs `setup.ps1 -Uninstall`. Built by CI; `installer/Output/` is gitignored. See *Distribution*. |
| `.github/workflows/` | `ci.yml` — the real `setup.ps1` flow (engine download + firewall + teardown), both `-SelfTest`s, `-DryRun`, `-WhatIf` and an installer smoke-build on every push. `release.yml` — tag `v*` → build installer + scripts zip + `SHA256SUMS.txt`, publish the GitHub release. |
| `pcairplay.ico` | The app icon — window/taskbar/tray, both shortcut sets, the setup exe, Apps & Features. **Regenerate with `tools/make-icon.ps1`, never hand-edit.** |
| `tools/make-icon.ps1` | Vector-draws the icon with WPF and packs a proper multi-size .ico (256 px as a PNG entry, the rest as 32-bit BGRA BMP entries with AND masks). |
| `tools/make-installer-art.ps1` | Vector-draws the installer's wizard bitmaps (`installer/wizard-*.bmp`: dark background, white AirPlay glyph, wordmark) that the .iss references. **Regenerate, never hand-edit the BMPs.** |

`uxplay-common.ps1` is dot-sourced by all five scripts (including
`frame-mirror.ps1`, which runs it under StrictMode 3 — keep it strict-clean)
and owns the answers that used to be duplicated (and drift): `Find-UxPlay`,
`Initialize-UxPlayEnvironment`, `Get-LanIPAddress`,
`Get-CompetingReceiverProcess`, `Build-UxPlayArgs`, `Resolve-UxPlayDeviceName`,
and the UI↔frame coupling (`Test-FramedMirrorRunning`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gbulog/pcairplay](https://github.com/gbulog/pcairplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
