---
trigger: always_on
description: Xbox NTSC-U Need for Speed: Underground 2, lifted to C with xboxrecomp and
---

# NFSU2 (Xbox) static recompilation — notes for Claude

Xbox NTSC-U Need for Speed: Underground 2, lifted to C with xboxrecomp and
built for Linux and Nintendo Switch (libnx NRO). This repo is
https://github.com/antoxa2584x/nfsu2-sw (`main`, commits as
`Anton Artemov <antoxa2584@gmail.com>`). On 2026-09-29 it replaced the old PS2
port there (history backed up in `/root/nfsu2x/nfsu2-sw-ps2-backup.bundle`).
The toolkit is vendored in `xboxrecomp/`; the default for `XBOXRECOMP_DIR`.

## Rules

- Never commit game data (disc, `default.xbe`, `switch_sd/`) or generated C
  (`gen/`). Ask before committing or pushing anything.
- The Switch build reads the **unpacked** disc at `sdmc:/switch/nfsu2x/game/`,
  never the ISO.
- Don't launch or kill Eden unless the user asked for an Eden check; check
  `tasklist.exe | grep -i eden` first. Eden's `sdmc\switch` is a junction to
  `nfsu2-xbox\switch_sd\switch`, so an Eden run overwrites the log
  there — back up a hardware log before running Eden. Use
  `run_eden.sh` (never a bare `taskkill /F`): it closes Eden politely,
  force-kills only after 15 s, and restores eden.exe/qt-config.ini from
  `/root/nfsu2x/eden_backup/` if a hard kill damaged them. `NRO=x.nro` runs
  another NRO from `switch/nfsu2x/`. The user drops real
  console logs in `switch_sd/switch/logs/`.
- One test at a time on Linux (runs share `fb/`, `gfb/`).
- Never `pkill -f` a pattern that also matches your own command line (it
  kills the shell, and `run_eden.sh` then force-closes Eden).
- Clean up every Linux test run. `pkill -x nfsu2_recomp` does NOT match (the
  process renames itself), and `timeout`/`xvfb-run` leave orphans that keep
  burning CPU; kill by PID:
  `ps -eo pid,args | awk '$2 ~ /nfsu2_recomp$/ || $2 ~ /^Xvfb$/ {print $1}' | xargs -r kill`

## Layout and builds

| Where | What |
|---|---|
| `/root/nfsu2x/` (WSL) | `game/` extracted disc, `gen/` lifted C, `build-pr128/` Linux, `build-switch/` Switch, `dis.py ADDR [+N]`, `snap.sh SECS ENV=..` (gdb stacks, `BIN=`), `run_eden.sh SECS` |
| `xboxrecomp/` | vendored toolkit = xboxrecomp main + PR #128 + all port changes (copied from the `/root/nfsu2x/xboxrecomp-pr128` worktree; keep the two in sync) |
| `src/main.c` | boot; defaults RECOMP_VBLANK=1, RECOMP_AC97_READY=plain, RECOMP_USB=1, RECOMP_PB_EXEC=1; APU at 0xFE800000 via `xbox_MmioRegister`; Switch runs the game on a 16 MB pthread |
| `src/recomp_manual.c` | memmove ×2 (0x2A7EE0, 0x2A9450), AC97 reset 0x33518D, DSP ack wrapper 0x32EB65, D3D fence wrapper 0x2E8F20 |
| `src/switch_nx.c` | log, `nfsu2x_env.txt`, exception handler, t= stamps |

- Regenerate C: `tools/regen.sh` (`LIFT_ONLY=1` after manual-override edits;
  full run after seed changes). Passes `--mmio-sections DSOUND,XPP`.
- Switch: `XBOXRECOMP_DIR=/root/nfsu2x/xboxrecomp-pr128 bash switch/build.sh`
  (without `XBOXRECOMP_DIR` it picks the main checkout and fails on OpenSSL).
  The copy step fails with "Permission denied" while Eden has the NRO open.
- Linux: `cmake --build /root/nfsu2x/build-pr128 -j8`; run with
  `NFSU2_GAME_DIR=/root/nfsu2x/game xvfb-run -a …/nfsu2_recomp`.
- Menu pad script (Linux): `RECOMP_PAD_SCRIPT="10000:start:300,…,70000:start:300,80000:a:200,90000:a:200"`
  (times from the first pad read). `RECOMP_GL_DUMP=<prefix>,N` dumps frames.
- Header changes in `templates/runtime/recomp_types.h` must be copied to
  `gen/recomp_types.h` (regen does it) and rebuild all generated code.

## Switch (Horizon) findings

- **Memory:** `svcCreateSharedMemory` → 0x4201 in Eden and may kill the
  process on hardware; `svcMapPhysicalMemory` → 0xFA01. Guest RAM uses code
  memory (`svcCreateCodeMemory` + `svcControlCodeMemory(MapOwner)`) — the
  default; `RECOMP_NX_SHM=1` opts into shared memory. Code memory cannot
  alias, and the title needs aliasing: it reaches RAM at 0x0 and at the
  0x80000000 contiguous window, and on hardware read a stale 0xAAAAAAAA
  pointer through 0x80xxxxxx after Start (crash, exception 257). Second
  views of a mapping now try `svcMapProcessMemory` on our own process
  (log: `[NX] aliasing views with svcMapProcessMemory` or `... refused
  (0xRC)`). If that is refused, the fallback is folding 0x80000000–0x83FFFFFF
  onto low RAM in `XBOX_PTR` *and* the runtime's translations.
- **Logging / boot time:** the SD log was the boot bottleneck — the runtime
  `fflush(stderr)`s after many lines and each flush was an SD write (console:
  ~33 s to display mode; Linux 0.3 s; Eden 5 s). Now stdout+stderr go to a
  `log:` devoptab (switch_nx.c) that appends to RAM with a `[  t.ttt]`
  timestamp per line; a thread writes it to the card every 0.5 s (crash
  handler drains it). Eden boot to display mode dropped 5 s → 2 s.
- **Loading screen:** NFSU2 logo (`assets/nfsu2_logo.png`, SteamGridDB →
  `tools/make_logo.py` → `src/nfsu2_logo.h`, RLE) + a moving bar, drawn on the
  SDL window/GL context that the renderer then adopts
  (`nv2a_gl_adopt_window`), and kept on presents until the title's first real
  draw (`nv2a_gl_draw_placeholder`, `s_drew_any`). A libnx framebuffer can't be
  used: `SDL_CreateWindow` hangs forever after one was open. `NFSU2_LOADER=0`
  disables it.
- **Unaligned atomics:** x86 `lock cmpxchg`/`xadd` on unaligned addresses are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoxa2584x/nfsu2-sw](https://github.com/antoxa2584x/nfsu2-sw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
