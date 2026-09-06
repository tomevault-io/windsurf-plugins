---
trigger: always_on
description: Practical, persistent instructions for AI agents working on this repository.
---

# AGENTS.md — Windows Hello Fix

Practical, persistent instructions for AI agents working on this repository.

## 1. Baseline

This branch (`release-v2.0-restructured`) contains a **behavior-preserving, mechanical extraction** of the original monolithic `release-v2.0/MyForm.h` into a modular `src/core/` tree. The extraction was verified by static analysis and a successful `Release|x64` build; runtime behavior is intended to be identical to v2.0.

Active source tree (inspect with `git ls-files` before assuming more):

```
main.cpp
MyForm.h                 # SHIM: #include "src/core/MyForm.h" (do not delete; preserves include path)
ProductionUtilities.h    # legacy/unused helper; not part of build flow
Windows_Hello_Fix_v2_0.{vcxproj,vcxproj.filters,sln}
Windows_Hello_Fix_v2_0.rc / _resources.rc / resource.h / resource1.h / app.manifest
src/core/
  MyForm.h            # declaration-only: class, CameraDeviceInfo, extern globals, forward decls
  MyForm_Camera.cpp   # native camera pipeline + Disable/Enable/Restore members
  MyForm_Config.cpp   # config.txt + diagnostic.log + save/load + target resolution
  MyForm_Core.cpp     # ctor/dtor/finalizer/InitializeComponent/MyForm_Load
  MyForm_Events.cpp   # WndProc: session/power/shutdown dispatch
  MyForm_System.cpp   # command parsing + wake listener
  MyForm_UI.cpp       # FormClosing + btnToggle_Click
docs/                  # current project documentation (see docs/ rules)
release-v2.0/          # canonical v2.0 reference (gitignored; DO NOT MODIFY)
x64/Release/install_script.nsi   # NSIS installer (load-bearing)
```

`MyForm` remains the **single, central state owner**. No controller classes were introduced.

## 2. Architecture policy

- The `src/core/` files above are the **known-good modular baseline**. Preserve their exact filenames and current responsibilities unless a task explicitly concerns architecture.
- Do **not** casually rename, split into many smaller files, merge back into a monolith, or introduce a controller/abstraction layer merely for aesthetics.
- Adding new files/folders or improving module boundaries **is allowed** when there is a real engineering reason. Follow the architectural-change rule (§8) for such work.
- New features should go in new appropriate files/folders rather than being forced into the existing seven.

## Stable Core Directory

`src/core/` contains the existing stable HelloFix implementation.

Agents should preserve this directory and its current file structure by default.

Bug fixes may modify existing files in `src/core/` when the affected behavior
belongs there, but agents should avoid adding unrelated new functionality to
existing `src/core/` files.

New features should normally be implemented in new appropriate source files or
folders outside `src/core/` when that provides a clearer separation of
responsibility.

The `src/core/` structure is a stability boundary, not an immutable
architecture. Significant restructuring of it requires investigation,
planning, and approval before implementation.

## 3. Behavioral source of truth

- For behavior-sensitive investigation, compare against `release-v2.0/MyForm.h` (known-good reference). It is gitignored and must not be modified.
- If current and reference behavior differ unexpectedly: investigate → determine if intentional → identify exact behavioral impact → do **not** silently revert or rewrite.

## 4. Core principles

1. Behavioral correctness. 2. Preserve known-good behavior. 3. Minimal changes. 4. Clear evidence before modification. 5. Small, reviewable changes. 6. Proper build verification. 7. Clear documentation.

Philosophy: **Reliable behavior first, clean architecture second, optimization third.** Prefer "slightly less elegant but behaviorally proven" over "elegant but behaviorally uncertain."

## 5. Protected behavior (investigate before modifying)

- **Camera hardware** — SetupAPI / Configuration Manager calls, device selection, verification, retries, recovery sequences, `Sleep` timings, error handling. Startup/UI/installer issues must NOT trigger unrelated camera refactoring.
- **Session & power events** — WTS notifications, lock/unlock, suspend/resume, lid/button handling, `WndProc` dispatch, cooldown/dedup (1500 ms windows), `isAlreadyDisabled` static. Preserve ordering and timing.
- **Single-instance** — named objects `Global\WindowsHelloFix_AppMutex` and `Global\WindowsHelloFix_WakeupEvent`. Do not introduce another mutex/event system; preserve wake and second-instance behavior.
- **GUI visibility** — hidden background startup (`--background`, `Opacity=0`), interactive startup, taskbar behavior, wake, `FormClosing` (cancel → minimize to background, `isBackgroundMode`). Do not make background launches visible.
- **Startup / command-line** — arguments `--background`, `--disable-camera`, `--enable-camera`, `--restore-camera`, `--repair-camera` are behavior-sensitive; preserve names and processing order.
- **Installer (NSIS)** — `x64/Release/install_script.nsi`: task names/args, privilege level, triggers, install order, uninstall cleanup, executable deployment, post-install launch. Inspect the full flow before changing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shivu516/Windows-Hello-Fix](https://github.com/Shivu516/Windows-Hello-Fix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
