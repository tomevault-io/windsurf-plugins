---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tooling constraints

- The `gh` CLI is available and can be used for GitHub operations (creating issues, PRs, etc.).

## Code review

When asked to do a review without further context, review the **uncommitted local changes** (`git diff` / `git diff --staged`). Use `git diff <base>...HEAD` only if the user explicitly asks to review a branch or a specific range of commits.

## Collaboration style

The primary role of the agent in this project is **code reviewer and rubber duck**: discuss design choices, raise issues, flag risks, reason through problems together with the developer.

Direct code interventions are limited to **repetitive or tedious tasks** (renaming, formatting, adding boilerplate, mechanical refactors, git operations). The developer writes the actual logic and makes all architectural decisions.

When in doubt, discuss first and act only when explicitly asked.

## Release flow

1. Merge all feature/fix branches into `develop` via PR.
2. Create `release/X.Y.Z` branch from `develop`:
   ```bash
   git checkout origin/develop -b release/X.Y.Z
   ```
3. Bump the version in **two files** (use the `bump-version` skill):
   - `src/TombLauncher/TombLauncher.csproj` — `<Version>`
   - `deploy/TombLauncher.pupnet.conf` — `AppVersionRelease`
4. Commit: `chore(release): bump version to X.Y.Z`
5. Tag the bump commit **before** pushing:
   ```bash
   git tag vX.Y.Z
   ```
6. Push branch and tag, then open PR `release/X.Y.Z` → `master`.
7. The CI pipeline triggers on the tag and builds the release artifacts.

Version scheme: `MAJOR.MINOR.PATCH` — bump MINOR for new features, PATCH for bugfix-only releases.

## Commit messages

Use **Conventional Commits** format. The subject line follows `type(scope): description (#issue)`.

The body should list what was added or changed, grouped by area:

```
feat(scope): short description (#123)

- New class/file: brief purpose
- Modified X to do Y
- Localization: added KEY to all N language files
- ...
```

Keep the subject under 72 characters. The body uses short bullet points — no prose.

## Commands

```bash
# Build
dotnet build TombLauncher.slnx

# Run
dotnet run --project src/TombLauncher

# Test
dotnet test

# Run a single test class
dotnet test --filter "FullyQualifiedName~TombLauncher.Tests.ClassName"
```

No linting/formatting tooling is configured. The project uses Rider/ReSharper conventions. All projects have `<Nullable>enable</Nullable>` and `<ImplicitUsings>enable</ImplicitUsings>`.

## Architecture

### Project layout

| Project | Role |
|---------|------|
| `TombLauncher` | Main WinExe — Views, ViewModels, Services, DI bootstrap |
| `TombLauncher.Contracts` | Shared enums, interfaces, and contracts (no external dependencies) |
| `TombLauncher.Ai` | Optional AI/RAG subsystem — LLM troubleshooting, vector search, Ollama/LM Studio backends |
| `TombLauncher.Controls` | Reusable Avalonia UI controls |
| `TombLauncher.Core` | Platform-agnostic business logic — DTOs, launchers, savegame parsing, installers |
| `TombLauncher.Data` | EF Core + SQLite — entities, migrations, data services |
| `TombLauncher.Localization` | AXAML resource dictionaries for en-US and it-IT |
| `TombLauncher.Patchers` | Game binary patching — Gameflow parsing, widescreen patching, TRX native patching |
| `TombLauncher.Tests` | xUnit tests with NSubstitute for mocking |

Dependency direction: `TombLauncher` → `Controls / Core / Data / Localization` → `Contracts`. `TombLauncher.Core` has no Avalonia dependency and can be tested in isolation.

### DI and startup

`App.axaml.cs` wires up the DI container in `InitializeServices()`. The static `Ioc.Default` (CommunityToolkit.Mvvm) is the root container. Key extension methods:

- `AddViewModels()` — registers all page ViewModels with appropriate lifetimes (singleton for long-lived pages, scoped/transient for the rest)
- `AddPageServices()` — registers application-layer services
- `AddDatabaseAccess(config, appDataDir)` — registers `TombLauncherDbContext` (SQLite), repositories, and data services
- `AddTombLauncherMappings()` — registers all manual mapper singletons
- `AddDownloaders()` — registers the three community-site downloaders

EF Core migrations run automatically at startup via `dbContext.Database.MigrateAsync()`.

### MVVM pattern

**ViewLocator** (`src/TombLauncher/ViewLocator.cs`) resolves Views from ViewModels by convention: `FooViewModel` → `FooView` (UserControl), sets `DataContext` automatically.

**ViewModelBase** → **PageViewModel** is the ViewModel hierarchy. `PageViewModel` provides:
- `OnNavigatedTo(parameter)` / `OnNavigatingFrom()` lifecycle hooks
- `BusyScope()` — disposable that sets the busy indicator while active
- `SaveCmd` / `CancelCmd` base commands
- `TopBarCommands` — `ObservableCollection<ITopBarCommand>` that the shell renders in the top bar; pages add commands here without any coupling to the shell

Settings page ViewModels extend **`SettingsSectionViewModelBase`**, which implements `IChangeTracking`. Properties decorated with `[IgnoreChanges]` are excluded from dirty-state detection; `IsChanged` drives Save/Cancel button availability.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALDamico/TombLauncher](https://github.com/ALDamico/TombLauncher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
