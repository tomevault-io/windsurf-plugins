---
trigger: always_on
description: provides analytics, includes a paid feature system with activation codes, and
---

# Arcadia Tracker — Claude Code Project Instructions

## Project Overview
WPF game companion app (.NET 8.0 / C# 12) for Star Rupture. Tracks save data,
provides analytics, includes a paid feature system with activation codes, and
provides real-time QA telemetry with event-driven bug capture and reporting.

## Architecture
- `src/Core/` — Interfaces, models, Result<T> monad
- `src/Engine/` — Reusable services (Entitlements, SaveSafety, Tasks, UI, QATelemetry)
- `src/Modules/` — Game-specific (StarRupture, SaveModifier)
- `src/Apps/ArcadiaTracker.App/` — WPF application (MVVM)
- `tests/` — xunit + FluentAssertions test projects

### QA Telemetry Pipeline (Engine Layer — Game-Agnostic)
- `src/Engine/GameCompanion.Engine.QATelemetry/` — Event-driven QA infrastructure
  - `Interfaces/` — IEventBus, IContextBuffer, ISaveFileWatcher, IRuleEngine,
    ICaptureSessionManager, IGameAdapter, IReportBuilder
  - `Models/` — TelemetryEvent, DetectionRule, CaptureSession, SaveStateSnapshot
  - `Services/` — EventBus, ContextBuffer, SaveFileWatcher, RuleEngine,
    CaptureSessionManager, ReportBuilder

### Game Adapters (Module Layer — Game-Specific)
- `StarRuptureGameAdapter` — implements IGameAdapter for Star Rupture
- `TelemetryBridgeService` — wires LogisticsAnomalyService results into the event bus

## Key Conventions
- Use `Result<T>` monad for all domain error handling (no thrown exceptions)
- Use `required init` properties for immutable models
- Use `Unit` type for void success results
- Capabilities are HMAC-SHA256 signed tokens, not boolean flags
- Admin access: separate namespace (`admin.*`), never piggybacks on paid features
- Non-discoverability: premium UI only appears after capability check passes
- Atomic file writes: temp file + File.Move(overwrite: true)
- **Defensive Coding**: See `.claude/skills/defensive_coding.skill.md` for null safety and DI patterns.

## Coding Standards
- File-scoped namespaces (`namespace X;`)
- `sealed class` for all concrete implementations (no accidental inheritance)
- XML doc comments (`/// <summary>`) on all public types and members
- Comments should explain WHY, not WHAT — describe intent, not mechanics
- No AI-generated boilerplate comments; every comment should add value a human
  developer would find useful
- Keep methods focused — one concern per method
- Prefer composition over inheritance
- Use collection expressions (`[]`) for empty/inline collections

### Naming Conventions
- PascalCase for types, methods, properties, events
- camelCase with underscore prefix for private fields (`_fieldName`)
- Suffix services with `Service`, view models with `ViewModel`
- Suffix interfaces with their concern (e.g., `IEventBus`, not `IEventBusInterface`)

### Event-Driven Design (QA Telemetry)
- No polling — use OS-level events (FileSystemWatcher) and in-process event bus
- Rolling buffers instead of unbounded storage
- Separation of concerns: capture, detection, and reporting are independent
- Detection rules are data-driven (JSON config), not hardcoded
- All telemetry services implement game-agnostic interfaces; game logic lives
  in adapters that implement `IGameAdapter`

### Support Report Export (QA Feedback → Developer Portal)
- `QAFeedbackExportService.GenerateSupportReportText()` — full scan → plain-text report
- `QAFeedbackExportService.GenerateSingleAnomalySupportText()` — single anomaly report
- Reports are plain text (no markdown) — web forms render `**` and ```` literally
- Format: ASCII sections (`===`, `---`, `[ ]`), grouped by severity, with investigation hints
- Flow: generate report → copy to clipboard → open browser to support portal → user pastes
- Support portal URL: `https://starrupture-game.com/support/`
- Per-anomaly "Report to Support" button on each anomaly card in the scanner tab
- Bulk "Report to Support" action bar appears after scan finds issues

## Build & Test
- Build: `dotnet build src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/ArcadiaTracker.App.csproj`
- Tests: xunit + FluentAssertions, test projects in `tests/`
- To verify code: review for compile errors, run existing test patterns
- QA Telemetry tests: `tests/GameCompanion.Engine.QATelemetry.Tests/`
- Support report tests: `tests/GameCompanion.Module.StarRupture.Tests/QAFeedbackExportServiceTests.cs`

## Security Rules
- All crypto uses System.Security.Cryptography (no custom implementations)
- HMAC comparison: always use CryptographicOperations.FixedTimeEquals()
- Encryption: AES-GCM only (not CBC)
- Key derivation: HKDF with domain-separated contexts
- Activation codes: HMAC-verified, one-time use
- Admin tokens: signed, encrypted at rest, time-bound (max 30 days)
- Never store secrets in plaintext

## Important Files
- Solution: `GameCompanionPlatform.sln` (root)
- DI wiring: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/App.xaml.cs`
- Main window: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/MainWindow.xaml.cs`
- Entitlements: `src/Engine/GameCompanion.Engine.Entitlements/`
- QA Telemetry: `src/Engine/GameCompanion.Engine.QATelemetry/`
- QA Feedback export: `src/Modules/GameCompanion.Module.StarRupture/Services/QAFeedbackExportService.cs`
- QA Feedback view: `src/Apps/ArcadiaTracker.App/ArcadiaTracker.App/Views/QAFeedbackView.xaml(.cs)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrizzwaldHouse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
