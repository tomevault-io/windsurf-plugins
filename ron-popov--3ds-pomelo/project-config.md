---
trigger: always_on
description: To check that the project compiles, run `make 3dsx` from the repo root
---

# Pomelo — APT internals notes

## Build

To check that the project compiles, run `make 3dsx` from the repo root
(requires `DEVKITARM` to be set).

This file documents what has been verified (via Ghidra reverse-engineering of a
real Nintendo `ns` system module, cross-checked against a real-hardware IPC
packet capture of the stock Home Menu, and empirical on-device/Mikage
debugging) about how the 3DS APT service actually behaves. Pomelo runs as the
Home Menu (`APT:A`/host role), which exercises code paths regular guest apps
never touch, so `libctru`'s stock assumptions don't fully apply here — this is
why pomelo ships its own libctru fork
([libctru-for-homemenu](https://github.com/ron-popov/libctru-for-homemenu),
vendored as the `libctru` submodule).

For the same reason pomelo also ships a **citro3d fork**, vendored as the
`citro3d` submodule (see the 28-bit offset ceiling section below). Both forks
are wired up through `LIBDIRS` in the Makefile, and **their entries must stay
ahead of `$(CTRULIB)`** — devkitPro's libctru directory bundles its own
`libcitro3d.a` and `c3d/` headers, `LIBDIRS` is expanded into `-I`/`-L` flags in
order, and first match wins. Get the order wrong and the build silently links
stock citro3d, reintroducing the GPU hang below with no visible error.

Mikage's APT HLE is treated as behaviorally equivalent to real hardware for
this project — bugs reproduced under Mikage are assumed to reflect real APT
semantics, not emulator-specific divergence, unless proven otherwise.

## NS-side APT command IDs (verified against a real-hardware IPC capture)

Confirmed by decoding real Home Menu IPC headers (`cmd_id = header>>16`,
`normal_params = (header>>6)&0x3F`, `translate_params = header&0x3F`) and
matching them against the `online_ns_module` binary's `handle_ns_ipc` switch:

| cmd  | Name                          |
|------|-------------------------------|
| 0x02 | Initialize                    |
| 0x0E | GlanceParameter               |
| 0x15 | PrepareToStartApplication     |
| 0x1A | GetSharedFont                 |
| 0x1B | StartApplication               |
| 0x1C | WakeupApplication              |
| 0x43 | NotifyToWait                   |
| 0x4B | AppletUtility                  |

**Do not trust `0x1a == WakeupApplication`** — this was an initial
misattribution (based on imperfect memory of 3dbrew's command table) that got
corrected mid-session. `0x1a` is actually `GetSharedFont`; the real
`WakeupApplication` is `0x1c`.

## The transition-lock bitmask (`0x001211aa` in `online_ns_module`)

A 16-bit global flags word in NS is the actual mechanism behind the
`0xe0a0cc08` ("Invalid State") error from `APT_PrepareToStartApplication` /
`APT_WakeupApplication`. Each "role" (Application, various system applets)
claims a distinct bit. Five primitives operate on it:

- **`FUN_0010d4ac(bit)`** — bit-test: `flags==0 || (flags&bit)!=0`.
- **`FUN_0010e3ec(bit1, bit2)`** — **non-blocking conditional acquire**. Fails
  immediately (returns 0, and callers surface `0xe0a0cc08`) if
  `flags!=0 && (flags&bit1)==0 && (flags&bit2)==0`. This is what
  `APT_PrepareToStartApplication`, the real `APT_WakeupApplication` (both use
  bit `0x10`), and `APT_TryLockTransition` (`AppletUtility` id 6) all use.
  **There is no retry/wait built into this path** — if something else holds
  the lock at that exact instant, the call fails outright.
- **`FUN_0010a288(bit)`** — **blocking** acquire: loops, sleeping and
  retrying, until it can claim the bit. Used by `APT_LockTransition` when its
  `flag` argument is `false`. Distinct from the non-blocking primitive above.
- **`FUN_0010e468(bit)`** — unconditional exchange (steals the lock
  immediately, no waiting). Used by `APT_LockTransition` when `flag` is
  `true`, and internally by `PrepareToStartApplication`'s fast-path.
- **`FUN_0010f2c4(bit)`** — behind `APT_UnlockTransition(transition)`.
  **Only actually clears the flags word if `bit==0` (unconditional force-clear)
  or the currently-held flags already overlap `bit`.** Otherwise it's a
  **silent no-op that still returns success** — this is the crux of the whole
  double-launch bug (see below).

`AppletUtility`'s `id` dispatch (from decompiling `FUN_0010ee9c`):
`id=4` → SleepIfShellClosed (reads the flags word via `FUN_0010e4a0`, branches
on whether it's zero), `id=5` → LockTransition, `id=6` → TryLockTransition,
`id=7` → UnlockTransition. The function's switch always falls through to
`return 0;` — i.e. **every `AppletUtility` call, real result aside, replies
with the same fixed success framing at the NS/IPC-response level**; the
actual outcome must be inferred from the effect (e.g. via a follow-up
`TryLockTransition` probe), not assumed from the fact that the call
"succeeded".

Other confirmed globals: `g_abAPTState` (0x00120580, 96 bytes — per-session
state: `+0x0f` FSM state, `+0x1b` NotifyToWait flag, `+0x40` self appID,
`+0x44` NotifyToWait target appID, `+0x48` lock/arbiter handle),
`g_abAPTStateLock` (0x001205e0 — a `svcArbitrateAddress`-based mutex, *not* a
second state struct — this was an initial, disproven hypothesis),
`g_abAppletSlots` (0x001217f0, 160 bytes = 5×32 — per-category
Application/HomeMenu/etc. applet slot array).

## Real bugs found and fixed in `libctru-for-homemenu`'s `apt.c`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ron-popov/3ds-Pomelo](https://github.com/ron-popov/3ds-Pomelo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
