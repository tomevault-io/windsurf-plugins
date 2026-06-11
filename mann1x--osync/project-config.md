---
trigger: always_on
description: This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.
---

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

osync is a CLI tool for managing Ollama models across local and remote servers. Written in C# targeting .NET 8 (net8.0-windows10.0.22621.0).

## Build Commands

```bash
dotnet build                    # Build debug
dotnet build -c Release         # Build release
dotnet publish -c Release       # Create standalone executables
dotnet test                     # Run all tests
dotnet test --filter DisplayName~CopyCommands  # Run specific feature tests
```

## Architecture

### Command Structure

The application uses PowerArgs for CLI parsing. All commands are defined as action methods in `OsyncProgram` class (Program.cs) with corresponding `*Args` classes in CommandArguments.cs:

- **Copy (cp)** - Model transfers with bandwidth throttling support, memory-buffered streaming for remote transfers
- **List (ls)** - Pattern matching with wildcards, multiple sort modes
- **Remove (rm/delete/del)** - Pattern-based deletion
- **Rename (mv/ren)** - Safe rename via copy → verify → delete workflow
- **Update** - Update models to latest versions
- **Pull** - Download from registry (includes HuggingFace support)
- **Show** - Display model metadata
- **Run/Chat** - Interactive chat with extended thinking mode support
- **Ps** - List loaded models in memory
- **Load/Unload** - VRAM management
- **Manage** - Full-screen TUI using Terminal.Gui
- **Qc** - Quantization comparison with test suites
- **QcView** - Quantization comparison test results viewer

### Key Files

- `Program.cs` - Entry point, CLI routing, all command action methods
- `CommandArguments.cs` - All argument classes for commands
- `ChatSession.cs` - Interactive chat session management
- `ManageCommand.cs` - TUI implementation with themes
- `QcCommand.cs` - Quantization comparison implementation
- `QcViewCommand.cs` - QC results viewer with PDF/HTML/Markdown output generation
- `QcModels.cs` - Data models for QC results (JudgmentResult, QuantResult, etc.)
- `QcScoring.cs` - Score calculation logic for QC results
- `OllamaModels.cs` - Ollama API data models
- `ThrottledStream.cs` - Bandwidth limiting for transfers
- `CloudProviders/` - Cloud AI provider implementations for judge models

### Test Structure

BDD tests using SpecFlow + xUnit in `osync.Tests/`:
- Feature files in `Features/` directory
- Step definitions in `StepDefinitions/`
- Test infrastructure in `Infrastructure/` (OsyncRunner, TestConfiguration)

### Key Technical Details

- Static HttpClient with 1-day timeout for large model transfers
- Model names auto-append `:latest` tag when not specified
- Tab completion support via PowerArgs with local models as source

### Dependencies

Core: PowerArgs (CLI), Spectre.Console (formatting), Terminal.Gui (TUI), TqdmSharp (progress bars)
PDF: iText7 (AGPL-3.0 licensed) - used for PDF report generation in QcView
AI SDKs: Anthropic, OpenAI, Azure.AI.OpenAI - for cloud judge providers
Test: xUnit, SpecFlow, FluentAssertions

### Test Data

QC test result files for testing qcview output are located in `d:\install\osync\test\`

## Important Guidelines

### Working Directory and File Management

- **Never save generated files in the repository directory** - The repository will be pushed to GitHub, so do not create test files, output files, logs, or any generated content in the osync source directory
- **Use `.\osync\bin` as the default working directory** for:
  - Running/testing the osync executable
  - Saving test output files (JSON results, logs, etc.)
  - Creating temporary files
  - Executing benchmark or QC test runs
- **Example paths:**
  - Run osync: `.\osync\bin\Debug\net8.0-windows10.0.22621.0\osync.exe`
  - Save test results: `.\osync\bin\test-results.json`
  - Log files: `.\osync\bin\test_log.txt`

### Version and Changelog

- **Never change the program version** unless explicitly asked by the user
- **Changelog is in README.md** - Update the Changelog section in README.md, not a separate CHANGELOG.md file
- **Do not bump version when updating changelog** - Only add entries under the current version section
- **Always check the current version** before updating the changelog - check `Program.cs` for the version constant or recent git tags
- **Check GitHub releases** at https://github.com/mann1x/osync/releases to see what's already been released before adding changelog entries
- When updating changelog, create a new version section if needed - don't add new features under an already-released version

### Spectre.Console Quirks

- **File access checks with confirmation prompts** must happen BEFORE starting a Progress display
- `AnsiConsole.Confirm()` cannot run inside a Progress context - causes "concurrent interactive functions" error
- When adding file output with overwrite confirmation, check access before `AnsiConsole.Progress().StartAsync()`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mann1x/osync](https://github.com/mann1x/osync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
