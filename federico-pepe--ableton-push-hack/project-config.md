---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Doc sync rule:** Keep this file, all `docs/` files, and `README.md` in sync with every code change. If a change affects behaviour, APIs, architecture, or known issues — update the relevant docs in the same commit.

## Project

`push-hack` — extensible hack framework for Ableton Push 3 (Intel Linux, runs full Ableton Live). Deploys via SSH. Never modifies system partition. Hacks: Push Manager (web file browser + display control), Push Display (LD_PRELOAD display hook), Browser Bridge (Live MIDI Remote Script to load `.adv`/`.adg` presets — **one-time manual activation required**), Automation (LFO/CC curve sequencer, port 7703).

**Core constraint:** Push is a live performance tool. Hacks must not crash it, hog CPU, or consume significant memory.

**⛔ Hard safety rules — never violate, no exceptions:**
- **Never modify `/boot/`** — bricks Push.
- **Never modify `/opt/`** — read-only system partition. Contains Push3 app, firmware, assets.
- **Never modify kernel parameters** — no `sysctl -w`, no `/proc/sys/` writes affecting stability.
- **Never write to `/etc/`** except: `/etc/udev/rules.d/99-push-hack-*.rules`, `/etc/init.d/push-hack-*`, and the LD_PRELOAD line in `/etc/init.d/push3` (all managed by install/uninstall scripts).

## Commands

### Build
```bash
cd hacks/push-manager && PATH=$PATH:/usr/local/go/bin make
cd hacks/automation && PATH=$PATH:/usr/local/go/bin make
cd hacks/push-display && make          # cross-compiles push_hook.so via Docker
```

### Deploy
```bash
./scripts/install.sh                              # deploy all enabled hacks (pre-built)
./scripts/install.sh --hack push-manager --build  # build from source then deploy
./scripts/uninstall.sh                            # remove all hacks + services
./scripts/uninstall.sh --purge                    # also delete /data/push-hack/ data
hacks/push-display/deploy.sh                      # standalone push-display re-deploy
```

### Discovery
```bash
./scripts/discover.sh                    # probe Push OS, print filesystem map
```

## Architecture

### Framework layer (`scripts/`, `lib/`)
SSH-based deploy system. `lib/common.sh` — shared SSH helpers (`push_exec`, `push_exec_root` use `-n` to prevent stdin consumption in loops), Push path detection, service install/remove. Push uses **sysvinit**, not systemd. Stop service before SCP — running binary is locked on Linux. Regular binaries copied as `ableton`; `.so` files copied as `root` via `push_copy_root`. `check_connection()` auto-clears a stale SSH host key (`clear_host_key()` → `ssh-keygen -R`) when an OS update regenerated the device key — detects `REMOTE HOST IDENTIFICATION HAS CHANGED` and retries.

### Hack structure (`hacks/<hack-id>/`)
- `hack.json` — metadata: id, name, version, port, allowed_roots, binary, enabled
- `service.initd` — optional custom init.d template; placeholders: `{{SVC_NAME}}`, `{{HACK_DIR}}`, `{{LOG_DIR}}`, `{{PORT}}`
- `remote-script/` — optional payload copied to `<remote_hack_dir>/remote-script` by install.sh
- Binary deployed to `/data/push-hack/hacks/<id>/`; service at `/etc/init.d/push-hack-<id>`

### Push Manager (`hacks/push-manager/`)
Go binary, no runtime deps. ~8–15MB RSS. Port 7701. See `hacks/push-manager/README.md` for full API.

| File | Role |
|------|------|
| `src/main.go` | HTTP server, routes, middleware |
| `src/files.go` | Filesystem ops with path traversal guard |
| `src/stats.go` | CPU/memory/disk/uptime/IP stats; top processes (Ableton Index, Live, Push3, push-manager) |
| `src/presets.go` | Preset index: scans `.adv`/`.adg` under Core Library, Factory Packs, User Library. In-memory cache + `presets.json`. `QueryPresets(PresetFilter)`, `presetFacets()`. Metadata (favourites, tags) in `preset_meta.json`. |
| `src/live_bridge.go` | One-shot TCP to `127.0.0.1:7704` (Browser Bridge). `liveLoad(name, category)` → `load:<root>:<name>`. Also: `livePlay()`, `liveStop()`, `liveIsPlaying()`, `liveTempo()`, `liveBeat()`. |
| `src/display.go` | Shared-memory bridge to push_hook.so. Mmaps `framebuf`. Three modes: 0=passthrough, 1=bar, 2=takeover. OSD subsystem: single-line and multi-line renderers. Startup splash on fresh hook attach. Screenshot: `shmReadFrame`+`bgr565ToImage` read the framebuf back and `png.Encode` it — captures only push-manager-owned frames (Shadow UI/OSD/image/testpattern), not the native Ableton UI (never copied into shm in passthrough). |
| `src/midi.go` | ALSA seq subscriber + LED output — pure Go ioctls (no cgo). **Boot-settle:** defers `/dev/snd` access until uptime ≥ 30s (USB-A safety). **Auto-detect:** `detectPush3Port()` scans `/proc/asound/seq/clients` by name on each connection attempt — handles shifted client numbers (e.g. 20 instead of 16) when USB MIDI devices are connected at boot; disabled once user manually subscribes. LED config system (trigger/momentary/exclusive modes, animations). Chords: Shift+Settings=intercept toggle, Shift+Set=open browser. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [federico-pepe/ableton-push-hack](https://github.com/federico-pepe/ableton-push-hack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
