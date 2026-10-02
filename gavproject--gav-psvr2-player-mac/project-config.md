---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Native 180°/360° video player for a PlayStation VR2 connected to an Apple Silicon Mac through Sony's PC adapter. macOS sees the headset as a plain 4000×2040 display (side-by-side, left half = left eye); head pose comes over USB via libusb (VID 0x054C, PID 0x0CDE). No kexts, no root, no Sony software. Swift + Metal + AVFoundation on top of a small C core.

The repo is published on GitHub: code, comments, UI strings, logs, README and new commit messages are in English (older history is Russian and stays that way).

## Commands

```sh
brew install libusb
cd player && make          # builds psvr2player and wraps it into PSVR2Player.app (ad-hoc codesigned)
cd player && make clean
player/play [video.mp4]    # launch; streams the app log to the terminal
```

- There is no Xcode project, no SwiftPM package, no tests and no linter. `make` is the only check — it must compile cleanly (C is built with `-Wall`).
- Always run through the `.app` bundle via `play`, not the bare `psvr2player` binary. Without `Info.plist` macOS can't show the removable-volume prompt and reads from external disks hang forever; `play` uses `open` so the app, not the terminal, is the "responsible" process for permissions. Rebuilding changes the ad-hoc signature, so granted permissions (Full Disk Access, Accessibility) may need to be re-added.
- The log always goes to `~/Library/Logs/PSVR2Player.log` when stdout is not a terminal (i.e. via `open`/Finder); `play` tails that file. Swift `print` and the C core's `stderr` share one descriptor. Log lines are tagged by subsystem (`[usb]`, `[display]`, …).
- Without a headset display the player falls back to a preview window on the main screen (and without USB it renders with default calibration and no tracking; proximity-sensor logic is disabled there), so the build can be smoke-tested without hardware: `player/play some_180_SBS.mp4`, then check the log. Hot-plugging the *display* is not supported (the player asks for a restart); the USB link reconnects by itself.
- Recon tools in `tools/` have no Makefile; build them one by one (binaries are gitignored):
  ```sh
  clang -O2 -I"$(brew --prefix)/include/libusb-1.0" -L"$(brew --prefix)/lib" -lusb-1.0 tools/psvr2_probe.c -o tools/psvr2_probe
  ```
  They claim USB interfaces exclusively — the player must be closed while they run.
- `swift -O tools/nebula.swift 8192 player/environment.jpg <variant>` regenerates the space environment panorama.
- `tools/tag-hvc1` (in-place 4-byte retag, reversible with `-r`) and `tools/fix-hev1` (ffmpeg remux) fix HEVC files tagged `hev1`, which AVFoundation refuses to decode (log shows `hev1 … decodable: NO`).

## Architecture

Everything lives in `player/`; all Swift files are compiled together as one module with `cpsvr2.h` as the bridging header. A new Swift or C file must be added to the `psvr2player` rule in `player/Makefile` (both the dependency list and the `swiftc` command line).

### C core (`cpsvr2.c`, `lut.c`)

`cpsvr2.c` owns the libusb session. Protocol is ported from Monado's `psvr2` driver; camera/gaze commands come from PSVR2Toolkit (keep `THIRD-PARTY.md` in sync when porting more).

- SLAM thread: bulk EP 0x83, interface 3, ~60 Hz — on-headset fused quaternion + position.
- Status thread: interrupt EP 0x88, interface 7 alt 1 — proximity sensor, Fn button, IPD and IMU batches at 2000 Hz.
- `psvr2_get_predicted_quat()` is what the renderer actually uses: last SLAM pose integrated forward through the IMU ring buffer (SLAM and IMU share the VTS timestamp scale) plus extrapolation by the requested lookahead. Its output is already in Monado-mapped axes; the raw `psvr2_get_pose()` path needs a different component remap (see `HeadTracker.currentOrientation`). Don't "unify" the two mappings.
- IMU facts (measured): exactly 2000 Hz with no drops, packets of 2 samples at 1 kHz; `vts_us` ticks once per millisecond, so both samples of a packet carry the same value (the core adds 500 µs to the second; `imu_ts_us` is the precise 16-bit clock). Gyro noise at rest ≈ 0.0013 rad/s — ~1/30 px over a 20 ms lookahead, so don't average/filter it. SLAM poses arrive 23 ms old; the IMU integration spans 35–50 ms per frame, and its start is sub-sample accurate (the first sample after the pose counts only from the pose time).
- A SLAM pose older than 0.5 s counts as invalid in all pose getters (dead stream / unplugged USB); `HeadTracker` then holds the last view and the renderer shows a notice.
- USB reconnect: a read thread that dies on a USB error sets `psvr2_link_lost()`; `AppDelegate.checkUSBLink` (1 s timer, headset-display mode only) runs `psvr2_restart()` until the device is back, then re-reads calibration. After a cable replug the device sometimes comes up with the status interface's alt setting missing (`SetAlternateInterface` → `kIOReturnNotFound`, libusb `ERROR_OTHER`); retrying never cures it, a re-enumeration does — `request_device_reset()` does that on its own thread/context because libusb blocks ~10 s inside `libusb_reset_device`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GAVProject/gav-psvr2-player-mac](https://github.com/GAVProject/gav-psvr2-player-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
