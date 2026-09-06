---
trigger: always_on
description: **This is 30-year-old hardware and libraries. Your training data is NOT sufficient.**
---

# libSDL2-amigaos3 -- Claude Code Project Instructions

## RULE 0: NEVER GUESS -- ALWAYS LOOK IT UP

**This is 30-year-old hardware and libraries. Your training data is NOT sufficient.**

Before writing or fixing ANY code that touches AmigaOS APIs, Exec, devices, or 68k hardware:

1. **Read the documentation first.** Use `amiga_api_lookup`, `amiga_search`, `amiga_pitfalls_for`, or read the ADCD docs in `docs/references/`. Every function, every struct field, every flag has documented behavior that may differ from what you expect.

2. **Search the internet.** Use `WebSearch` for forum posts, existing implementations, and known issues. Check EAB (eab.abime.net), Aminet, and GitHub for reference code.

3. **Never assume how an API works.** CheckIO, WaitIO, BeginIO, SendIO, CreateMsgPort, Signal -- these all have specific documented semantics. A wrong assumption about `ln_Type`, signal delivery, or task context wastes hours.

4. **When debugging, instrument and observe before hypothesizing.** Add debug output (DLOG via `SDL_os3debug.h`), read the actual values, THEN form a theory. Don't guess at the fix -- verify the root cause first.

5. **When stuck after 2 attempts, STOP coding and research.** Read ADCD docs, search forums, check reference implementations (SDL 1.2, OS4 port). The answer is almost always documented somewhere.

**Why this rule exists:** Multiple debugging sessions wasted hours because the agent guessed at AmigaOS behavior instead of reading the documentation. Examples: MsgPort signal delivery (ADCD ch.22), CheckIO return semantics, audio.device CMD_WRITE completion, CreateNewProcTags tc_UserData race. Every one of these is documented in the ADCD.

## What This Project Is

A port of SDL2 (Simple DirectMedia Layer 2) to AmigaOS 3.x on Motorola 68k. This is a **platform library** -- it provides `libSDL2.a` that other programs link against. It is NOT a POSIX port; it implements SDL2's platform abstraction layer using native AmigaOS APIs.

## Architecture

SDL2 has a backend/driver architecture. Each subsystem (video, audio, threading, etc.) has a platform-specific implementation. We provide AmigaOS 3.x backends:

| Subsystem | Backend | AmigaOS API | Source Dir |
|-----------|---------|------------|-----------|
| Video | CyberGraphX / Picasso96 | `WritePixelArray()`, screen modes | `src/video/amigaos3/` |
| Video (AGA) | Custom chipset + c2p | Blitter, bitplanes | `src/video/amigaos3/` (Phase 6) |
| Audio | AHI | `AHI_AllocAudio()`, callbacks | `src/audio/amigaos3/` |
| Threading | Exec Tasks | `CreateNewProc()`, `SignalSemaphore` | `src/thread/amigaos3/` |
| Timer | timer.device | `ReadEClock()` | `src/timer/amigaos3/` |
| Input/Events | Intuition IDCMP | `IDCMP_RAWKEY`, `IDCMP_MOUSEMOVE` | `src/events/` (via video driver) |
| Joystick | gameport.device | `GPD_ASKCTYPE` | `src/joystick/amigaos3/` |
| Filesystem | dos.library | `Lock()`, `Examine()` | `src/filesystem/amigaos3/` |
| Loadso | Stub (no dlopen) | -- | `src/loadso/dummy/` |
| Haptic | Stub | -- | `src/haptic/dummy/` |
| Render | Software (built-in) | -- | `src/render/software/` |

## Codebase Map

```
include/SDL2/              # Standard SDL2 public headers (from upstream)
src/
  video/amigaos3/          # CyberGraphX/P96 + optional AGA + optional AMMX
  audio/amigaos3/          # AHI audio
  thread/amigaos3/         # Exec Tasks
  timer/amigaos3/          # timer.device
  joystick/amigaos3/       # gameport.device
  filesystem/amigaos3/     # dos.library
  loadso/dummy/            # Stub (no dlopen on OS3)
  haptic/dummy/            # Stub (no haptic)
  main/amigaos3/           # SDL_main entry point
  render/software/         # SDL2 built-in software renderer
  events/                  # IDCMP -> SDL event translation
examples/                  # Test programs (testsprite2, loopwave, testkeys, etc.)
docs/
  references/              # Amiga hardware and API reference docs
  adr/                     # Architecture Decision Records
Makefile                   # Cross-compile to libSDL2.a via bebbo-gcc
```

## Build Instructions

```bash
make setup-toolchain   # Pull/verify bebbo-gcc Docker image
make                   # Build libSDL2.a (cross-compile via Docker)
make examples          # Build example/test programs
make test              # Run tests via FS-UAE with RTG
make clean             # Remove build artifacts
```

**Prerequisites:** Docker, Python + amitools, FS-UAE with RTG-capable config.

## Compiler Settings

- **Language:** C99 (`-std=gnu99`). SDL2 requires C99.
- **CPU target:** `-m68020` for the library (RTG cards require 68020+).
- **Optimization:** `-O0` initially until backends are proven stable. Upgrade to `-O2` per-file after testing. bebbo-gcc has codegen bugs at `-O1`/`-O2` (see crash pattern #16 via `amiga_crash_diagnosis`).
- **`SDL_DYNAMIC_API`:** Disabled (`#define SDL_DYNAMIC_API 0`). AmigaOS 3.x has no `dlopen()`.

## Coding Standards

### C99 with AmigaOS Constraints

- Use C99 features (for-init, `//` comments, mixed declarations, `inline`).
- Do NOT assume C99 library functions exist -- libnix is a C89 runtime. Check via `amiga_search "libnix [function]"` or `amiga_pitfalls_for "libnix"`.
- Use `<proto/*.h>` for Amiga system calls (never `<clib/*.h>`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bdgscotland/libSDL2-amigaos3](https://github.com/bdgscotland/libSDL2-amigaos3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
