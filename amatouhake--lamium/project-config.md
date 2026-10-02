---
trigger: always_on
description: Lamium is a client-side quality-of-life mod for Minecraft Bedrock on
---

# Lamium agent guide

Lamium is a client-side quality-of-life mod for Minecraft Bedrock on
LeviLamina Client (Windows x64, C++20, clang-cl, xmake). This file is the
working manual for any coding agent (Claude Code, Pi, OpenCode, ...).
Read it fully before changing code.

- Product direction, UI rules and colors: [docs/DESIGN.md](docs/DESIGN.md)
- What to work on, in what order, and who should do it: [docs/BACKLOG.md](docs/BACKLOG.md)
  (release rules and pre-release checks are in it; finished items:
  [docs/BACKLOG-DONE.md](docs/BACKLOG-DONE.md))
- What works in game and what is unchecked, per feature: [docs/VALIDATION.md](docs/VALIDATION.md)
  (the evidence behind it: [docs/VALIDATION-LOG.md](docs/VALIDATION-LOG.md))
- Per-feature technical notes: `docs/*.md` (CAMERA, OVERLAYS, RESTRICTIONS,
  HAND-RESTOCK, EQUIPMENT, ...)
- Distribution/package contract: [docs/DISTRIBUTION.md](docs/DISTRIBUTION.md)
- UI mockups agreed with the maintainer: `docs/demos/` (see its README)
- Machine-specific paths (instance folder etc.): `AGENTS.local.md` if present.
  It is gitignored; never copy its contents into tracked files.

The repository is the source of truth for work: everything needed to pick up
a task is in this file and `docs/`. Start with BACKLOG.md's short
**Current execution order**, then read the selected L-item and its feature doc.
The L-item is authoritative if a summary ever drifts. If a decision made in
chat affects implementation, write it into DESIGN.md or BACKLOG.md.

BACKLOG-DONE.md and VALIDATION-LOG.md are long, append-only records. Do not
read them whole: search them for the L-number or feature you need
(`git grep -n "L-66" docs/BACKLOG-DONE.md docs/VALIDATION-LOG.md`).

## Talking to the user

- Reply in the user's language. Code, comments, commit messages and repo
  docs are English.
- The user tests in Minecraft. You cannot. Every change that touches game
  behavior ends with a short checklist: what to do in game and what they
  should see. Say which build (commit and DLL SHA-256) they are testing.
- Do not claim runtime behavior you have not seen confirmed. "Builds and
  tests pass" is not "works in game".

## Build, test, deploy

```bash
xmake f -a x64 -m release -p windows --target_type=client -y   # once
xmake build Lamium            # DLL -> bin/Lamium/Lamium.dll (+ .pdb, manifest.json)
xmake build LamiumTests && xmake run LamiumTests               # pure tests
xmake build LamiumNativeTests && xmake run LamiumNativeTests   # SDK-type tests
```

- Run `LamiumTests` after every change. It must print no failures.
- Deploy only when the user wants to test: Minecraft must be closed
  (`tasklist | grep -i Minecraft.Windows` prints nothing), otherwise the DLL
  copy fails. Copy `bin/Lamium/Lamium.dll`, `Lamium.pdb`, `manifest.json` into
  `<instance>/mods/Lamium/` and report the SHA-256 of both copies.
- Debug traces are xmake options (`xmake f --camera_trace=y` etc., see
  `xmake.lua`). Never deploy or commit with a trace option left on unless the
  user asked for a trace build. Reset with `xmake f --camera_trace=n ...`.
- Runtime log: `<instance>/mods/Lamium/logs/lamium.log`.
- Search with `git grep` or limit searches to `src tests docs`. `build/` holds
  full SDK source copies; a plain recursive grep over it takes minutes.
- SDK headers (read-only reference for game types): the xmake package cache,
  `%LOCALAPPDATA%/.xmake/packages/l/levilamina-client-sdk/26.51.5/*/include/mc/...`.
  Search there before guessing a member or function name.

### Windows shell pitfalls

- `python3` is a Store stub; use `python`.
- Write multi-line files with the file-writing tool, not bash heredocs with
  quotes/backticks inside.
- Files are UTF-8 (Japanese strings in `src/ui/Translations.h`). Tools like
  perl need `-Mutf8`/`-CSD` or they corrupt text; prefer the edit tool.

## Architecture in one page

```
src/app        Runtime (feature lifecycle, logging, settings access)
src/settings   Settings struct, option catalog (Options.h), JSON store
src/input      Actions and default keys (Binding.h), dispatch (Actions.cpp)
src/ui         Settings screen, Shapes view, widgets, layout math, translations
src/overlay    World-space geometry (pure) and rendering (WorldOverlay.cpp)
src/features   camera / information / inspection / interaction / inventory /
               lighting / visuals — one folder per area, hooks live here
tests          Pure unit tests (no game types). tests-native: SDK-type tests
```

Rules the code already follows; keep them:

1. **Pure logic in headers, game glue in .cpp.** Layout, geometry, planning
   and state machines live in `*.h` with no Minecraft types and are covered
   in `tests/`. Hooks and rendering call into them. When adding behavior,
   first ask "which part can be a pure function with a test?".
2. **No game pointers kept across frames.** Collect owned values each frame.
   Key and mouse events arrive from the window procedure, outside the client
   tick: never touch game state there. `input/CustomInput.cpp` queues actions
   and runs them on the client thread (touching inventories from the event
   crashed Minecraft).
3. **Every feature restores vanilla behavior** when disabled, on world exit,
   dimension change and focus loss.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amatouhake/Lamium](https://github.com/amatouhake/Lamium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
