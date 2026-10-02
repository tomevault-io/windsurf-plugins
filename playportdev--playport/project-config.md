---
trigger: always_on
description: An iOS app that runs x86-64 Windows games on a non-jailbroken iPhone. It is
---

# Playport

An iOS app that runs x86-64 Windows games on a non-jailbroken iPhone. It is
built on [willfaust/Madeira](https://github.com/willfaust/Madeira): Wine
(ARM64EC), FEX-Emu and DXMT as one Mach process. Everything builds, signs and
installs from one Linux workstation, with no Mac. This file covers how to
work here: build, ship, test. Reference material is in `docs/`, listed at the
end.

## Layout

| Path | What |
| --- | --- |
| `pp` | the one entry point for everything below (`./pp --help`; `pp <cmd> --help` for each command) |
| `pins.lock`, `upstream/madeira` | the upstream commits a build uses; Madeira is a submodule, and `pins.lock` madeira must equal its gitlink |
| `patches/<target>/` | every change to upstream code, as `git format-patch` files in `series` order |
| `build/` | what `pp build`, `pp install` and `pp verify` run: `pipeline`, `install`, `verify-ipa.py`, `stages/`, `toolchain/` |
| `app/` | the iOS app (a Swift package: `Sources/S1Probe` is the app, `HostIOKit`, `PlayportKit`, `SteamClient`) |
| `tools/` | `pp`'s modules: `phonelib.py` (the phone, the lock, events), `checks.py` (what `pp test` and `pp sync` both check), `ui.py`, `perf.py`, `gpu.py` (GPU debugging), `pad.py` with the scripts in `pad/`, `afc.py`, `sync.py`, `slots.py`; `tests/`; the netmuxd unit in `systemd/` |
| `docs/` | reference (below), decisions (`docs/decisions/`), evidence records (`docs/evidence/`) |
| `.work/` | the build area (`$PLAYPORT_BUILD`, gitignored): run trees, caches, IPAs, run state, and the phone's lock and install record |

## Rules

- **Commit every completed chunk of work.** After the relevant checks pass, commit
  each coherent chunk before starting the next or reporting completion; do not
  wait for a separate request. Include its documentation and changed build records,
  but never unrelated changes or private data. If a check blocks completion, report
  the blocker rather than committing the chunk as finished.
- **Upstream code is never edited in place.** Change it with a `git format-patch`
  file in `patches/<target>/` and its `series`, with `Class:`, `Evidence:` and
  `Offered-upstream:` trailers ([ARCHITECTURE.md](docs/ARCHITECTURE.md#patch-series)).
  The build checks every tree against its series by content, so an edited patch
  is reapplied on the next build.
- **Name gate.** `pp names` must stay clean (also run by `pp test` and CI). Name no project other than willfaust/Madeira as provenance.
- **Pins.** The Madeira pin moves only through `pp sync <sha>`
  ([UPSTREAM-SYNC.md](docs/UPSTREAM-SYNC.md)). The `wine`, `dxmt`, `fex` and `rpmalloc` pins
  and their `patches/*-port` series (for Wine also the `madeira-port` patches at the end of
  `patches/madeira-unix`, and `patches/wine-valve` with its `wine-valve` pin, Valve's
  Proton Wine commits picked onto WineHQ) move only by a manual rebase or pick plus a Hollow Knight
  play on the phone (decisions 0007, 0008, 0013, 0018). In such a rebase, check every auto-merged hunk against both
  sides with `git range-diff`: git misplaced several in the FEX rebase. After a DXMT
  rebase, rerun `gen_remote_guard.py` and `gen_api_names.py` in the patched tree
  (`run/dxmt-patched/dxmt/src/winemetal/`), then `pp slots` must be clean: the slot
  numbers are an ABI a merge cannot check.
- **Committed build records.** A build in `.work/run` rewrites
  `app/artifacts.tsv` and `build/generated/wine-pe-*.tsv` when its outputs change. Commit
  them with the change that caused it; any other diff there means a tree was built from
  something else. Wine PE and DXMT hash their build directory, so a build anywhere else
  (another `PLAYPORT_RUN`, a `pp sync` candidate) puts them back and leaves its own beside
  the IPA (`out/…/records/`).
- **The UI is the only entry point** (decision 0012). A person does everything in the
  app's UI, and the workstation tests it by driving that UI (`pp ui`). Add no launch
  mode, environment switch, terminal client or push-a-file tool that reaches app
  functionality another way; a capability the tooling needs goes into the UI first.
- **Two apps from one tree** (decision 0009). `dev` (the default) is `S1Probe.app`,
  which the UI driver can drive; every driver needs it installed. `release` is
  `Playport.app`: no driver, a quiet runtime, a size-capped `playport.log`. Dev-only
  Swift goes in `app/Sources/S1Probe/Dev/` or under `#if !PLAYPORT_RELEASE`;
  `pp verify --variant release` fails an executable that names a driver variable or file.
- **Swift in the app.** xtool builds it at `-Onone`; a CPU-heavy package must opt into
  `-O` itself (as `app/SteamClient/Package.swift` does). The app build loads no
  Observation macro plugin, so `@Observable` fails there: use `ObservableObject`.
- **Disk.** Everything the project makes stays in the repository: build and scratch
  work go under `.work/` (`$PLAYPORT_BUILD`, gitignored), never `/tmp` (a small
  RAM-backed tmpfs) or elsewhere in `$HOME`. System tools (llvm-mingw, xtool's darwin
  SDK and account, pymobiledevice3, netmuxd) stay installed system-wide, and nothing
  committed or built names a path outside the repository: `./pp setup` records where
  those tools are in `.work/inputs.local`; write `$PLAYPORT_BUILD`, `$LLVM_MINGW` or a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [playportdev/playport](https://github.com/playportdev/playport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
