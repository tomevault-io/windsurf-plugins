---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Start Warning

**IMPORTANT: At the beginning of every new conversation, you MUST display the following warning to the user before doing anything else:**

> **WARNING: This codebase has ongoing work that affects builds and contributions.**
>
> - The full build requires internal NuGet packages (`Microsoft.Agents.*`) that are **not publicly accessible**. Builds will fail without internal feed access.
> - Migration from the internal Dataverse SDK to PAC CLI is **in progress** to enable external builds. This work is not yet complete.
> - The CI pipeline (`.github/workflows/ci.yml`) currently builds and tests **only the .NET language server**, not the TypeScript extension.
> - External contributors can modify TypeScript code and documentation, but **cannot validate .NET changes locally** without the internal feed.
>
> Proceed with awareness of these constraints.

## Project Overview

This is the Copilot Studio Extension for Visual Studio Code - a hybrid TypeScript/C# project that provides a full-featured VS Code extension for editing Microsoft Copilot Studio agents. The extension enables developers to clone agents locally, edit components (topics, triggers, actions, knowledge sources) in YAML format, and sync changes bidirectionally with cloud environments. IntelliSense is powered by a Language Server Protocol (LSP) backend written in C#.

**Current State:** Generally Available (GA)

**Documentation:** https://learn.microsoft.com/en-us/microsoft-copilot-studio/visual-studio-code-extension-overview

**Important:** The full build currently requires internal NuGet packages (`Microsoft.Agents.*`) and is not externally reproducible. Migration to PAC CLI is underway to enable external builds soon.

## Build and Test Commands

### TypeScript / VS Code Extension

The `package.json` is located at `src/vscode-extensions/microsoft-powerplatformlang-extension/package.json`. All npm commands must be run from that directory.

```bash
# Change to the extension directory first
cd src/vscode-extensions/microsoft-powerplatformlang-extension

# Install dependencies
npm install

# The parent src/vscode-extensions/.npmrc sets omit-lockfile-registry-resolved=true
# so lockfile updates omit registry/feed-specific resolved URLs. Do not add
# registry or feed URLs to committed .npmrc or package-lock.json files;
# registry/feed/auth configuration belongs in user npm config or CI restore-time
# configuration.

# Build everything (LSP + type check + esbuild bundle)
npm run compile

# Build only the C# language server for current platform
npm run buildLsp

# Watch LSP for changes (rebuilds on save)
npm run watchLsp

# Type check only (no emit)
npm run check-types

# Lint
npm run lint

# Run VS Code extension tests
npm test

# Package VSIX (win32-x64, pre-release)
npm run package

# Watch TypeScript compilation
npm run watch
```

### .NET / Language Server

```bash
# Build the language server solution
dotnet build src/LanguageServers/PowerPlatformLS/PowerPlatformLS.sln

# Run all .NET unit tests
dotnet test src/LanguageServers/PowerPlatformLS/PowerPlatformLS.sln

# Run a single test by filter
dotnet test --filter "FullyQualifiedName~Namespace.ClassName.MethodName" src/LanguageServers/PowerPlatformLS/PowerPlatformLS.sln

# Create NuGet packages
dotnet pack --no-build --no-restore -c debug src/LanguageServers/PowerPlatformLS/PowerPlatformLS.sln
```

### LSP Journal Tests

```bash
# Run one journal test
dotnet run --project src/LanguageServers/PowerPlatformLS/Tools/LspJournalCli -- lifecycle

# Run all journal tests
dotnet run --project src/LanguageServers/PowerPlatformLS/Tools/LspJournalCli -- --all

# Accept pending journal changes
dotnet run --project src/LanguageServers/PowerPlatformLS/Tools/LspJournalCli -- accept lifecycle

# List pending journal changes
dotnet run --project src/LanguageServers/PowerPlatformLS/Tools/LspJournalCli -- pending
```

### Development Workflow

1. Open the repo in VS Code
2. Press **F5** to launch the Extension Development Host
3. The extension activates and starts the language server automatically
4. For .NET language server debugging, attach to the `LanguageServerHost` process

### Prerequisites

- .NET 10.0 SDK (specified in `global.json`)
- Node.js 22 LTS
- VS Code 1.96.0+

## Architecture

### High-Level Communication Flow

```
VS Code Extension (TypeScript)
    |
    | JSON-RPC over named pipe (Windows) / Unix socket (macOS/Linux)
    v
LanguageServerHost.exe (C# / .NET 10)
    |
    | DI-based handler routing
    v
Language Implementations (MCS / PowerFx / YAML)
    |
    | Pluggable rules
    v
Completions, Diagnostics, Semantic Tokens, Go-to-Definition
```

### TypeScript Extension (`src/vscode-extensions/microsoft-powerplatformlang-extension/`)

**Entry Point:** `client/src/extension.ts`

Initialization sequence:
1. Generate session ID (UUID) for telemetry
2. Initialize logger with session context
3. Register auth commands (signIn, resetAccount, reportIssue) - no LSP dependency
4. Initialize and start LSP client (blocking - all downstream features depend on this)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-copilotstudio](https://github.com/microsoft/vscode-copilotstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
