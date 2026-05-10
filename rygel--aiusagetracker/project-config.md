---
trigger: always_on
description: This document provides essential information for agentic coding assistants working on this .NET 8.0 WPF application.
---

# AI Usage Tracker - Monitor Guidelines

This document provides essential information for agentic coding assistants working on this .NET 8.0 WPF application.

## Critical Rules

### NEVER Create Releases Without Explicit Permission
- **I MUST NEVER** create git tags or releases without your explicit instruction
- **I MUST NEVER** initiate CI/CD release workflows without your permission
- **I MUST NEVER** create beta or stable releases on my own
- **You must explicitly tell me** when to create a release
- **I will wait for your command** before any release-related action

### Provider Visibility Requirement
- **ALL configured providers MUST be visible** in the UI at all times
- **NEVER** filter out providers just because they have `IsAvailable=false`
- Providers with missing API keys should show as "Not Available" or "Configure Provider"
- **DO NOT** wait for fresh data before showing providers - display cached data immediately
- **DO NOT** show only a subset of providers (e.g., only antigravity) on startup
- The UI must show provider cards immediately, even if data is stale or unavailable

### Lean Code Requirement
- **The goal is to keep as little source code as necessary** to deliver required functionality across these applications.
- Prefer deleting redundant layers/wrappers over adding new abstraction when behavior can remain clear and testable.
- New code must justify its existence; avoid duplicate logic paths and unnecessary fallback branches.
- Interfaces are only for types that are mocked in tests. Single-implementation interfaces with no test mocking are unnecessary indirection — use the concrete class directly.

### Database Architecture
- **Never duplicate provider metadata into DB tables.** The database stores raw values only; provider definitions (ProviderDefinition class) are the authority for interpretation.
- Provider definitions are immutable per provider ID. If semantics change, a new provider class with a new ID is created.
- When encountering data integrity concerns, do NOT suggest adding columns. Keep the schema minimal.

### Take Full Ownership
- Do not give hedged answers like "this will work IF the API returns X." Investigate the actual data (database, logs, live endpoints) and report what IS happening.
- Give definitive "working" or "broken" with evidence. If something is broken, identify the root cause and fix it.

### Development Workflow

- **Never push directly to `main` or `develop`**: All changes MUST go through feature/fix branches with Pull Requests targeting `develop`. Beta releases target `develop`; only stable releases target `main`.
- **Never force push to `main` without explicit user permission**: If you need to force push to main, ALWAYS ask for confirmation first.
- **Atomic Commits**: Keep commits focused and logically grouped.
- **CI/CD Compliance**: Ensure that any UI changes or tests are compatible with the headless CI environment.
- **No Icons in PRs**: When creating pull requests, do not use emojis or icons in the title or body.
- **PR Management**: ALWAYS modify existing PRs (`gh pr edit`) instead of closing and creating new ones. Use `gh pr edit --base <branch>` to change the target branch. Closing and recreating wastes CI runs and loses conversation context.

### Verify Before Acting (CRITICAL)

Before implementing ANY change, verify the current state first:

1. **Before version bumps**: Run `git tag -l "v*" --sort=-v:refname | head -5` to check the latest released tag. Never trust file contents alone — tags may have advanced beyond what's in the files.
2. **Before branch operations**: Run `git log --oneline origin/<branch> -5` to see what's actually on the remote.
3. **Before releases**: Check tags AND file versions AND changelog to ensure consistency.
4. **Before pushing**: Run validation scripts locally:
   - `bash scripts/validate-release-consistency.sh "<version>"` for version bumps
   - `dotnet build` for compilation
   - `dotnet test` for test suite
5. **Before creating PRs**: Check the correct target branch (`develop` for betas, `main` for stable).

**Why**: Pushing without verification wastes 5-10 minutes of CI time per failed round-trip and blocks the user.

## Project Structure

- **AIUsageTracker.Core**: Domain models, interfaces, and business logic (PCL)
- **AIUsageTracker.Infrastructure**: External providers, data access, configuration
- **AIUsageTracker.UI.Slim**: Lightweight WPF desktop application with compact UI
- **AIUsageTracker.Monitor**: Background service that collects provider usage data via HTTP API
- **AIUsageTracker.CLI**: Console interface (cross-platform)
- **AIUsageTracker.Web**: ASP.NET Core Razor Pages web application for viewing data
- **AIUsageTracker.Tests**: xUnit unit tests with Moq mocking

## Build & Test Commands

### Building
```bash
# Build entire solution
dotnet build AIUsageTracker.sln --configuration Debug

# Build specific project
dotnet build AIUsageTracker.UI.Slim/AIUsageTracker.UI.Slim.csproj

# Restore dependencies
dotnet restore
```

### Testing
```bash
# Run all unit tests
dotnet test AIUsageTracker.Tests/AIUsageTracker.Tests.csproj --configuration Debug

# Run all tests (no rebuild)
dotnet test --no-build --verbosity normal


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rygel/AIUsageTracker](https://github.com/rygel/AIUsageTracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
