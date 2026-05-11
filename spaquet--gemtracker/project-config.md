---
trigger: always_on
description: **Gemtracker** is an interactive Terminal User Interface (TUI) written in Go that analyzes Ruby gem dependencies from `Gemfile.lock` files. It helps developers identify security risks, outdated dependencies, version conflicts, and provides comprehensive dependency analysis for Ruby, Rails, and other Ruby-based projects.
---

# Gemtracker Development Guide

## Project Overview

**Gemtracker** is an interactive Terminal User Interface (TUI) written in Go that analyzes Ruby gem dependencies from `Gemfile.lock` files. It helps developers identify security risks, outdated dependencies, version conflicts, and provides comprehensive dependency analysis for Ruby, Rails, and other Ruby-based projects.

## Purpose & Vision

The tool provides developers with quick, actionable insights into their gem dependencies through an intuitive interactive interface. Instead of manually parsing Gemfile.lock or running multiple separate commands, developers get a unified view of:
- First-level gems (directly installed dependencies) with version info and availability of updates
- Full dependency tree showing which gems depend on other gems
- Vulnerability detection with pointers to affected gems
- Search functionality to find gems and their usage across the project

## Technology Stack

- **Language**: Go 1.24.0
- **TUI Framework**: BubbleTea (charmbracelet/bubbletea) for interactive terminal UI
- **Dependencies**:
  - `charmbracelet/bubbles` - Reusable components for BubbleTea
  - `charmbracelet/lipgloss` - Terminal styling and rendering

## Architecture

### Directory Structure

```
gemtracker/
├── cmd/gemtracker/          # Application entry point
│   └── main.go             # CLI bootstrap and TUI initialization
├── internal/
│   ├── gemfile/            # Gemfile.lock parsing and analysis logic
│   │   ├── parser.go       # Parse Gemfile.lock format
│   │   ├── analyzer.go     # Analyze dependencies and relationships
│   │   ├── dependencies.go # Dependency tree data structures
│   │   ├── outdated.go     # Check for newer versions available
│   │   └── vulnerabilities.go # CVE detection and reporting
│   └── ui/                 # BubbleTea TUI components
│       ├── model.go        # Root BubbleTea model and state management
│       ├── update.go       # Message handling and state updates
│       ├── view.go         # Rendering logic for all screens
│       └── styles.go       # Terminal colors, fonts, spacing
└── go.mod                   # Go module definition
```

### Core Components

**Gemfile Parser** (`internal/gemfile/parser.go`)
- Parses `Gemfile.lock` YAML-like format
- Extracts gem specs, versions, dependencies, and platforms

**Dependency Analyzer** (`internal/gemfile/analyzer.go`)
- Builds dependency graphs
- Identifies first-level vs transitive dependencies
- Resolves gem relationships and usage chains

**Outdated Checker** (`internal/gemfile/outdated.go`)
- Queries gem repositories for latest available versions
- Compares against currently installed versions
- Returns "latest" or new version number for display

**Vulnerability Scanner** (`internal/gemfile/vulnerabilities.go`)
- Detects known CVEs in gem versions
- Maps vulnerabilities to their gem locations in the tree

**UI Model** (`internal/ui/model.go`)
- Central BubbleTea model managing all TUI state
- Tracks current screen, selection state, search queries
- Owns version info, commit hash, date for display

**UI Update & View** (`internal/ui/update.go`, `internal/ui/view.go`)
- Message handling for user input and async operations
- Screen rendering (gem list, details, search, vulnerabilities)
- Keyboard navigation and interaction

### Async Architecture

**Version Update Checking** (`internal/gemfile/outdated.go`)
- Fetches latest available gem versions from rubygems.org API
- Runs asynchronously in background to keep UI responsive
- Results cached for 24 hours per project (in `~/.cache/gemtracker/`)
- Graceful degradation: if API unavailable, uses cached data or shows "checking..."
- Status indicator shows progress: "Checking updates... (15/189)"
- Respects API rate limits and implements exponential backoff

**Gem Health Checking** (`internal/gemfile/health.go`)
- Fetches maintenance health metrics from:
  - RubyGems API (last release date, maintainer count)
  - GitHub API (stars, maintainers, open issues, last push)
- Evaluates health into three tiers based on `ComputeHealthScore()`:
  - **HEALTHY** (🟢) - Activity within 1 year AND 2+ maintainers
  - **WARNING** (🟡) - No activity for 1-3 years OR single maintainer
  - **CRITICAL** (🔴) - No activity for 3+ years, archived, or disabled
- Runs one gem at a time (sequential) to avoid rate limiting
- Caches results for 24 hours per gem (`~/.cache/gemtracker/{hash}_health.json`)
- Shows health dots in gem list as data loads in background
- Handles GitHub rate limiting gracefully (60 req/hour for anonymous)

**BubbleTea Integration**
- Version checks and health updates sent as BubbleTea `Msg` events
- UI state updated via `Update()` method without blocking
- Progress indicators show loading state in real-time
- Error states (rate limited, network issues) displayed in statusbar
- Can be toggled on/off depending on user needs

## Key Features & Screens

### 1. First-Level Gems List
Display all gems directly installed (not dependencies of other gems):
- Gem name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spaquet/gemtracker](https://github.com/spaquet/gemtracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
