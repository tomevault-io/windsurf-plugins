---
trigger: always_on
description: Reverse engineering Nikon Coolscan film scanner firmware and Windows drivers to document
---

# Coolscan RE -- Project Context for Claude

## What This Project Is

Reverse engineering Nikon Coolscan film scanner firmware and Windows drivers to document
the complete SCSI communication protocol. **Two deliverables**: (1) Protocol documentation (docs/kb/),
(2) H8/3003 CPU emulator running the actual firmware binary for HIL-free development.

**Primary target**: Coolscan V (LS-50, uses LS5000.md3 module). Later: LS-5000, LS-4000, LS-8000, LS-9000.

## Session Bootstrap (READ THESE IN ORDER)

Every new session, follow this chain:

1. **You are here** -- `CLAUDE.md` (this file) gives you project context
2. **If doing RE work**: Read `docs/log/general.md` → current phase doc → phase log → component log → KB
3. **If doing emulator work**: Read `emulator/docs/log/general.md` → current emulator phase log → component log
4. **Read `docs/phases/phase-NN-<name>.md`** for the current phase (RE) or relevant emulator phase log
5. **Read relevant `docs/log/components/NAME-attempts.md`** or `emulator/docs/log/components/`
6. **Read relevant `docs/kb/` docs** -- existing findings to build upon

Only then begin work.

## Work-Log-Verify Workflow (CRITICAL)

For EVERY unit of work (analyzing a function, tracing a code path, identifying a command), follow this cycle:

### 1. WORK -- Perform the analysis
Do the actual RE work: decompile, trace, pattern match, cross-reference.

### 2. LOG -- Record what you did and found (even failures!)
- **Append** to the relevant component log (`docs/log/components/NAME-attempts.md`)
- Include: date, tool used, target (function/address), what you tried, what you found, confidence level
- **Failed attempts are equally important** -- log what didn't work and why, so we don't repeat it
- Update the phase log (`docs/log/phases/`) with progress

### 3. VERIFY -- Cross-check the finding
- Can this be confirmed from another source? (host-side vs device-side, string xref, etc.)
- Set confidence level (see RE Approach below)

### 4. KB -- Write it up
- **ALL new knowledge MUST go to `docs/kb/`** -- the KB is our final deliverable
- KB docs must be comprehensive enough that a **junior developer** could understand them
- Explain the "why" not just the "what" -- why does this SCSI command exist? What problem does it solve?
- Include hex dumps, decompiled code snippets, diagrams where they help understanding
- Cross-reference related KB docs with links

If a finding is too uncertain (Low confidence), still add it to KB but mark it clearly and list what would be needed to verify it.

## Project Layout

- `CLAUDE.md` -- THIS FILE. Bootstrap for every Claude session
- `ARCHITECTURE.md` -- Call-chain overview, links to detailed KB docs
- `docs/` -- **All model-written documentation**
  - `docs/phases/` -- Phase instruction docs (completion criteria + methodology)
  - `docs/kb/` -- **Knowledge base (ALL findings go here)** -- this is our final output
  - `docs/log/` -- Progress and attempt logs (**APPEND ONLY** - see rules below)
  - `docs/re-backlog.md` -- **RE backlog (Tracks A-E, 6 outstanding tasks as of 2026-07-01)** -- pick targets here
- `binaries/` -- Original firmware + NikonScan 4.03 files (**READ ONLY, never modify**)
- `ghidra/projects/` -- Ghidra project dirs (NikonScan_Drivers, _Modules, _TWAIN, _ICE, CoolscanFirmware)
- `ghidra/scripts/` -- Ghidra Python/Java analysis scripts
- `ghidra/exports/` -- Exported function lists, decompiled code snapshots
- `r2/scripts/` -- radare2 analysis scripts (firmware_init.r2 etc.)
- `scripts/python/` -- PE analysis, RTTI extraction, SCSI pattern matching scripts
- `scripts/shell/` -- bootstrap_ghidra.sh and other shell scripts
- `.claude/skills/` -- RE-specific slash command skills
- `tools/` -- Third-party tools (Ghidra H8/300H SLEIGH module etc.)
- `emulator/` -- H8/3003 CPU emulator (Rust workspace, clean-room implementation)

## Key Binaries (by RE priority)

Full path prefix: `binaries/software/NikonScan403_installed/`

1. `Drivers/NKDUSCAN.dll` (88KB) -- USB transport layer (LS-40, LS-50, LS-5000)
   - Exports: `NkDriverEntry` (1 export, 9 function codes). 14 RTTI classes.
   - Key classes: CUSB2Command, CUSBSession, CUSBDeviceTable, CUSBDevInfo, CSBP2CommandManager
   - Uses DeviceIoControl -> usbscan.sys, WriteFile/ReadFile on bulk pipes
   - **Ghidra project**: NikonScan_Drivers

2. `Module_E/LS5000.md3` (1MB) -- Scanner model module (shared by LS-50 + LS-5000)
   - Exports: MAIDEntryPoint, NkCtrlEntry, NkMDCtrlEntry
   - Loads transport DLL at runtime (LoadLibraryA/GetProcAddress, NOT static import)
   - Constructs SCSI CDBs, calls NkDriverEntry to send them
   - **Ghidra project**: NikonScan_Modules
   - **Note**: No LS50.md3 exists. LS-50 and LS-5000 share this module.

3. `Twain_Source/NikonScan4.ds` (2.2MB) -- TWAIN data source
   - 59 exports (DS_Entry + scanner-specific API: StartScan, GetSource, etc.)
   - 321 RTTI classes (MFC 7.0 based). Full scan workflow orchestration.
   - Model-agnostic: delegates all hardware specifics to .md3 modules
   - **Ghidra project**: NikonScan_TWAIN

4. **Firmware**: `binaries/firmware/Nikon LS-50 MBM29F400B TSOP48.bin` (512KB)
   - CPU: Hitachi H8/3003 (H8/300H, 24-bit, big-endian)
   - Contains INQUIRY strings for both "LS-50 ED" and "LS-5000" (shared lineage)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevihiiin/Nikon-Coolscan-RE](https://github.com/kevihiiin/Nikon-Coolscan-RE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
