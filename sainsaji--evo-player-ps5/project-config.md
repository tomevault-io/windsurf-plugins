---
trigger: always_on
description: A native media player for jailbroken PS5 (FW 12.70), built against the PS5
---

# EVO Player

A native media player for jailbroken PS5 (FW 12.70), built against the PS5
Payload SDK and shipped as an app module (`.ffpfsc`, title `PPSA99039`). The UI
is RmlUi on bare-metal `sceAgc`; decode is `sceVideodec2` with an FFmpeg
fallback.

This file holds the rules that are expensive to relearn on real hardware. The
full docs are in `docs/` ([docs/README.md](docs/README.md)). Read the linked
doc before working deep in an area.

**Replies:** Reply in plain, simple words: the result first, then the next step. No tables or extra detail unless I ask.

**Test on the console yourself.** After any fix, run it on the PS5 yourself: `evo-remote.sh quit` → `close` → deploy → `launch`, then drive it with `screen` / `key` / `key l3` + `tools/shot.sh grab` and read the screenshots. Never ask me to test what you can test. → [tooling.md#hardware-cycle](docs/build/tooling.md#hardware-cycle)

**Implementing a GitHub issue?** Start at [roadmap.md](docs/planning/roadmap.md).
Each issue body also has a "References & sequencing" block.

---

## Rules that must not be broken

- **Never call `make` directly.** The Makefile is missing FFmpeg's transitive
  dependencies. Use `scripts/package-app.sh` for the real build, or
  `scripts/build-evoplayer.sh` for a compile check only.
- **Everything toolchain-related runs in the pinned Docker container.**
  Scripts under `scripts/` and `tools/` re-run themselves through
  `docker compose` when started from Windows. From Git Bash, call
  `docker compose run --rm -e PS5_HOST=... ps5-dev ./tools/...` directly, and
  set `MSYS_NO_PATHCONV=1` when you pass a container path.
- **One hardware path: the `.ffpfsc` app module.** Build with `package-app.sh
  --ffpfsc`, deploy with `deploy-app.sh --ffpfsc`, then start it with
  `evo-remote.sh launch`. ShadowMount+ v1.7 does NOT auto-launch an updated
  image. Never push an EVO ELF. For layout questions use the host renderer
  (`uiview.sh`), not the console.
- **Never deploy over a running EVO, and never stack launches.** Both have
  kernel-panicked the console. Close it with `evo-remote.sh quit` (soft close:
  stop media, drain the GPU, park), then `evo-remote.sh close`. `close` is a
  kill, the same as the PS button, so the script only sends it once
  `evo_status` says `parked=1`. Never send the controllers in
  `tools/ps5-controllers/` by hand. A kill while the GPU is still submitting
  panicked the console on 2026-09-18.
- **Lifecycle truth is ShadowMount's `/data/shadowmount/debug.log`**
  (`[GAME] started` / `runtime layers released`), not an exit code. Run one
  cycle at a time, stop at the first anomaly, and never auto-retry.
- **Never sweep kernel `.text`** (`kernel_copyout` over a range). It panics the
  console every time.
- **Never call `sceVideoOutOpen` from a payload.** It returns a bogus handle
  and panics once a compute queue is allocated.
  → [hardware-decode.md](docs/hardware/hardware-decode.md)
- **Do not kill `kstuff`.** It destabilises the console into a panic.
- **The console's `/fs` web route is read-only.** Delete over FTP
  (`tools/shot.sh clean`).
- **Diagnostics: one file, `/mnt/usb0/evo.log`.** It holds the boot trace,
  breadcrumbs, decoder notes and per-file stats. klog gets the same lines live.
  `evo-remote.sh log` pulls it. `evo_status` is the live one-line state
  (`--usb-remote` builds only). A `timeout`/`curl` timeout on an
  `evo-remote.sh` call is a normal, successful outcome.
- **Framebuffer format is `0xAABBGGRR`** (BGRA in memory). Get it wrong and the
  colours swap silently; nothing crashes.

---

## Quick commands

```bash
# build + deploy (always --usb-remote for testing: enables evo-remote)
docker compose run --rm ps5-dev ./scripts/package-app.sh --ffpfsc --usb-remote
docker compose run --rm ps5-dev ./scripts/deploy-app.sh --ffpfsc   # sha256-verified

# drive the console (PS5_HOST=192.168.0.x)
./tools/evo-remote.sh launch | quit | close | status | log
./tools/evo-remote.sh screen <id> | key <button> | play <path>      # key l3 = screenshot
./tools/shot.sh grab                                                 # -> output/screenshots/latest.png
./tools/evo-remote.sh cycle --secs 60   # whole loop, classified result in output/cycles/

# host UI renderer - any layout question, no console
./tools/uiview.sh --all                 # every RmlUi screen -> output/uiview/rml_*.png
EVO_UIVIEW_IPTV_SWEEP=<m3u> ...         # walk every IPTV group with the D-pad (see the .cpp)

# serve provider bundles / a playlist TO the console - on the HOST, not the container
./tools/provider-server.sh

# compile check only, never deploys
docker compose run --rm ps5-dev ./scripts/build-evoplayer.sh
```

All scripts, screenshot measurement (`shot.sh probe/scan/crop/diff`) and env
vars: [tooling.md](docs/build/tooling.md).

---

## Repo layout

```
projects/evoplayer/
  main.cpp       entry point only: crash/SIGTERM handlers -> Application::run()
  main.c.legacy  the old monolith. NOT COMPILED. Don't edit it or read it to
                 learn current behaviour; grep core/ instead.
  core/          the player: Application.cpp (frame loop, soft close), screens/,
                 services/ (PlaybackController, settings, metadata)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sainsaji/EVO-PLAYER-PS5](https://github.com/sainsaji/EVO-PLAYER-PS5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
