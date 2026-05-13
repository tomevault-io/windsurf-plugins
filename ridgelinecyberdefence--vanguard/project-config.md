---
trigger: always_on
description: Cross-platform (Windows/Linux) enterprise DFIR toolkit for connected, isolated, and air-gapped environments. Go core with bubbletea TUI, PowerShell/Bash modules, Python3 analysis engine. Velociraptor is the primary IR capability. Open-sourced under RidgeLine Cyber.
---

# VanGuard DFIR Toolkit

## Project Overview

Cross-platform (Windows/Linux) enterprise DFIR toolkit for connected, isolated, and air-gapped environments. Go core with bubbletea TUI, PowerShell/Bash modules, Python3 analysis engine. Velociraptor is the primary IR capability. Open-sourced under RidgeLine Cyber.

## Language Stack

- Core Launcher: Go (bubbletea TUI, single binary)
- Windows Modules: PowerShell 5.1+
- Linux Modules: Bash + Python3
- Analysis Engine: Python3 (Volatility3)

## Repository Structure

    cmd/vanguard/
        main.go                         Entry point, platform detection, privilege check
    internal/
        config/
            config.go                   YAML config loading, validation, defaults
        logging/
            logger.go                   Structured file+stderr logger with rotation
        tui/
            menu.go                     Bubbletea TUI engine, sidebar+content layout
            styles.go                   Color schemes, lipgloss styles, dark blue theme
        velociraptor/
            manager.go                  Binary download, SHA256 verify, init, server lifecycle
            deploy.go                   Client deployment (WinRM/SSH/PSExec/USB)
            artifacts.go                Artifact collection engine
            usecases.go                 Use case runner
        tools/
            manager.go                  Tool registry, download, verification, status
        case/
            manager.go                  SQLite case management, evidence chain
        modules/
            executor.go                 Module orchestrator, ToolResult struct
        network/
            ssh.go                      SSH client
            winrm.go                    WinRM client
            psexec.go                   PSExec wrapper
        updater/
            updater.go                  Auto-update from GitHub releases + git
        output/
            engine.go                   CSV/HTML/TXT output generation
            templates.go                Jinja2-style HTML report rendering
    modules/
        windows/                        PowerShell IR modules (Verb-Noun.ps1)
        linux/                          Bash IR modules (snake_case.sh)
    config/
        vanguard.yaml                   Master configuration
        targets.yaml                    Remote target definitions
        velociraptor/                   Auto-generated Velociraptor configs
    usecases/                           YAML use case definitions (UC-WIN/LNX/XP-xxx)
    velociraptor/artifacts/             Custom VQL artifacts (VG.Windows.*, VG.Linux.*)
    rules/                              Sigma/YARA/Hayabusa detection rules
    templates/                          HTML report templates (.j2)
    output/                             Runtime: output/{case_id}/{memory,disk,triage,velociraptor,reports}
    logs/                               Runtime logs
    docs/                               Documentation
    tests/                              Test files

## Code Conventions

### Go
- Standard library preferred. Minimal external dependencies.
- internal/ for all private packages. No exported packages outside cmd/.
- All errors wrapped with context: fmt.Errorf("operation: %w", err)
- Structured error handling: ToolResult struct for all external tool calls.
- No global state. Pass config/context explicitly.
- ALL exec.Command calls must run in tea.Cmd goroutines — never block the TUI.
- View() must be a pure render function — no I/O, no filesystem access, no computation.
- Use strings.Builder for string assembly in View().

### PowerShell
- Verb-Noun function naming (Invoke-MemoryCapture, Export-Results).
- [CmdletBinding()] on all functions.
- Error handling via try/catch with structured output objects.

### Bash
- snake_case function and file naming.
- set -euo pipefail in all scripts.
- Structured logging to stderr, data to stdout.

### Python
- Type hints on all functions.
- Dataclasses for structured data.
- pathlib.Path for all file paths.

### General
- All paths relative to VanGuard root. Never hardcode absolute paths.
- All tools/data must reside within VanGuard directory structure.
- Output always to output/{case_id}/ subdirectories.
- Brand name: RidgeLine Cyber (not RidgeLine).

## Key Dependencies
- github.com/charmbracelet/bubbletea — TUI framework
- github.com/charmbracelet/lipgloss — TUI styling
- github.com/charmbracelet/bubbles — TUI components (textinput, table, spinner)
- gopkg.in/yaml.v3 — YAML config
- github.com/mattn/go-sqlite3 — SQLite case DB (CGO required, build with CGO_ENABLED=1)
- golang.org/x/crypto/ssh — SSH remote ops
- github.com/masterzen/winrm — WinRM remote ops

## Build Commands

    # Windows (PowerShell) — CGO required for SQLite
    $env:CGO_ENABLED=1; go build -o vanguard.exe ./cmd/vanguard/

    # Linux
    CGO_ENABLED=1 go build -o vanguard ./cmd/vanguard/

    # Windows release build with version info (PowerShell)
    $version = "1.0.0"
    $date = Get-Date -Format "yyyy-MM-dd"
    $commit = git rev-parse --short HEAD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ridgelinecyberdefence/vanguard](https://github.com/ridgelinecyberdefence/vanguard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
