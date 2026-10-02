---
trigger: always_on
description: Entry point for **any** agent working in this repo — Claude Code, Codex,
---

# Agent Operating Guide

Entry point for **any** agent working in this repo — Claude Code, Codex,
Antigravity, or a future API/LiteLLM loop. This file is intentionally tool-
agnostic. Coordination happens through files and a small CLI, never through any
one tool's private memory.

## Resuming ("continue")

If told only to "continue", do this: if `progress/ledger.sqlite` is missing (fresh
clone), run `work bootstrap` once — it inits submodules and rebuilds the ledger from
the committed `progress/status.json` + `progress/tu_deps.json`, restoring exactly where
the last commit left off. Then `work claim` → pick up the next ready TU. No other context
is needed. (If the maintainer gave you a coordination-server URL, set it up first — see
"Coordination server" below; otherwise you work locally, no setup needed.)

**Continuing the verification sweep** (a distinct task from "continue" reconstruction):
if asked to continue, resume, or run the **verify sweep** — the correctness audit that
re-verifies every already-`done` TU against the X360 asm and fixes divergences — read
[`progress/sweep/VERIFY_SWEEP_HANDOFF.md`](progress/sweep/VERIFY_SWEEP_HANDOFF.md) first. It is the
self-contained operating guide for that pass; its state/queue lives in
[`progress/verify_sweep.json`](progress/verify_sweep.json) (per-TU `state`:
`pending`/`pass`/`fixed`/`flagged`/`conductor_fix`/`not_reconstructed`).

### Environment Checklist (Verify Before Reconstructing)

Before compiling code or exporting functions, verify these settings:
1. **Visual Studio / MSVC Path:** MSVC is auto-located by [`tools/build/msvc_env.bat`](tools/build/msvc_env.bat) (live `cl` 19.x, then default VS2022 installs, then `vswhere`). Only set the `VCVARS64` environment variable if VS2022 lives somewhere non-standard. If no MSVC is found, the compile gate reports `skip`, meaning errors won't be caught. Compile flags/includes are the canonical `tools/build/msvc_flags.txt` + `msvc_includes.txt` — the same set the shipping exe build uses.
2. **IDA Pro Path:** If you need to generate stubs/skeletons for new functions or run the parallel exporter, make sure `idat.exe` is available. You can pass the path explicitly via the `-IdaPath` parameter to `tools/export_db.ps1`, or set the `IDA_PATH` environment variable.
3. **Submodules:** The `b5-decomp` EA vendor submodules must be initialized. `work bootstrap` does this, but you can verify them under `b5-decomp/vendor/`.
4. **Coordination config (only if invited):** If the maintainer gave you a server URL, `cp .env.example .env`, uncomment `WORK_SERVER`, set it to that URL, and set a unique `WORK_AGENT`. With no URL, skip this entirely — you work locally. See "Coordination server" below.
5. **Building the exe / game data:** follow [BUILD.md](BUILD.md). Machine paths live in `build.config.toml` (copy from `build.config.example.toml`); `build doctor` verifies the whole toolchain and `build all` sequences tools → lua → ffmpeg → exe → data.

## Read first, in order

1. [`README.md`](README.md) — what this repo is (orchestration, not the decomp).
2. [`STRATEGY.md`](STRATEGY.md) — the plan, the build roles, the identity model,
   the stub scaffold, and what "done" means. **Do not start work without it.**
3. The ledger under [`progress/`](progress/) — current state of every TU/function.

## What you are doing

Reconstructing the **X360 build** as compilable **PC C++**, one translation unit
at a time, landing recovered code in [`b5-decomp/src`](b5-decomp/src/). Target is
**semantic parity, not byte-matching**. A unit is done when: reconstructed → the
TU compiles → a reviewer pass approves.

## The work loop

```
work claim <tu>...    # claim specific TU id(s) — when you want a particular one
work claim [-n N]     # ...or, with no id, claim the next N ready TUs from the queue.
                      #   With a coordination server (invite-only, see below) every claim
                      #   is atomic across everyone; without one it claims locally.
work next             # read-only PREVIEW of the queue (reserves nothing)
work show <tu>        # concise overview (functions, signatures, dependency TUs, console audit)
work audit <tu>       # the per-commit evidence audit for the TU: what the console's functions do
                      #   that our bodies don't (case ids, event posts, callees, asserts, bodies) + stubs
                      #   + the instruction shape of the BUILT exe vs the console (tier A/B/C per function)
work show <tu> --full # the full dossier: pseudocode, locals, DecFIGS dwarfdump
                      #   hints, Feb-2007 original source, callee signatures
                      #   (--asm for disasm, -o to a file)
work start <tu>       # claim one specific TU by id (todo -> in_progress) — use when you
                      #   already know which TU you want; `work claim` is the normal path
work stubs <tu>       # trap-stub the callees this TU needs that aren't done yet
                      #   (--list shows what must be declared — the part that matters
                      #   under the compile-only gate; defs are for the future link)
  …reconstruct the C++ into b5-decomp/src/<mirrored path>…
work submit <tu>      # run the compile gate; on pass, run the parity check + emit a reviewer packet

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BurnoutDecomp/BP-Decomp_Workflow](https://github.com/BurnoutDecomp/BP-Decomp_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
