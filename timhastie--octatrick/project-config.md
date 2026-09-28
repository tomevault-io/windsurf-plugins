---
trigger: always_on
description: **Read `PLAN.md` first.** It says what octabam is now — a remixer for the
---

# Working in this repository

**Read `PLAN.md` first.** It says what octabam is now — a remixer for the
Octatrack's OS that composes modules, each credited to its author, into
one image built from the user's own 1.40C — where that
stands, what is measured about the ground, and the work order.
`docs/remixer/PLACEMENT.md` is the architecture record for where code goes.

The repo is organised as **modules** (`modules/<name>/manifest.py` declares one
contribution) composed into **remixes** (`remixes/<name>.py` selects a set).
`make modules` lists them, with the compatibility matrix; `make remix`
composes one. `docs/remixer/MODULES.md` is the contributor guide and
`CONTRIBUTING.md` the contract. The build refuses to start when two selected
modules claim the same FX2 id, cave, hook site, detour site, poke, runtime
write, core-private Y word, or the per-core FX2 buffer region — by name.

**Modules come in kinds, and the traps below say which they belong to.** A
**ColdFire module** (linked GNU-as units, detours by symbol, a runtime in
DRAM: midisc, Octakit, octalab, REPITCH) never touches the DSP and none of the DSP traps
apply to it; its own traps are in the last section. On the DSP side an
insert has no bus role, no shared-window claim, sits in both payloads and
runs on any track; a **server** pays for the rotation, the housekeeping
election, the auto-gain and the payload asymmetry, and most of the DSP
traps are a server's.

**A port is a proof.** The author's own build is the oracle: `pinned`,
`reference(addr)`, `Linked.reference` and a `Runtime` recipe's identities
are four forms of one rule, and the build refuses on drift. Never "port" by
rewriting; run their build against the shared stock image first.

**If you change the BUILD rather than a module, prove it changed nothing:**
`scripts/refhash.sh save` on a tree you trust, then `scripts/refhash.sh check`
— 26 configurations, artifacts and build reports, bit-identical. Every step
of the DRAM platform landed under that gate. (Since 10 Sep 2026 the report
prints tool paths; a path change is a report change and needs a re-save
after the artifacts are shown identical.)

**Never an Elektron byte in the repo** — no image, no slice, no `.syx`, in a
commit or on an issue. `.incbin` from the user's stock image at build time
is the pattern.

## Build and check

```bash
make modules                    # the index, the compatibility matrix, the remixes
make check REMIX=<name>         # build + cycles + every gate + boot under the port, no hardware
OT_PROJECT=<dir> [OT_BANK=2] make check REMIX=<name>   # + that project on the image under the port: ids, page-2 delivery, chain audio, main out
make bus REMIX=<name>           # THE build (XBUS=1 SPEC=1) -> out/mainos_bus.bin
make render                     # hear the bus locally, ~6x real time
make reverb IN=loop.wav ARGS='--wet --mode all'
```

Never claim something works because it assembled or linked. `make check` is
the floor.

**ALWAYS WORK IN A GIT WORKTREE, never in the main checkout.** Several
sessions share this repository at once; the main checkout's working tree,
index and stash list are theirs as much as yours. Start every task with
`git worktree add .claude/worktrees/<name> -b <branch> origin/main`, work
and run the gates there, and open the PR from it. Never `git stash` or
`git stash pop` in the main checkout: on 13 Sep 2026 a pop there took
another session's stash (`o14 port edits`) instead of the caller's own
and a `stash drop` removed it from the list -- restored by commit SHA,
but only because the dangling commits were still there. In a worktree:
`vendor/` and `.venv/` are symlinks to the main checkout's (gitignored,
and excluded in `.git/info/exclude`); `out/raw/section_3_MAIN_OS.bin`
must be there too (`make os && make recon`, or copy it); without them the
selftest reports "remix X does not build" and every gate fails before it
starts. `git submodule update --init` in the worktree as well. **Do NOT
symlink `out/emu`**: its CMake cache names the main checkout's sources,
so `cmake --build` there compiles THEIR `tools/emu/ot_emu`, not yours
(14 Sep 2026: a port edit "built" fine and the binary did not have it).
Build the port into the worktree: `make emu-cf` (a fresh cache, ~1 min).
**The same holds for `dsp_host` and `dsp_asm`:** `scripts/setup.sh` builds
them from a COPY staged into `vendor/dsp56300/source/dsp_host/`, so in a
worktree the shared binary is the main checkout's, whatever the branch's
`tools/harness/dsp_host/dsp_host.cpp` says, and `dsp_host` ignores an
option it does not know. PR #356 was reviewed twice (21–22 Sep 2026) as
"MOD has no effect, residual 0.000" for exactly this reason: its
`-paramfile` never ran, every render used default knobs, and the effect
was fine (34 gates pass under its own host). A branch that changes
`dsp_host.cpp` is built in an isolated tree, never into `vendor/`:

```
cat > /tmp/hostpr/CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.10)
project(hostpr CXX)
set(CMAKE_CXX_STANDARD 17)
add_subdirectory(/ABS/PATH/TO/main/vendor/dsp56300 dsp56300)
add_executable(dsp_host_pr /ABS/PATH/TO/worktree/tools/harness/dsp_host/dsp_host.cpp)
target_include_directories(dsp_host_pr PRIVATE /ABS/PATH/TO/main/vendor/dsp56300/source)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timhastie/octatrick](https://github.com/timhastie/octatrick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
