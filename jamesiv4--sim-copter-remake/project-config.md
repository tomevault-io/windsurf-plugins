---
trigger: always_on
description: Instructions for AI agents working on **sim-copter-remake** — a from-scratch Unreal Engine 5.8
---

﻿# AGENTS.md

Instructions for AI agents working on **sim-copter-remake** — a from-scratch Unreal Engine 5.8
re-implementation of Maxis's *SimCopter* (1996), ported by decompiling the original
`SimCopter.exe` and reproducing its behaviour rather than approximating it.

Read this first, then `Docs/memory/MEMORY.md`.

---

## 1. Everything you produce goes in the repo

| What | Where |
| --- | --- |
| Durable notes / memories | `Docs/memory/<slug>.md`, indexed in `Docs/memory/MEMORY.md` |
| Scratch: build logs, throwaway scripts, screenshots, decompile dumps | `Docs/scratchpad/` |
| Plans, walkthroughs, format specs | `Docs/` |

Do **not** write these to an agent's machine-local memory or temp scratchpad directory. Those
are untracked and per-machine: nobody else sees them, they die with the machine, and they never
appear in a diff. Full rationale in `Docs/memory/agent-workspace-conventions.md`.

`Docs/memory/MEMORY.md` is the real index — read it at the start of a session. It carries hard-won
traps (unit conventions, sparse dispatch tables, which Ghidra decompiles are wrong) that will cost
you hours to rediscover.

## 2. Building

```powershell
cmd /c "S:\Repos\sim-copter-remake\RebuildUnrealCpp.bat < nul"
```

**Always use `RebuildUnrealCpp.bat`.** Never call `Build.bat` directly — the wrapper pins the
engine root, the `SimCopterRemakeEditor Win64 Development` target, and `-NoLiveCoding`. A Live
Coding session holding `UnrealEditor-SimCopterRemake.dll` otherwise makes the build fail to link
or silently hot-patch, leaving you testing stale code.

The script ends in `pause`, so feed it empty stdin or it hangs. PowerShell 5.1 reserves bare `<`,
which is why the redirect lives inside the `cmd /c` string. A clean build is ~60 s and ends with
`Result: Succeeded`. Engine: `C:\GameDev\UE_5.8`. Details in `Docs/memory/build-and-run.md`.

## 3. Testing

Automation tests live in `Source/SimCopterRemake/Private/Tests/` (20 files) and are named
`SimCopter.<Area>.<Case>`, e.g. `SimCopter.Winch.Constants`.

```powershell
& "C:\GameDev\UE_5.8\Engine\Binaries\Win64\UnrealEditor-Cmd.exe" `
  "S:\Repos\sim-copter-remake\SimCopterRemake\SimCopterRemake.uproject" `
  -unattended -nop4 -nosplash -NullRHI -stdout -FullStdOutLogOutput `
  -ExecCmds="Automation RunTests SimCopter.Winch; Quit"
```

Prefer a headless test over a manual check when the logic is pure (fixed-point maths, table
lookups, parsers). Gameplay and rendering still need the real game — see §7.

## 4. Layout

```
SimCopterRemake/Source/SimCopterRemake/{Public,Private}/
    City/     SC2 city load, terrain, buildings, hangar
    Flight/   helicopter physics, controls, tools
    Formats/  original file-format readers (GEO, DF, SC2, TWK, SIM3D)
    Game/     game modes, session/career subsystems
    Ground/   people, traffic, dispatch, ambient vehicles, particle FX
    Missions/ mission scheduler, fire sim, HUD markers
    UI/       Slate front end, cockpit, hangar shell
    Debug/    dev-only helpers
Docs/         plans, walkthroughs, memory, scratchpad
Tools/        re-agent + ghidra-bridge (Python, venv is gitignored)
Reference/SimCopterOriginalGame   original game files — user-provided and gitignored, but the
                                  source for packaged runtime data
SimCopter/    optional developer override for the same data; `../SimCopter` from the .uproject is
              the repo root here and the automatically populated folder beside the .exe in a
              packaged build
```

Every reader finds that data through `Formats/SimCopterOriginalGamePaths.h` — one candidate list,
not a per-file copy. Add a search root there, never at a call site.

Game-target builds automatically stage the runtime-required original directories (`bmp`, `cities`,
`geo`, `sound`, `tweak`, and `x`) from `Reference/SimCopterOriginalGame` into
`<package root>/SimCopter`. `SimCopterRemake.Build.cs` declares them as loose NonUFS runtime
dependencies through a gitignored `Intermediate/OriginalGameStaging` tree, and
`Config/DefaultGame.ini` remaps that tree to the package root. Do not replace this with a manual
post-package copy, include the original executable/DLLs/manuals/saves, or stage the files inside a
pak: the runtime readers need ordinary filesystem paths. A Game build must fail if any of those six
source directories or its representative required files are absent; an unplayable package is not
an acceptable fallback. Verify packaging against `Manifest_NonUFSFiles_Win64.txt`, as documented in
`Docs/memory/simcopter-packaged-build.md`.

`bUseUnity = false` in `SimCopterRemake.Build.cs`, deliberately: format readers reuse
same-named helpers in anonymous namespaces, and unity chunking collided them whenever a file
was added. Do not turn it back on.

## 5. Porting from the original executable

This is a **decompile-and-port** project, not a reimagining. When you touch ported behaviour:

- Find ground truth first. Main path is the ghidra-bridge over the `.ghidra-exports/` dump:
  `Tools\re-agent\.venv\Scripts\ghidra-bridge.exe decompile 0x4abce0` (also `xrefs-to`,
  `xrefs-from`, `strings`, `search`, `global`). Run from the repo root so `ghidra-bridge.yaml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamesIV4/sim-copter-remake](https://github.com/JamesIV4/sim-copter-remake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
