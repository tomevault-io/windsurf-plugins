---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository purpose

This is the **Rocket League fork** of [UE Explorer](https://github.com/UE-Explorer/UE-Explorer) — a Windows GUI for browsing and decompiling Unreal Engine 1/2/3 packages (`.upk`, `.u`). The fork's reason to exist is that Rocket League ships heavily customized UnrealScript bytecode: licensee-modified opcode table, custom extended-native opcodes, custom variadic-array natives, etc. Most ongoing work in this repo is **adding/fixing tokens under `UELib/src/Branch/UE3/RL/`** so that RL packages decompile to readable script.

RL packages are encrypted; UELib does not decrypt them. They must first be processed with [RLUPKTool](https://github.com/AltimorTASDK/RLUPKTool) before being opened.

## ⚠️ Validation discipline (read first)

**Every claim in this file and in `UELib/src/Branch/UE3/RL/snapshots/*.md` is a
snapshot in time. Past sessions have committed wrong claims** — the most recent
example was a warning that `sub_7FF6CD38C840` was NOT the real parser; it
actually IS, and the wrong warning misled work for hours before the user pointed
it out. Treat every documented byte→token mapping, handler-address assertion, or
"verified" wire format as a *hypothesis to re-check*, not as ground truth.

**Before acting on a documented claim:**
1. **Verify the binary still matches.** Re-decompile the cited handler in IDA,
   re-read `UStruct::SerializeExpr`'s case for the byte, or re-disassemble a
   function that exercises the byte. Do NOT skip this step even if the claim
   looks recent.
2. **If the claim contradicts what you observe, STOP and ask the user.** Do not
   silently "correct" the doc and proceed — the discrepancy might mean either
   the doc is stale OR your observation is misreading the binary, and the user
   has context (RL build version, which IDB is open, recent fixtures, anything
   they tested manually) that you don't.
3. **After verifying or correcting, update the doc in the same commit as the
   work that depends on it.** Stale claims compound across sessions.

**What this looks like in practice:**
- Memory says "byte 0x2C = StatementWrapper" but the cooked bytecode at the
  site has 3 sub-expressions following → STOP, ask "the runtime handler reads
  1 sub + 1 byte but the cooked stream has more — should I cross-check
  `UStruct::SerializeExpr`?"
- Sentinel function listed as "currently broken" decompiles cleanly → STOP,
  ask "the doc marks `IsGroundHit` as broken but the current output is clean
  — has this been fixed since the doc was written?"
- Stock UE3 says `EX_X = 0xYY`, RL snapshot says different byte → that's
  normal (RL rotates), but verify the current byte by checking BOTH the v868
  GNatives table AND `UStruct::SerializeExpr`, never by trusting either alone.

## Test packages — version pitfall

When verifying token-map changes against actual bytecode, the `.upk` files **must come from the same RL build** as whatever binary is open in IDA / being reverse-engineered. RL rotates its opcode permutation across patches, so a SerializeExpr in the current binary can dispatch byte `0x3E` as EndFunctionParms while older `.upk` files on disk still emit `0x4C` for the same role. A version mismatch silently invalidates every shape inference made from the binary.

To produce a fresh, version-matched fixture set:

```pwsh
# 1. Copy the canonical script packages (TAGame, Core, ProjectX, Engine, IpDrv, GFxUI,
#    GuidCache, Startup, WinDrv) from the live install:
Copy-Item "D:\Games\rocketleague\TAGame\CookedPCConsole\TAGame.upk" `
          "C:\Users\Authority\Desktop\RE stuff\rldecrypted\absolutelynewupks\"
# (repeat for each package)

# 2. Decrypt each in place — RLUPKTool writes a new decrypted file alongside the input:
& "C:\Users\Authority\Desktop\RE stuff\rldecrypted\RLUPKTool.exe" `
  "C:\Users\Authority\Desktop\RE stuff\rldecrypted\absolutelynewupks\TAGame.upk"
```

Then load the **decrypted** output via the uelib MCP (`mcp__uelib__load_package` with `build_target: "RocketLeague"`) and disassemble. Older fixtures under `rldecrypted\`, `rldecrypted\upkbackup\`, `rldecrypted\newupks\`, `rldecrypted\newupks2\` are from earlier RL versions and should not be used to validate work derived from a newer binary.

After decryption, class names in the loaded package have **no namespace prefix**: pass `Actor` to `mcp__uelib__disassemble_function` / `decompile_function`, not `Engine.Actor`. (The package summary's `path` shows `Engine_decrypted` so the qualified path would be `Engine_decrypted.Actor`, but the bare name resolves correctly.)

## Iterating on UELib code with the MCP server attached

The Codex MCP launches `UELib/MCP/publish/Eliot.UELib.MCP.exe` once per session and holds an exclusive lock on it. So the normal `dotnet publish ... -o UELib/MCP/publish` rebuild fails with "process cannot access the file" while a session is live.

Workflow that works (PowerShell automation — much faster than the manual stop/copy/restart):

1. Make code changes (any project — both `Eliot.UELib` and `Eliot.UELib.MCP` get bundled).
2. Build into a sibling directory:
   ```pwsh
   dotnet publish UELib/MCP/Eliot.UELib.MCP.csproj `

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lauralex/ue-explorer](https://github.com/lauralex/ue-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
