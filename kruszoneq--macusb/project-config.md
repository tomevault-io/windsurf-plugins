---
trigger: always_on
description: This file defines how AI coding agents should work in this repository.
---

# AGENTS.md for macUSB

This file defines how AI coding agents should work in this repository.

## Scope and precedence

- This repository intentionally keeps `AGENTS.md` under `docs/` for repository cleanliness.
- Direct user instructions in chat have top priority, except explicit protected-branch safety rules defined in this file.
- If instructions conflict, follow the stricter safety rule and ask for clarification before proceeding.
- `docs/AGENTS.md` is the single source of truth for process, commit, and changelog rules.

## Mandatory context bootstrap

Before implementation, recommendations, or review:

1. Read this file in full.
2. Read `docs/reference/README.md`.
3. Read the topic-specific runtime reference(s) for the task scope.
4. For downloader-scope tasks, read `docs/reference/features/downloader/DOWNLOADER.md`.
5. For helper-scope tasks, read `docs/reference/features/helper/HELPER.md`.
6. If scope is cross-cutting or uncertain, read:
   - `docs/reference/core/APP_RUNTIME_OVERVIEW.md`
   - `docs/reference/core/USER_FLOW.md`
   - `docs/reference/core/FILE_STRUCTURE.md`
   - `docs/reference/core/RISK_AREAS.md`
   - plus all affected topic references listed in `docs/reference/README.md`.
7. Build one active ruleset before changing code.

## Repository map

- App runtime: `macUSB/`
- Privileged helper: `macUSBHelper/main.swift`
- Runtime reference index: `docs/reference/README.md`
- Downloader runtime reference: `docs/reference/features/downloader/DOWNLOADER.md`
- Helper runtime reference: `docs/reference/features/helper/HELPER.md`
- Release notes: `docs/CHANGELOG.md`
- Agent process rules: `docs/AGENTS.md`

## Critical runtime invariants (must preserve)

These are the non-negotiable runtime contracts. If a task touches any of them, preserve behavior unless the user explicitly requests a change.

### Permissions and startup gating

- App operation depends on both:
  - Full Disk Access for `macUSB`
  - Allow in the Background approval for helper operation
- Startup/activation flow must refresh and surface permission/helper readiness state.
- Missing required permissions must be visible in UI and may block reliable helper execution.
- External drive support must default to disabled on launch/termination unless the user explicitly enables it in app options.

### User flow and destructive safety

- Main screen sequence remains:
  - `WelcomeView -> SystemAnalysisView -> UniversalInstallationView -> CreationProgressView -> FinishUSBView`
- Start of installer creation is destructive and must require explicit confirmation.
- Cancel path must preserve deterministic cleanup/result behavior.

### USB target safety and capacity

- Before installer recognition, required capacity in UI is unresolved (`-- GB`).
- Capacity rules:
  - major version `<= 14`: `16 GB` UI target and `15_000_000_000` bytes threshold
  - major version `>= 15`: `32 GB` UI target and `28_000_000_000` bytes threshold
- Proceed action stays blocked until selected target passes validation.
- APFS-selected target must block proceed and require manual reformat in Disk Utility.
- In PPC flow, target formatting behavior is specialized and must not be forced through standard preformat assumptions.

### Detection and compatibility routing

- Analysis flags are the source of truth for workflow branch selection.
- Supported detection families include modern, legacy, restore-legacy, PPC, Sierra-specific, Catalina, and Mavericks handling.
- Panther must remain explicitly unsupported.
- For `.cdr` and `.iso`, if image is already manually mounted in macOS, analysis must be blocked with user guidance to unmount and retry.

### Installer workflow branching

- Standard path uses `createinstallmedia` family behavior.
- Legacy restore and Mavericks use restore-style pipelines.
- PPC uses dedicated PPC formatting/restore behavior.
- Catalina/Sierra use dedicated handling where required.
- Temp ownership and cleanup behavior must remain deterministic.

### Helper architecture and security invariants

- Privileged install operations must run via `SMAppService` + LaunchDaemon helper.
- No terminal fallback privileged path.
- App/helper naming, mach service, listener, plist wiring, bundle identifiers, and signing compatibility must stay aligned.
- Helper progress must remain observable in app state.
- Live tool output remains diagnostic data, not stage UI source of truth.

### UI/UX invariants

- Flow window contract remains fixed at `550 x 750`.
- Bottom action zones use `BottomActionBar` with `safeAreaInset(edge: .bottom)`.
- Use compatibility wrappers from `LiquidGlassCompatibility.swift`:
  - `macUSBPanelSurface`
  - `macUSBDockedBarSurface`
  - `macUSBPrimaryButtonStyle`
  - `macUSBSecondaryButtonStyle`
- Spacing/radii use `MacUSBDesignTokens`.
- `DEBUG` UI must not appear in Release builds.

### Localization invariants

- Source language is Polish (`pl`) in `Localizable.xcstrings`.
- New UI copy is authored in Polish first.
- Runtime non-`Text` user-facing strings use `String(localized:)`.
- Helper localization keys and app-side rendering keys must remain synchronized.
- For strings that mirror macOS system UI, notifications, or alerts, match Apple’s terminology in each language instead of inventing custom wording.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kruszoneq/macUSB](https://github.com/Kruszoneq/macUSB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
