---
trigger: always_on
description: (Authoritative quick brief. Keep user-facing answers in the user's language; code comments in English only.)
---

# PolicyPlusMod – Coding Agent Onboarding (Concise, Rev. 2025-09 R2 – WinUI 3 Only)

(Authoritative quick brief. Keep user-facing answers in the user's language; code comments in English only.)

## Refactor Work (Specs in Docs)

When working on the ongoing refactor (starting from `PolicyPlusCore/Core`), treat the following documents as the primary source of truth and keep them up to date:

- `Docs/Architecture/Refactor/README.md`
- `Docs/Architecture/Refactor/TestPlan.md` (spec-first; behavior preservation contract)
- `Docs/Architecture/Refactor/Overview.md`
- `Docs/Architecture/Refactor/ChangeLog.md`
- `Docs/Architecture/Refactor/ADR/*` (decisions and rationale)

Default stance for refactor tasks: add/adjust tests per `TestPlan.md` before moving code, keep public APIs stable, and do incremental, reversible changes (facades/adapters).

## 1. What This App Does
PolicyPlusMod loads Administrative Template (ADMX/ADML) definitions, lets users search / view / edit Windows Group Policy (registry-based) settings, and export/import them (REG / semantic policy). The modern WinUI 3 UI (`PolicyPlusPlus`) is the only UI. Core logic resides in `PolicyPlusCore` and stays UI-agnostic. An elevation host (`PolicyPPElevationHost`) handles privileged operations.

## 2. Tech Stack
- Language: C#
- Projects / Targets:
  - Core: `PolicyPlusCore` → `net8.0-windows`
  - WinUI 3 UI: `PolicyPlusPlus` → `net8.0-windows10.0.26100.0` (desktop app; min platform 10.0.17763.0)
  - Elevation Host: `PolicyPPElevationHost` → `net8.0-windows` (console/elevation helper)
  - Tests: `PolicyPlusModTests`, `PolicyPlusPlus.Tests.UI` (xUnit)
- Windows App SDK 1.8; CommunityToolkit WinUI controls
- SDK pinned by `global.json` (8.0.413) – do not drift
- Packaging: Dual path – Packaged (MSIX) for standard installs and Unpackaged (Velopack-enabled) for self-updating portable style. Config names with *-Unpackaged select the Velopack/update path; others build MSIX packages.


## 3. Repository Layout (High Value)
```
PolicyPlusPlus.sln
PolicyPlusCore/            Domain models & policy processing
PolicyPlusPlus/            WinUI 3 UI
PolicyPPElevationHost/     Elevation helper process
PolicyPlusModTests/        Core + limited UI logic tests
PolicyPlusPlus.Tests.UI/   WinUI 3 focused tests
Docs/                      Architecture & terminology
.github/                   Automation & this guide
```
Generated output: `artifacts/`, `obj/`, `bin/`.

## 4. Key Domain Concepts
- ADMX + ADML parsed into `AdmxBundle`.
- Policies → registry mutations (evaluation logic in Core).
- `IPolicySource` abstracts persistence targets (POL file, registry, etc.).
- Policy state = tri-state + element values.
- Pending changes tracking (queue, apply, discard, reapply) in UI services.

## 5. Build & Run (cmd.exe)
```
dotnet --version
dotnet restore PolicyPlusPlus.sln
dotnet build PolicyPlusPlus.sln -c Debug-Unpackaged
dotnet run --project PolicyPlusPlus/PolicyPlusPlus.csproj -c Debug-Unpackaged
```
Release build:
```
dotnet build PolicyPlusPlus.sln -c Release-Unpackaged
```
Packaged vs Unpackaged quick reference:
```
Debug / Release: Packaged=true (MSIX; Store-like deployment)
Debug-Unpackaged / Release-Unpackaged: Packaged=false (Velopack, self-contained when configured)
```
Publishing guidance:
1. Unpackaged: invoke publish profile (win-<Platform>-standalone) then Velopack pipeline packages outputs.
2. Packaged: standard dotnet build/publish generates MSIX with signing per certificate settings.

## 6. Coding Guidelines
- Favor small, clear methods; keep hot paths lean.
- Core first: put reusable logic in `PolicyPlusCore`; UI layer orchestrates only.
- Nullability: no blanket suppressions—fix causes.
- Warnings treated as errors.
- Only `IPolicySource` implementations touch the registry.
- Minimize public surface; prefer `internal`.
- Avoid new external dependencies unless essential (prefer BCL).
- No blocking of UI thread with file / network I/O (prefer async + background queue).
- Cache repeated lookups; avoid repeated XML parse; keep heavy LINQ off hot loops.
- Respect `global.json`.
- XAML code-behind minimal: prefer ViewModels / Services. Event handlers should delegate quickly.

Comment style (strict):
- Intent / rationale only when non-obvious.
- Prohibited prefixes (case-insensitive): `// New:`, `// Added:`, `// Removed:`, `// Deleted:`, `// Modified:`, `// Changed:`, `// Update:`, `// Old:` etc.
- No BEGIN/END change markers.
- Actionable TODOs only: `// TODO: <concise actionable item>`.
- Do not restate method name or parameters.
- No version / diff history commentary.

## 7. Typical Safe Feature Flow
1. Search repo to confirm feature not already implemented.
2. Extend domain model / logic in Core (add tests simultaneously).
3. Add or update xUnit tests (happy path + at least one edge case).
4. Integrate into WinUI 3 (graceful degradation if partial data).
5. Run `dotnet test` (Debug; optionally Release) until clean.
6. Assess performance with large ADMX set if logic on critical path.
7. Update docs only if public semantics changed.

## 8. Test Guidance
```
dotnet test PolicyPlusPlus.sln -c Debug-Unpackaged
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tttza/PolicyPlusPlus](https://github.com/tttza/PolicyPlusPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
