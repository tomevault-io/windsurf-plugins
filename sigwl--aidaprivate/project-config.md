---
trigger: always_on
description: This repository is AiDA: a Windows reverse-engineering toolkit with an IDA Pro plugin, a standalone ImGui/DX11 IDE, and a kernel driver (WhosWho) for functional RE features. Read this file before making changes.
---

# AiDA Agent Instructions

This repository is AiDA: a Windows reverse-engineering toolkit with an IDA Pro plugin, a standalone ImGui/DX11 IDE, and a kernel driver (WhosWho) for functional RE features. Read this file before making changes.

**AiDA is a personal-only tool. There is NO license system, NO server, NO ARC, NO anti-tamper, NO protector, NO Sentinel driver, NO string obfuscation — all of that was permanently deleted in August 2026. Do not reintroduce any protection, obfuscation, virtualization, packing, licensing, activation, phone-home, or self-defense mechanism. AiDAStandalone.exe must boot straight into the IDE with no key prompt and no network requirement.**

## Build System

AiDA uses CMake 3.25+ with Ninja generator and MSVC on Windows. C++17, C, and ASM_MASM (kernel_symbols.cpp alone compiles as C++20).

- **BUILDS:** The user runs builds themselves unless they explicitly tell the host AI to build. When told to build: incremental: `.\build-host.cmd`. Full clean rebuild including drivers: `.\build-host.cmd -FullClean`. Driver rebuild only: `.\build-host.cmd -Drivers -CleanDrivers`. Dry-run plan: `.\build-host.cmd -PlanOnly -FullClean`.
- The wrapper writes timestamped logs under `%TEMP%\aida-build-*`, stable logs at `%TEMP%\aida_driver_build_out.txt`, `%TEMP%\aida_configure_out.txt`, `%TEMP%\aida_build_out.txt`, `%TEMP%\aida_build_verify_out.txt`, and a machine-readable summary at `%TEMP%\aida_build_summary.json`.
- The only supported configure/build preset is `ninja-msvc-release` in `CMakePresets.json`.
- Driver embed pipeline: `driver\WhosWho\WhosWho.sln` (contains WhosWho + WindMapper) builds `build-ninja\Release\WhosWho.sys`; the CMake `encrypt_drivers` target then runs `python src/encrypt_whoswho.py --from-binary build-ninja/Release/WhosWho.sys` to generate the PLAIN (unencrypted) embedded byte array in `src/whoswho_embedded.h`, which `src/driver_loader.cpp` includes. The driver binary must exist before configure/build, or `driver_loader.cpp` fails on the missing header.

## Subagent Policy

**For very massive tasks (large refactors, multi-file redesigns, UI overhauls, cross-module implementations), the host AI MUST dispatch implementer/designer subagents.** Solo inline-editing on big jobs is wrong; parallel implementer/designer subagents with surgical briefs is the default.

For serious crash, hang, Test Lab, MCP startup, or driver-backed debugging tasks, use a dedicated subagent when the investigation spans multiple files or needs heavy instrumentation. The most valuable subagent pattern is a tightly scoped logging/instrumentation implementer: give it the exact evidence window from logs, tell it to add comprehensive breadcrumbs across that path, and forbid it from diagnosing beyond evidence or building.

**SUBAGENTS ARE FORBIDDEN FROM BUILDING. NEVER. UNDER ANY CIRCUMSTANCE.**

**SUBAGENTS: THE WRITE TOOL IS FORBIDDEN ON ALREADY-EXISTING FILES — EVER.** Using Write on an existing file replaces ALL of its contents; that is the entire reason for the rule (a subagent once overwrote `helpers.cpp` with a Write call). Subagents modify existing files ONLY with the Edit tool; deletions happen via bash (`git rm` / `Remove-Item`). **Write IS allowed — and expected — for creating NEW files** (e.g. plan files under `plans/`, new source files the host assigned). Before any Write, the subagent must verify the target does not already exist. Bulk mechanical transforms may be done with a script created under `%TEMP%` (never in the repo) and verified hunk-by-hunk on one file before running the rest.

**SUBAGENTS ARE FORBIDDEN FROM GIT MUTATIONS.** No `git commit`/`add`/`push`/`reset`/`rebase`/`stash`. Only `git rm` for deletions is allowed.

**THE HOST AI IS FORBIDDEN FROM GIT MUTATIONS. NEVER run `git commit`, `git add`, `git push`, `git reset`, `git rebase`, `git stash`, `git commit --amend`, force-push, or any other git mutation — EVER, under any circumstance, even after completing a wave of work. There is NO standing instruction to commit or push; that instruction was permanently revoked in August 2026. The owner performs all commits and pushes personally. If a commit seems needed, stage nothing and tell the owner the working tree is ready for them to commit.**

- Subagents are scoped to: **implement / code / think / investigate / design / audit / research**. That is the entire allowed surface.
- When dispatching, explicitly tell the subagent: **Do not build. Do not run cmake, msbuild, ninja, or vcvars. Do not use the Write tool. Do not run git mutations.**
- Subagents are not alone in the codebase. Tell them not to revert other changes and to work with existing edits.
- The host AI is also never alone in this project. Unrelated modified files are user-owned workspace state. Do not tell subagents that unrelated dirty files are suspicious; just scope the task clearly and require subagents to list the files they changed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sigwl/AiDAPrivate](https://github.com/sigwl/AiDAPrivate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
