---
trigger: always_on
description: This file contains instructions for AI agents working on this project.
---

# Agent Instructions

This file contains instructions for AI agents working on this project.

## Knowledge Sources

### GitHub Copilot SDK Documentation

When working on this project, agents should consult and update their knowledge from the following official GitHub Copilot SDK resources:

### Primary SDK Documentation
- **Copilot SDK Repository**: https://github.com/github/copilot-sdk/
- **Copilot SDK .NET Implementation**: https://github.com/github/copilot-sdk/tree/main/dotnet

### Cookbook and Examples
- **Copilot SDK .NET Cookbook**: https://github.com/github/awesome-copilot/tree/main/cookbook/copilot-sdk/dotnet 
- **C# Specific Instructions**: https://github.com/github/awesome-copilot/blob/main/instructions/csharp.instructions.md

### SDK .NET Code Examples
- Client.cs: https://github.com/github/copilot-sdk/blob/main/dotnet/src/Client.cs
- Session.cs: https://github.com/github/copilot-sdk/blob/main/dotnet/src/Session.cs
- Types.cs: https://github.com/github/copilot-sdk/blob/main/dotnet/src/Types.cs
- Auto-Generated SessionEvents.cs: https://github.com/github/copilot-sdk/blob/main/dotnet/src/Generated/SessionEvents.cs

## Agent Workflow

Each time this project is revisited:

1. **Check for Updates**: Review the above knowledge sources for any updates to SDK patterns, best practices, or API changes
2. **Apply Latest Patterns**: Ensure the codebase follows current best practices from the Copilot SDK documentation
3. **Validate Implementation**: Verify that SDK usage aligns with official examples and recommendations
4. **Update Documentation**: If SDK changes affect this project, update README.md and code comments accordingly

## Project-Specific Context

This is a WinUI 3 desktop application that:
- Uses the GitHub Copilot SDK (v0.1.32) for chat functionality with 5-minute timeouts for complex operations
- Integrates with Windows taskbar via System.Windows.Forms.NotifyIcon
- Detects active Windows Explorer folders and applications for context
- Identifies WSL distributions when Windows Terminal shows Unix-style prompts
- Collects relevant environment variables (PYTHONPATH, NODE_ENV, DOTNET_ROOT, etc.)
- Maintains conversation history (last 10 messages) for context continuity
- Uses Windows Accessibility API (UI Automation) as fallback for enhanced context inference
- Shows "Thinking..." placeholder while processing requests
- Persists chat history in SQLite
- Renders assistant responses with Markdown (CommunityToolkit.WinUI.UI.Controls.Markdown)
- Targets .NET 11 Preview with partial trimming on ARM64 and x64
- Full Native AOT disabled due to WinUI 3 incompatibility (data binding, XAML resources)
- Includes Efficiency Mode utilities for process QoS and priority management
- Has a prepared ChatInputControl with file attachment, drag-drop, model selection (not yet wired into MainWindow)

### Context Inference Strategy

The application infers user intent/questions/problems using a **tiered optimization strategy**:

**Tier 1: Quick Detection (10-50ms)**
- Win32 Z-order walking for active focus
- Detects Explorer paths, Terminal windows, IDEs (VS Code, Visual Studio, Rider)
- Strong context = early exit to skip heavier operations

**Tier 2: Medium Detection (100-200ms)**
- File System Context: Open Explorer windows (Shell COM APIs)
- Application Context: Visible windows (Win32 EnumWindows)
- **Screenshot Capture**: Only when context is weak/ambiguous (Base64 JPEG, 1024px max)
- Runs in parallel with other Tier 2 operations

**Tier 3: Heavy Detection (500ms+)**
- Only for developer scenarios (project folders detected)
- WSL distributions
- Background services (Docker, databases, language servers)

**Always Included:**
- System environment (OS, user)

**Fallback Mechanism:**
- Windows Accessibility API (UI Automation) when Win32 insufficient
- Extracts focused UI element details, control hierarchy, and process info

**Screenshot Optimization:**
- Skipped when strong text context exists (Explorer path, Terminal, IDE)
- Only captured for ambiguous scenarios where visual context adds value
- Prevents unnecessary OCR/vision processing latency on LLM side

**Environment Variables Collected:**
- PATH (filtered to remove common Windows system paths)
- PYTHONPATH, NODE_ENV, JAVA_HOME, GOPATH, CARGO_HOME
- DOTNET_ROOT, DOTNET_CLI_HOME, DOTNET_INSTALL_DIR, MSBuildSDKsPath

**WSL Distribution Detection:**
- Detects Unix-style prompts in Windows Terminal (e.g., "user@hostname:~")
- Checks running WSL distributions via `wsl --list --verbose`
- Reports single running distro, or lists multiple for disambiguation

**Conversation History:**
- Last 10 messages (5 exchanges) included with each request
- Enables context continuity ("install podman" → "uninstall it")
- Model maintains awareness of previous actions and environments

## Key Technical Decisions

1. **System Tray Icon**: Uses official Microsoft System.Windows.Forms.NotifyIcon API instead of third-party libraries for maximum reliability
2. **Deployment**: Self-contained deployment required for unpackaged WinUI 3 applications
3. **SDK Integration**: Direct usage of GitHub.Copilot.SDK NuGet package with JSON-RPC communication to bundled Copilot CLI
4. **Authentication**: Authentication via GitHub CLI (`gh auth login`) required

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sirredbeard/ghcopilot-taskbar-gui](https://github.com/sirredbeard/ghcopilot-taskbar-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
