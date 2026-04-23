---
trigger: always_on
description: PowerShell completion module combining Tab completion (NativeAOT) + inline predictions (managed DLL) with generic learning and cross-session persistence. See README.md for features, installation, configuration, and usage.
---

# PSCue - Quick Reference for AI Agents

PowerShell completion module combining Tab completion (NativeAOT) + inline predictions (managed DLL) with generic learning and cross-session persistence. See README.md for features, installation, configuration, and usage.

## Architecture Internals
- **ArgumentCompleter** (`pscue-completer.exe`): NativeAOT exe, <10ms startup, computes completions locally with full dynamic arguments support
- **Module** (`PSCue.Module.dll`): Long-lived, implements `ICommandPredictor` + `IFeedbackProvider` (7.4+), provides PowerShell module functions
- **Learning System**:
  - **CommandHistory**: Ring buffer tracking last 100 commands
  - **CommandParser**: Parses commands into typed arguments (Verb, Flag, Parameter, ParameterValue, Standalone)
  - **ArgumentGraph**: Knowledge graph of command → arguments with frequency + recency scoring
    - **ArgumentSequences**: Tracks consecutive argument pairs for multi-word suggestions (up to 50 per command)
    - **ParameterStats**: Tracks parameters and their known values
    - **ParameterValuePairs**: Tracks bound parameter-value pairs
  - **ContextAnalyzer**: Detects command sequences and workflow patterns
  - **SequencePredictor**: ML-based n-gram prediction for next commands
  - **WorkflowLearner**: Learns command → next command transitions with timing data
  - **GenericPredictor**: Generates context-aware suggestions (values only after parameters, flags otherwise)
  - **Hybrid CommandPredictor**: Blends known completions + generic learning + ML predictions + workflow patterns
  - **PcdSubsequenceScorer**: fzf-style subsequence matching with boundary bonuses and match position output for highlighting (adapted from Wade's FuzzyScorer)
  - **PcdCompletionEngine**: Enhanced directory navigation with subsequence matching, frecency scoring, distance awareness
- **Persistence**:
  - **PersistenceManager**: SQLite-based cross-session storage
  - **Tables**: commands, arguments, co_occurrences, flag_combinations, argument_sequences, command_history, command_sequences, workflow_transitions, parameters, parameter_values
  - **Auto-save**: Every 5 minutes + on module unload
  - **Concurrent Access**: SQLite WAL mode handles multiple PowerShell sessions safely
  - **Additive Merging**: Frequencies summed, timestamps use max (most recent)

## Common Tasks
```bash
# Build (requires .NET 10.0 SDK — ArgumentCompleter targets net10.0, Module targets net9.0)
dotnet build src/PSCue.Module/ -c Release -f net9.0
dotnet publish src/PSCue.ArgumentCompleter/ -c Release -r win-x64

# Test
dotnet test test/PSCue.ArgumentCompleter.Tests/
dotnet test test/PSCue.Module.Tests/

# Run specific test groups
dotnet test --filter "FullyQualifiedName~Persistence"
dotnet test --filter "FullyQualifiedName~FeedbackProvider"
dotnet test --filter "FullyQualifiedName~CommandPredictor"
dotnet test --filter "FullyQualifiedName~SequencePredictor"
dotnet test --filter "FullyQualifiedName~WorkflowLearner"

# Install locally
./install-local.ps1

# Dev testing (isolated from production install)
./install-local.ps1 -Force -InstallPath D:\temp\PSCue-dev
# Then in a new PowerShell session:
#   $env:PSCUE_DATA_DIR = "D:\temp\PSCue-dev\data"
#   Import-Module "D:\temp\PSCue-dev\PSCue.psd1"
# Cleanup: Remove-Item -Recurse -Force D:\temp\PSCue-dev
```

## Key Technical Decisions
1. **NativeAOT for ArgumentCompleter**: <10ms startup required for Tab responsiveness
2. **Shared logic in PSCue.Shared**: NativeAOT exe can't be referenced by Module.dll at runtime
3. **ArgumentCompleter computes locally**: Tab completion always computes locally with full dynamic arguments. Fast enough (<50ms) and simpler.
4. **NestedModules in manifest**: Required for `IModuleAssemblyInitializer` to trigger
5. **Concurrent logging**: `FileShare.ReadWrite` + `AutoFlush` for multi-process debug logging
6. **PowerShell module functions**: Direct in-process access, no IPC overhead

## Performance Targets
- ArgumentCompleter startup: <10ms
- Total Tab completion: <50ms
- Module function calls: <5ms
- Database queries: <10ms
- PCD tab completion: <10ms
- PCD best-match navigation: <50ms

## When Adding Features
- Put shared completion logic in `PSCue.Shared`
- DynamicArguments (git branches, scoop packages) are computed locally by ArgumentCompleter
- Learning happens automatically via `FeedbackProvider` - no manual tracking needed
- When adding support for new commands, add the completer registration in `module/PSCue.psm1` as well
- **Command aliases**: Use `Alias` property on `Command` class, include in tooltip like `"Create a new tab (alias: nt)"`
- **Parameter aliases**: Use `Alias` property on `CommandParameter` class, include in tooltip like `"Only list directories (-d)"`
  - **IMPORTANT**: Do NOT create separate parameter entries for short and long forms (e.g., `-d` and `--diff` as separate parameters)
  - Instead, define the long form with the short form as an alias: `new("--diff", "Compare files (-d)") { Alias = "-d" }`
  - This prevents duplicate suggestions and keeps the completion list clean

## Testing Patterns
```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucaspimentel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
