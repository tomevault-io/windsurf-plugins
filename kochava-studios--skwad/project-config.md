---
trigger: always_on
description: This document provides context for AI agents working on the Skwad codebase.
---

# Skwad - Agent Development Guide

This document provides context for AI agents working on the Skwad codebase.

## Project Overview

Skwad is a macOS SwiftUI application that manages multiple AI coding agents, each running in an embedded terminal. It supports two terminal engines (Ghostty and SwiftTerm), agent-to-agent communication via MCP, and git worktree integration.

## Tech Stack

- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI (macOS 26+)
- **Terminal Engines**:
  - Ghostty (libghostty) - GPU-accelerated, default
  - SwiftTerm - fallback option
- **MCP Server**: Hummingbird HTTP framework
- **Persistence**: @AppStorage (UserDefaults) + Codable JSON
- **Build**: Xcode project + Swift Package Manager

## Project Structure

```
Skwad/
├── Models/
│   ├── Agent.swift              # Agent data model with runtime state
│   ├── AgentManager.swift       # Central agent lifecycle management
│   └── AppSettings.swift        # App settings with @AppStorage
├── Views/
│   ├── ContentView.swift        # Main layout with sidebar + terminal
│   ├── Sidebar/
│   │   ├── SidebarView.swift    # Agent list with drag-drop
│   │   └── AgentSheet.swift     # New/edit agent dialog
│   ├── Terminal/
│   │   ├── AgentTerminalView.swift   # Terminal wrapper
│   │   ├── GhosttyHostView.swift     # Ghostty NSViewRepresentable
│   │   └── TerminalHostView.swift    # SwiftTerm NSViewRepresentable
│   ├── Git/
│   │   ├── GitPanelView.swift   # Sliding git status panel
│   │   ├── DiffView.swift       # Syntax-highlighted diff display
│   │   └── CommitSheet.swift    # Commit dialog
│   └── Settings/
│       └── SettingsView.swift   # Settings window
├── Git/
│   ├── GitCLI.swift             # Low-level git command runner (with timeout)
│   ├── GitRepository.swift      # High-level git operations
│   ├── GitWorktreeManager.swift # Worktree discovery and creation
│   ├── GitFileWatcher.swift     # FSEvents file monitoring
│   └── GitTypes.swift           # FileStatus, DiffLine, etc.
├── MCP/
│   ├── AgentCoordinator.swift    # Actor managing messages and agent data
│   ├── MCPServer.swift          # Hummingbird HTTP server + hook event handler
│   ├── MCPSessionManager.swift  # MCP session tracking
│   ├── MCPToolHandler.swift     # Tool execution
│   └── MCPTypes.swift           # Message, AgentInfo structs
├── Services/
│   ├── NotificationService.swift       # macOS desktop notifications
│   ├── RepoDiscoveryService.swift      # Background repo discovery
│   ├── TerminalAdapter.swift           # Protocol + Ghostty/SwiftTerm adapters
│   ├── TerminalCommandBuilder.swift    # Agent command construction
│   └── TerminalSessionController.swift # Terminal session lifecycle + status state machine
├── GhosttyTerminal/             # Ghostty integration (libghostty wrappers)
└── SkwadApp.swift               # App entry point
```

## Architecture

### Terminal Management
- All terminals are kept alive in a ZStack with opacity toggle (not recreated on switch)
- This preserves terminal state/history when switching between agents
- `restartToken` on Agent model forces terminal recreation on restart while keeping same ID
- Focus is managed via `window?.makeFirstResponder()` in updateNSView

### Terminal Engines
- **Ghostty** (default): Uses libghostty C API via Swift wrappers
  - `GhosttyAppManager` - singleton managing Ghostty app instance
  - `GhosttyHostView` - NSViewRepresentable wrapper
  - Reads user's `~/.config/ghostty/config` for styling
- **SwiftTerm**: Fallback option
  - `TerminalHostView` - NSViewRepresentable wrapper
  - `ActivityDetectingTerminalView` - subclass for activity detection

### Activity Detection
- `TerminalSessionController` owns the status state machine per agent
- `ActivityTracking` bitfield controls which sources trigger status changes:
  - `.all` (default): terminal output + user input drive running/idle
  - `.userInput`: hook-managed agents — only user input is tracked locally, hooks handle running/idle
  - `.none`: shell agents — no status tracking
- When hooks are active (`sessionId` set + agent supports hooks), terminal output is ignored as a status source
- Status colors: orange=Working, green=Idle, red=Blocked, red=Error
- **Blocked status**: set via hook when agent needs user attention (e.g. permission prompt). Unblocked by Return (→ running) or Escape (→ idle) keypress only
- **Input protection**: user keypresses activate a 10s guard that blocks automatic text injection (`injectText`), preventing message delivery while user is typing. Messages stay in MCP queue and are delivered on next idle or when protection expires
- `onUserInput` callback passes `UInt16` keyCode (macOS keyCode from Ghostty, mapped from raw bytes for SwiftTerm)
- When idle, checks for unread MCP messages

### MCP Communication
- `AgentCoordinator` (actor) manages message queue and agent queries
- `AgentDataProvider` protocol bridges MainActor-isolated AgentManager safely
- Server starts AFTER AgentManager is set to avoid race conditions
- Registration prompt injected ~3s after terminal starts (if MCP enabled)
- Communication tools: `register-agent`, `list-agents`, `send-message`, `check-messages`, `broadcast-message`
- Management tools: `list-repos`, `list-worktrees`, `create-agent`

### Git Integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kochava-Studios/skwad](https://github.com/Kochava-Studios/skwad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
