---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MapleClaude** is a brand-new, modernized 64-bit MapleStory v95 game client written in C# 13 / .NET 10. It is greenfield — not a fork of any existing client — and is built to connect to the Kinoko Java server (v95, locale 8, patch `"1"`). The client speaks the standard v95 wire protocol (Shanda + Maple-AES + IGCipher IV rotation) and reads standard GMS v95 WZ files for visuals.

The codebase is organized for long-term, multi-phase extension. Phase 1 delivers the full pre-game flow: launch → title → login → server/world select → character list → character create → PIN entry → channel migrate handoff. Phase 2 picks up at field load. See `docs/roadmap.md` for the full roadmap.

## Build & Run

```powershell
dotnet restore
dotnet build -c Debug
dotnet test
dotnet run --project src/MapleClaude
```

Or open `MapleClaude.slnx` in Visual Studio 2026.

Runtime requires a running Kinoko login server (defaults to `127.0.0.1:8484`) and a directory of standard v95 WZ files. See `README.md` for the environment variables.

### Hot-reload dev loop (no close/build/deploy/reopen)

```powershell
.\watch.ps1
```

Runs the client under `dotnet watch run` (from `bin/`, not the deployed single-file exe). On save,
**method-body edits** (Draw/Update/layout logic) apply live via .NET Hot Reload; **structural edits**
(new/changed fields, field initializers, signatures, types) are "rude edits" → `dotnet watch`
auto-rebuilds and relaunches. Either way there's no manual cycle. The script finds the WZ folder
from `MAPLECLAUDE_WZ_DIR`, then `MAPLECLAUDE_DEPLOY_DIR`, then `.deploy.local` (the deploy folder
*is* the WZ folder) and passes `-p:NoAutoPublish=true -p:NoAutoDeploy=true` so each rebuild skips
the single-file publish.

`watch.ps1` also sets `MAPLECLAUDE_DEBUG=1`, which opens the **live layout overlay**: tick "drag"
and drag a registered position knob (e.g. the `CharCreate` panels/fields) to tune layout with
**zero rebuild**, then bake the value into the default. Any non-empty `MAPLECLAUDE_DEBUG` enables it.

### Single-file `.exe` is the default build output

**Every `dotnet build` (and every F5 in Visual Studio) auto-produces a single self-contained `MapleClaude.exe`** at:

```
artifacts/<Configuration>/MapleClaude.exe
```

The .NET runtime, MonoGame, and all native dependencies are bundled. Drop the `.exe` next to your `UI.wz` / `Map.wz` and run it.

Size: **~73 MB** for both Debug and Release. The single-file bundler's compression and `IncludeAllContentForSelfExtract` options are intentionally **off** — both trigger `STATUS_STACK_BUFFER_OVERRUN (0xc0000409)` on launch when combined with MonoGame's native deps under .NET 10. Do not re-enable them without a verified fix upstream.

First launch takes 30–90 s while Windows Defender scans the binary; subsequent launches are sub-second. Add the deploy folder to Defender exclusions to skip the scan.

The MSBuild target `AutoPublishSingleFile` in `src/MapleClaude/MapleClaude.csproj` hooks `AfterTargets="Build"` and invokes the standard `Publish` target. It auto-skips for:
- Visual Studio design-time builds (IntelliSense passes).
- `dotnet test` (the test projects don't reference `MapleClaude.csproj`, so the target never fires).
- Anything passing `-p:NoAutoPublish=true`.

Escape hatch for the fastest possible iteration:

```powershell
dotnet build -c Debug -p:NoAutoPublish=true      # ~3 s; skips publish, multi-file output only
```

`PublishTrimmed` and `PublishAot` are explicitly disabled because MonoGame relies on reflection (content pipeline, type lookup) that breaks under trimming.

### Auto-deploy on every build (MapleStory folder)

If you want every `dotnet build` (and every Rebuild Solution in VS2026) to also **drop `MapleClaude.exe` into your MapleStory folder**, configure a deploy directory once. The MSBuild target `AutoDeploySingleFile` runs right after `AutoPublishSingleFile` and copies the freshly built exe.

Resolution order:

1. **Env var** `MAPLECLAUDE_DEPLOY_DIR` (preferred — visible to any tool, persists across projects).
   ```powershell
   [Environment]::SetEnvironmentVariable('MAPLECLAUDE_DEPLOY_DIR', 'X:\path\to\maplestory', 'User')
   ```
   Then **restart Visual Studio** so the new VS process inherits the env var.

2. **`.deploy.local` file** at the repo root containing the path on a single line (fallback — no restart needed, easier per-machine setup).
   ```powershell
   "X:\path\to\maplestory" | Set-Content .deploy.local -Encoding ascii
   ```

Both options are gitignored. If neither is set, the deploy step prints a skip hint and continues. To disable for one build: `dotnet build -p:NoAutoDeploy=true`.

`publish.ps1` is still around for explicit Release + deploy from the CLI, but it's no longer required since `dotnet build` does the same work automatically.

## Repository Layout

```
src/
  MapleClaude/         main executable (MonoGame entry, App/, Stages/, UI/, Platform/)
  MapleClaude.Net/     Crypto/, Packet/, Session/, Handlers/
  MapleClaude.Wz/      WzPackage, WzReader, WzImage, WzCanvas, WzCrypto

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MujyKun/MapleClaude](https://github.com/MujyKun/MapleClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
