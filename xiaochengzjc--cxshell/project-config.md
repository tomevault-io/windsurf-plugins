---
trigger: always_on
description: CxShell is a single-project Avalonia desktop terminal and remote session client targeting .NET 10. The application entry points are `Program.cs`, `App.axaml`, and `App.axaml.cs`.
---

# Repository Guidelines

## Project Structure & Module Organization

CxShell is a single-project Avalonia desktop terminal and remote session client targeting .NET 10. The application entry points are `Program.cs`, `App.axaml`, and `App.axaml.cs`.

- `Views/`: Avalonia `.axaml` views and their code-behind files.
- `ViewModels/`: MVVM presentation logic, commands, and observable state.
- `Models/`: session, SFTP, and monitoring data objects.
- `Services/`: SSH/SFTP connections, persistence, monitoring, and Linux parsing.
- `Terminal/`: terminal buffer, cells, ANSI parsing, and color handling.
- `Controls/` and `Converters/`: reusable UI controls and binding converters.
- `Assets/`: resources embedded through `CxShell.csproj`.

`AtomUI/` is ignored reference source and explicitly excluded from compilation. Do not treat `bin/` or `obj/` as source.

## Build, Test, and Development Commands

Run commands from the repository root:

```powershell
dotnet restore
dotnet build CxShell.csproj
dotnet run --project CxShell.csproj
dotnet format CxShell.csproj
```

`restore` downloads NuGet dependencies, `build` compiles the app, `run` launches the desktop client, and `format` applies standard .NET formatting.

## Coding Style & Naming Conventions

Use four-space indentation in C# and follow existing file-scoped namespace style. Keep nullable reference types enabled. Use PascalCase for types, public members, views, and view models; use camelCase for parameters and `_camelCase` for private fields. Pair each view with matching names such as `TerminalView.axaml` and `TerminalView.axaml.cs`. Prefer CommunityToolkit.Mvvm attributes (`[ObservableProperty]`, `[RelayCommand]`) over repetitive boilerplate.

## Testing Guidelines

No automated test project currently exists. Before submitting changes, run `dotnet build CxShell.csproj` and manually exercise affected SSH, SFTP, terminal, or monitoring workflows. New test projects should use names such as `CxShell.Tests`, with test files named `<ClassName>Tests.cs`; run them with `dotnet test`.

## Terminal Interaction Requirements

`Controls/TerminalControl.cs` owns terminal-grid interaction. Preserve left-button forward, reverse, and multi-line highlighted selection. Right-click opens the `Popup` in `Views/TerminalView.axaml`, containing Copy and Paste; disable Copy without a selection. Copy writes selected text to the system clipboard. Paste normalizes line endings and sends text through `InputReceived` to the SSH shell at its current cursor. Use Avalonia 12 clipboard extensions from `Avalonia.Input.Platform` (`SetTextAsync`, `TryGetTextAsync`) and catch platform failures. Follow the AtomUI 6.0 manual and local `AtomUI/` reference source for menu styling.

Terminal scrollback is stored by `TerminalBuffer`; `TerminalControl` renders a scroll-offset viewport over that history. Mouse wheel up/down must move through scrollback without sending data to the SSH shell. Hide the cursor while viewing history, preserve selection/copy against the visible viewport, and return to the bottom when the user types, pastes, or sends a key command. If output arrives while the user is viewing history, keep the historical viewport stable instead of forcing it to the bottom.

Unexpected SSH or remote-shell closure must trigger automatic terminal reconnection every five seconds. Successful reconnection stops retries; explicitly closing a tab or disconnecting must cancel them. Treat a zero-byte blocking shell read as remote EOF.

## Protocol Implementation Notes

Implement protocols incrementally in the agreed order: SSH, SFTP, FTP, TELNET, RLOGIN, SERIAL, then RDP. `Services/IFileTransferService.cs` is the shared backend contract for file browsers; SFTP uses `SftpService`, and plain FTP uses `FtpService` with FluentFTP. `SftpViewModel` selects the service from `SessionInfo.Protocol`, so keep uploads, downloads, rename, delete, and mkdir behavior protocol-neutral. SFTP startup options on `SessionInfo` include local start folder, remote start folder, and custom server command; local/remote folders are active, while SSH.NET still opens the standard SFTP subsystem unless a lower-level custom server implementation is added.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaochengzjc/CxShell](https://github.com/xiaochengzjc/CxShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
