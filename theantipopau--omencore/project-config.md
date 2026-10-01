---
trigger: always_on
description: Guidance for AI agents working in this repo. Read this first, then the current cycle's
---

# CLAUDE.md — OmenCore

Guidance for AI agents working in this repo. Read this first, then the current cycle's
`docs/ROADMAP_v*.md` and `docs/CHANGELOG_v*.md`.

## What this is

OmenCore is an open-source replacement for HP OMEN Gaming Hub (OGH) for HP OMEN / Victus laptops
(and some desktops): fan control and curves, performance modes, GPU power (TGP/PPAB), MUX switch,
CPU undervolt / power limits, keyboard RGB, peripheral RGB (Corsair/Logitech/Razer), telemetry.
GitHub: `theantipopau/omencore`. Maintainer: Matt (theantipopau). Users report via GitHub issues,
diagnostics exports (zip/json attached to issues), Discord, forks and PRs.

## Layout

| Path | What |
|---|---|
| `src/OmenCore.Core/` | Hardware + service logic shared by all frontends (net8.0-windows) |
| `src/OmenCore.Core/Hardware/` | `HpWmiBios` (HP WMI BIOS calls), `WmiFanController`, `FanController` (EC), `ModelCapabilityDatabase`, `CapabilityDetectionService`, `DeviceCapabilities`, `HardwareWorkerClient`, PawnIO EC/MSR access |
| `src/OmenCore.Core/Services/` | `FanService`, `FanVerificationService`, `HardwareWatchdogService`, `ConfigurationService`, `KeyboardLighting/` backends (`WmiBiosBackend`, `EcDirectBackend`), power/perf services |
| `src/OmenCoreApp/` | Main WPF app (MVVM: `ViewModels/`, `Views/`, `App.xaml.cs`) |
| `src/OmenCore.HardwareWorker/` | Out-of-process LibreHardwareMonitor worker (isolates native crashes: NVML/AMD ADL) |
| `src/OmenCore.Cli/` | CLI |
| `src/OmenCore.Linux/` + `.Linux.Tests/` | Linux daemon/CLI (hp-wmi sysfs), net8.0 |
| `src/OmenCore.Avalonia/`, `src/OmenCore.Desktop/` | Cross-platform UI (Desktop csproj still on old 3.6.3 version — ignore on bumps unless asked) |
| `src/OmenCoreApp.Tests/` | xUnit tests for Core + App (~1650 tests) |
| `installer/OmenCoreInstaller.iss` | Inno Setup installer |
| `docs/` | Per-version CHANGELOG / ROADMAP, evidence docs, bug-report logs |
| `website/` | GitHub Pages site (`pages.yml`) |
| `.github/workflows/` | `ci.yml` (build+test on windows-latest, Linux job), `release.yml` (on tag), `alpha.yml`, `linux-qa.yml` |

## Build & test

```bash
dotnet build OmenCore.sln
dotnet test OmenCore.sln                       # full suite, ~7 min; run in background
dotnet test src/OmenCoreApp.Tests/OmenCoreApp.Tests.csproj --filter "FullyQualifiedName~DeviceCapabilitiesTests"
```

- The full suite must stay green before every push (last known: 1658 App tests + 30 Linux tests).
- Build should be 0 warnings / 0 errors.
- Tests that touch config use `[Collection("Config Isolation")]` and `OMENCORE_CONFIG_DIR` temp dirs.
- Shell is Windows (Git Bash / PowerShell). Repo root is `E:\OmenCore\omencore`.

## Core working discipline (non-negotiable)

1. **Evidence first.** Root-cause from real data: diagnostics exports, logs, the reporter's board ID
   and BIOS version. Download attachments (`github.com/user-attachments/...`) with curl and read them.
   Never guess a fix from the symptom alone.
2. **Evidence gate.** Distinguish clearly between *confirmed* (verified on real hardware by a
   reporter) and *implemented, pending confirmation*. Label them that way in changelogs, roadmap,
   model-database notes and GitHub replies. Never claim hardware behaviour you haven't seen evidence of.
3. **Narrow fixes.** Change only what the evidence supports. Check who else a gate/flag affects
   (e.g. every board in the DB) before broadening or narrowing it; write tests pinning both sides.
4. **Tests with every fix** where feasible — ideally one that fails on the old code.
5. **Honest replies.** When answering on GitHub, say what's fixed, what's pending, what you need from
   the reporter (usually a diagnostics export or a Guided Fan Verification run). Only claim what the
   repo actually contains (e.g. don't say "credited in changelog" before it is).
6. **Credit contributors.** If a PR/fork raised an issue first, credit it even if you reimplement it.
   Check open PRs and forks *before* implementing to avoid duplicate work.
7. **Don't release, tag, close issues or post publicly without the maintainer's go-ahead** unless
   the current instruction clearly covers it.

## Model capability database (`ModelCapabilityDatabase.cs`)

- Boards matched by exact **ProductId** (4-hex board ID, e.g. `8BBE`, `88F8`, `8C2F`).
  `ModelNamePattern` is a fallback only; `RequiredCpuVendor` prevents Intel/AMD variants inheriting
  each other's entry (cause of #115/#172). Unknown boards fall back to a family default
  (Victus family default = 1 fan, which is often wrong).
- New entries: `UserVerified = false` unless a user confirmed it; `Notes` must cite the evidence
  source (issue #, BIOS version, what was verified). Leave unproven features off (curves, GPU boost,
  undervolt) — enable later on evidence.
- Add a `ModelCapabilityDatabaseTests` test for each new entry (resolves exactly, key flags).
- `HasFourZoneRgb` does **not** control zone count; flipping it removes colour control entirely.

## Hard-won technical facts

- **HP WMI BIOS** is the primary control path; EC direct (PawnIO) is the fallback/older path.
  Firmware often *accepts* a command and ignores it (e.g. `SetFanMax`) — verify via readback and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theantipopau/omencore](https://github.com/theantipopau/omencore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
