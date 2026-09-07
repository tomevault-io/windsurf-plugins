---
trigger: always_on
description: Windows 11 ISO builder. Windows-native. Requires `pwsh` 7.6.0+ for backend/runtime scripts (offline staging downloads the latest available 7.6.x+).
---

# WinMint — Agent context

Windows 11 ISO builder. Windows-native. Requires `pwsh` 7.6.0+ for backend/runtime scripts (offline staging downloads the latest available 7.6.x+).
Development usually happens from WSL or an editor, but all project scripts execute on Windows.

`AGENTS.md` is the compact implementation contract for coding agents. User-facing product behavior, usage examples, and rationale belong in `README.md`.

The core design rule: **UI creates intent. Engine performs work. Reports explain work. FirstLogon finishes live-user setup.**

PowerShell owns the backend and all real product work: profile normalization, ISO/WIM servicing, setup payloads, FirstLogon, reports, release tooling, and validation tooling. The actual build logic must stay headless. The shipped UI is a WebView2 host (`WinMintSetupShell.exe`) plus HTML/JS wizard assets under `assets/runtime/setup/setup-shell/`; it creates intent, previews choices, and invokes the headless PowerShell engine through `tools/ui-bridge/`. It must not own servicing, setup orchestration, offline registry edits, or live-user package installation.

Backend composition uses thin PowerShell modules under `src/runtime/modules/`. Public entrypoints import `WinMint.Bootstrap` (elevation/relaunch), `WinMint.Profile` (profile authoring for the UI bridge), and `WinMint.Engine` (dot-sources `src/runtime/image/WinMint.ps1` as the single canonical runtime load order). Do not add parallel per-area module wrappers unless they have real consumers.

## Product stance (opinionated)

- **User ISO is the truth.** There is no pinned “golden” Windows build inside the repo. Whatever **source ISO** the user picks (subject to documented minimums, e.g. Windows 11 **25H2+** in `README.md`) is the version DISM services. AppX prefixes and registry stamps are **best-effort** against common SKUs; odd OEM bundles may need follow-up outside the wizard.
- **Home-first target.** The primary product target is Windows 11 Home / Home Single Language / en-US. Generated profiles use schema v3, default visible region values of en-US / GeoID 244, and default fixed-edition builds to standard `Windows 11 Home`. Do not silently fall back to Pro when a fixed Home image is missing.
- **Source choice stays simple.** The user-provided official Microsoft ISO is the source of truth. WinMint does not expose UUP Dump selection or conversion as a public product choice. Do not bundle Microsoft payloads or silently download them; require only a high-level consent/automation acknowledgement when network download or conversion is needed.
- **No debloat/performance wizard flags.** Defaults live in engine/profile/setup scripts only—one coherent WinMint posture, not a choice matrix.
- **Subtractive default + opt-in "keep" flags, not granular toggles.** The default build removes full serviceable AI, Xbox/gaming, and the developer tweaks are folded in as baseline. Edge browser noise/AI/promo policy is applied by default. Edge stays installed (`keep.edge` is always true); WinMint does not automate Edge uninstall and does not present a keep/remove Edge choice (`-KeepEdge` is an accepted no-op). Opt-in keep flags suppress one domain each: `-KeepGaming` (keep Xbox/Game Bar AppX and suppress gamebar-policy), `-KeepCopilot` (keep all Copilot+ AI features *except Recall*, which is always removed). Game Mode / HAGS (`gaming-performance-policy`) is baseline for all builds, not gated on `-KeepGaming`. `-DesktopUI` is still an additive shell selection; `-Install windhawk,yasb,komorebi` picks window-manager tooling; editors stay opt-in while WSL2 is always enabled and distro selection remains explicit. WinMint is WSL-first, but Linux distro installs remain explicit. The baseline also uses XDG-style dotfolders for config/data/state/cache, with a temp-backed runtime directory, so XDG-aware tools avoid `AppData` by default. Dev Drive is Off by default; opt in with `target.devDrive` / `-DevDrive Partition|VhdDynamic` and `-DevDriveSizeGb 64|128|256` (default 128 when on). Partition is carved at Setup via diskpart (requires `AutoWipeDisk0` or `DualBootReserved`); VhdDynamic creates an expandable VHDX at FirstLogon. Not for WSL Linux trees. There is no `Developer`/`CopilotPlus` group and no `profileGroups`/`setupOption` vocabulary — the keep flags are the only profile dimension.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yanai-sh/winmint_v1](https://github.com/yanai-sh/winmint_v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
