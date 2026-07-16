---
trigger: always_on
description: Building a modern, feature-complete open-source PlayStation 3 SDK that supports **C++17 on both PPU (PowerPC 64) and SPU (Cell SPE)**. Replaces the stale ps3dev baseline (GCC 7.2.0 / binutils 2.22 / newlib 1.20.0) with current versions, fills PSL1GHT API gaps, ships fragment shader support, and uses the proprietary reference proprietary reference SDK privately as a coverage oracle (never committed, never shipped — see `reference/REFERENCE_POLICY.md`).
---

# PS3 Custom Toolchain — Project Context (auto-loaded)

## What this is

Building a modern, feature-complete open-source PlayStation 3 SDK that supports **C++17 on both PPU (PowerPC 64) and SPU (Cell SPE)**. Replaces the stale ps3dev baseline (GCC 7.2.0 / binutils 2.22 / newlib 1.20.0) with current versions, fills PSL1GHT API gaps, ships fragment shader support, and uses the proprietary reference proprietary reference SDK privately as a coverage oracle (never committed, never shipped — see `reference/REFERENCE_POLICY.md`).

## Authoritative plan

`~/.Codex/plans/i-m-going-to-create-melodic-dawn.md` (per-host). Also mirrored at `docs/plan.md` in this repo for reference.

## Status snapshot

- **Phase 0 (bootstrap):** complete on Windows side. ~30 authored files: scripts, Rust tooling, samples, docs.
- **Phase 1 (PPU toolchain):** complete — binutils 2.42, GCC 12.4.0, newlib 4.4.0, GDB 14.2 all built on CachyOS.
- **Phase 2a (SPU toolchain):** complete — spu-elf-gcc 9.5.0 + newlib 4.4, C/C++17 smoke tests pass.
- **Phase 3 (PSL1GHT v3 RFC):** builds green 2026-04-17; RFC / NV40-FP / NID tooling still in progress.
- **libspurs surface (2026-04-24):** full `<cell/spurs.h>` PPU surface (Spurs2, Taskset/Taskset2, event-flag / queue / semaphore / barrier / task / workload) + SPU task runtime (`libspurs_task.a` + `spurs_task.ld` + `cell/spurs/spu_task.h`).  `hello-spurs-task` runs end-to-end in RPCS3 (PPU spawns embedded SPU task, SPU task DMAs flag back to PPU, Taskset Shutdown+Join).  SPU ABI: crt at LS 0x3000 (byte-identical to reference `libspurs.a` main.o), dispatch via LS 0x2fb0 control block + `dispatch_base+0xc4`, exit code at LS 0x2fd0.
- See `docs/phase-0-status.md` for full inventory.
- See `docs/HANDOFF.md` for session resume notes.

## Locked-in decisions

| Decision | Choice |
|---|---|
| Build host | Native CachyOS (Arch-based). MSYS2 was the Windows plan; abandoned because user is migrating to native Linux for build perf and toolchain stability. |
| PPU compiler | GCC 12.4.0 (`powerpc64-ps3-elf`) |
| SPU compiler | GCC 9.5.0 (`spu-elf`) — last with intact SPU backend; binutils 2.42 spu-elf still upstream |
| Long-lead | Phase 2b: forward-port SPU backend to GCC 12+ (separate worktree, parallel) |
| binutils | 2.42 (both PPU and SPU). **No patches needed** — modern binutils handles both targets via existing catchalls (verified). |
| newlib | 4.4.0.20231231 with PS3 libsysbase glue ported forward |
| GDB | 14.2 (PPU only; SPU combined-debug orphaned post-2019) |
| PSL1GHT | v3 RFC: Cell-style naming (`cellXxx`, `CELL_XXX_*`, `CellXxx`); ships `psl1ght-compat.h` shim for legacy homebrew |
| Fragment shader | NV40-FP assembler first; full GLSL/Cg compiler later |
| NID/stub tooling | Rust (`tools/nidgen`, `tools/coverage-report`) |
| reference SDK mount | Deferred — user provides path later. Phase 3 coverage tooling falls back to PSL1GHT-only mode until then. |
| Eventual host targets | Linux primary for development. **Windows-hosted toolchain binaries needed eventually** — Canadian-cross from Linux to `x86_64-w64-mingw32` to produce `powerpc64-ps3-elf-gcc.exe`. Phase 5 infra concern. |

## Workspace layout

```
PS3_Custom_Toolchain/
├── scripts/                     bootstrap.sh, env.sh, build-{ppu,spu,psl1ght,portlibs}-toolchain.sh
├── src/                         .gitignored — vendored upstream + ps3dev + forks (re-clone via bootstrap.sh)
├── patches/                     ppu/{binutils-2.42,gcc-12.x,newlib-4.x,gdb-14.x}/ + spu/* + psl1ght/ + portlibs/
├── tools/                       Rust workspace: nidgen (FNID + stubgen), coverage-report
├── reference/private/          .gitignored read-only mount; never commit
├── samples/                     hello-ppu-c++17, hello-spu (build after Phase 1+2)
├── docs/                        plan.md, phase-0-status.md, HANDOFF.md, patches-inventory.md
├── stage/ps3dev/                .gitignored — $PS3DEV install prefix (bin/, ppu/, spu/, psl1ght/, portlibs/)
├── build/                       .gitignored — intermediates (real builds use $PS3_BUILD_ROOT for short paths)
├── ci/, cmake/, tests/          scaffolded; populated in later phases
└── reference/REFERENCE_POLICY.md hard rules for the private mount
```

## Build orchestration

```bash
source ./scripts/env.sh
./scripts/bootstrap.sh                          # clones upstream + ps3dev + forks
./scripts/build-ppu-toolchain.sh --only binutils # Phase 1a — start here
./scripts/build-ppu-toolchain.sh --only gcc-newlib   # Phase 1b/c (after patches rebased)
./scripts/build-spu-toolchain.sh                # Phase 2a
./scripts/build-psl1ght.sh                      # Phase 3
./scripts/build-portlibs.sh                     # Phase 4
```

Both build scripts accept `--only <step>` (binutils | gcc-newlib | gdb | symlinks) and `--clean`.

## Patches inventory (what needs rebasing)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FirebirdTA01/PS3DK](https://github.com/FirebirdTA01/PS3DK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
