---
trigger: always_on
description: A WPF .NET 10 multi-terminal host for Claude Code (and other CLI tools). Runs multiple pseudo-terminal sessions in a tabbed/grid layout with full-text search, alert detection, git status, and session persistence.
---

# CodeShellManager

A WPF .NET 10 multi-terminal host for Claude Code (and other CLI tools). Runs multiple pseudo-terminal sessions in a tabbed/grid layout with full-text search, alert detection, git status, and session persistence.

## Build & Run

```bash
# From repo root
dotnet build src/CodeShellManager/CodeShellManager.csproj
dotnet run --project src/CodeShellManager/CodeShellManager.csproj

# Or open CodeShellManager.slnx in Visual Studio / Rider
```

**Requirements:** .NET 10 SDK, Windows 10/11 (uses ConPTY + WebView2)

### Visual Studio: Hot Reload disabled

`Properties/launchSettings.json` ships with `"hotReloadEnabled": false`, and the csproj sets `<MetadataUpdaterSupport>false</MetadataUpdaterSupport>` in Debug. Both are workarounds for a `System.ExecutionEngineException` that crashes the app on F5 under .NET 10.0.8 + VS 18 — `Microsoft.Extensions.DotNetDeltaApplier.dll` faults during its own startup before any managed code runs. Ctrl+F5 (Start Without Debugging) is unaffected either way. **Remove both when the runtime bug is fixed upstream.**

### Command-line flags

| Flag | Effect |
|---|---|
| `--clean` | Debug isolation mode — see below. |

**`--clean`** (parsed in `App.OnStartup`, exposed as `App.CleanStart`):
- `MainWindow.OnLoaded` skips the restore loop and clears the in-memory `SessionManager` — both sessions AND groups — so any new work in the run starts from a blank slate.
- `MainViewModel.SaveStateAsync` short-circuits — **nothing is written to `state.json`** for the entire run. Window bounds, layout changes, settings tweaks, and any sessions created during the clean run are all discarded on exit.
- The user's prior `state.json` survives the run untouched, so this is the safe way to test from a blank slate.

## Architecture

### Key layers

```
PTY (ConPTY) → PseudoTerminal → TerminalBridge → WebView2 (xterm.js)
                                     ↓
                           OutputIndexer → SQLite FTS5
                           AlertDetector → SessionViewModel.RaiseAlert()
```

- **PseudoTerminal** (`Terminal/PseudoTerminal.cs`): Windows ConPTY wrapper, P/Invoke only. Implements `IPseudoTerminal` (`Terminal/IPseudoTerminal.cs`) — the minimum surface needed by `RunInstance` (`DataReceived`, `Exited`, `ExitCode`, `Start`). Tests inject a fake via the `internal RunInstance(item, Func<IPseudoTerminal>)` constructor.
- **TerminalBridge** (`Terminal/TerminalBridge.cs`): Routes bytes between PTY and xterm.js via WebView2 messages. Surfaces accelerator keys (Ctrl-combos, F-keys, Esc) via `_webView.PreviewKeyDown` — the newer WPF WebView2 wrapper forwards accelerators through standard key events rather than a separate `CoreWebView2Controller.AcceleratorKeyPressed`. Bridge re-raises them as `AcceleratorKeyPressed` so `MainWindow.OnBridgeAcceleratorKey` can run global shortcuts even when the terminal has focus.
- **OutputIndexer** (`Terminal/OutputIndexer.cs`): Async channels → SQLite, strips ANSI
- **AlertDetector** (`Services/AlertDetector.cs`): Regex on raw PTY output, fires after 1.5s idle

### MVVM

- `MainViewModel` — sessions collection, layout mode, search state, alert count
- `SessionViewModel` — per-session state: alert, git info, waiting state, bridge/pty refs
- `MainWindow.xaml.cs` — heavy orchestration code-behind (sidebar building, layout, search handlers)

### Services

| Service | Purpose |
|---|---|
| `SessionManager` | CRUD for ShellSession models |
| `StateService` | JSON persistence → `%AppData%/CodeShellManager/state.json` |
| `SearchService` | SQLite FTS5 search of all terminal output; also owns the `project_notes` table |
| `ColorService` | FNV-1a hash of folder path → 12-color palette |
| `GitService` | Async `git branch --show-current` + `git status --porcelain` |
| `AlertDetector` | Pattern matching for Claude prompts/approvals |
| `CommandPresetsService` | Launch presets + in-session shortcuts |
| `ClaudeSessionService` | Detects `claude` invocations; finds last `--resume` session id under `~/.claude/projects/` |
| `UpdateService` | GitHub Releases version check; caches result for 24h at `%AppData%/CodeShellManager/update-cache.json` |
| `ImportExportService` | Read/write a full `AppState` to a JSON file (settings + sessions backup) |
| `ToastHelper` | Tray balloon notifications |
| `SessionRunner` | Per-session owner of `RunInstance` dictionary (run commands runtime) |
| `RunInstance` | One headless PTY-backed run with ANSI-stripped output buffer |
| `RunCommandTemplatesService` | Detects project type (dotnet/cargo/node/python/make) → seed run-command list |
| `WindowsTerminalProfileService` | Reads Windows Terminal `settings.json` from all install variants |
| `BuiltInTerminalSchemes` | Lookup table of WT default color schemes not present in user `settings.json` |
| `SchemeMapper` | WT scheme JSON → xterm.js theme JSON (renames `purple` → `magenta`, rewrites background as `rgba()` when opacity < 1) |
| `CursorShapeMapper` | WT `cursorShape` → xterm.js `cursorStyle` (+ optional forced blink) |
| `PaddingParser` | WT `padding` shorthand (1/2/4 comma ints) → CSS `Npx` shorthand |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umage-ai/CodeShellManager](https://github.com/umage-ai/CodeShellManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
