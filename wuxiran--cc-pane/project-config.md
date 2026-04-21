---
trigger: always_on
description: > Multi-instance split-pane manager for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — a cross-platform desktop app built with Tauri 2.
---

# CC-Panes Agent Guide

> Multi-instance split-pane manager for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — a cross-platform desktop app built with Tauri 2.

This document provides essential information for AI coding agents working on the CC-Panes codebase.

## Project Overview

CC-Panes is a desktop application that lets users run multiple Claude Code CLI instances side by side in a split-pane terminal layout. It organizes AI-powered development workflows with workspaces, projects, and tasks — all from a single desktop app.

### Key Features

- **Split-Pane Terminal** — Run multiple terminals in flexible horizontal/vertical split layouts
- **Workspace Management** — Organize projects into workspaces with pinning, hiding, and reordering
- **Built-in Terminal** — Full-featured terminal (xterm.js + PTY) with multi-tab support
- **Claude Code Integration** — Launch Claude Code sessions, resume conversations, manage providers
- **Git Integration** — Branch status, pull/push/fetch/stash, worktree management, git clone
- **Session Management** — Track launch history, clean broken sessions, resume previous work
- **Local History** — File version tracking with diff view, labels, branch-aware snapshots
- **File Browser** — Project file tree with search, create, rename, delete, copy, move operations
- **Code Editor** — Monaco-based editor with 60+ language support, Markdown preview, image preview
- **Screenshot** — Region capture with global shortcut, multi-monitor support
- **MCP Server Config** — Configure MCP servers per project

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Desktop Framework | Tauri 2 | Rust backend + system WebView |
| Frontend | React 19 + TypeScript 5.6 | UI components |
| State Management | Zustand 5 + Immer | Immutable state updates |
| UI Library | shadcn/ui + Radix UI | Component library |
| Styling | Tailwind CSS 4 | Utility-first CSS |
| Terminal | xterm.js + portable-pty | Frontend rendering + backend PTY |
| Split Panes | Allotment | Resizable split layout |
| Data Storage | SQLite (rusqlite) | Local persistence |
| Build Tool | Vite 6 | Frontend bundler |
| Testing | Vitest + jsdom | Frontend tests |
| Testing | Built-in Rust test | Backend tests |

## Architecture

### Data Flow

```
React Component → Zustand Store → Service (invoke) → Tauri IPC → Command → Service → Repository → SQLite/FS
```

```
┌─────────────────────────────────────────────────────────────┐
│  React Frontend                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────┐  │
│  │ Sidebar  │ │ Panes    │ │ Panels   │ │ UI Components │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └───────────────┘  │
│       │             │            │                           │
│  ┌────┴─────────────┴────────────┴────┐                     │
│  │  Services (invoke) + Stores        │                     │
│  └────────────────┬───────────────────┘                     │
├───────────────────┼─────────────────────────────────────────┤
│  Tauri IPC        │                                         │
├───────────────────┼─────────────────────────────────────────┤
│  Rust Backend     │                                         │
│  ┌────────────────┴───────────────────┐                     │
│  │  Commands → Services → Repository  │                     │
│  └────────────────┬───────────────────┘                     │
│  ┌────────────────┴───────────────────┐                     │
│  │  SQLite / File System / PTY        │                     │
│  └────────────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

### Workspace Crate Structure

The Rust backend is organized as a Cargo workspace:

| Crate | Purpose |
|-------|---------|
| `src-tauri` | Tauri application entry point, command handlers |
| `cc-panes-core` | Core business logic, framework-independent |
| `cc-panes-api` | HTTP/WebSocket API adapter |
| `cc-panes-web` | Web terminal server for Docker deployment |
| `cc-panes-cli-hook` | Shared CLI hook runner source directory |
| `cc-memory` | Memory system for Claude memories |
| `cc-memory-mcp` | MCP server for memory system |
| `cc-notify` | Push notification system |
| `cc-cli-adapters` | CLI tool adapter layer |

## Project Structure

```
cc-panes/
├── web/                    # React frontend source
│   ├── components/         # React components
│   │   ├── panes/          # Split-pane terminal components
│   │   ├── sidebar/        # Sidebar components
│   │   ├── settings/       # Settings sub-components
│   │   └── ui/             # shadcn/ui base components
│   ├── stores/             # Zustand state management
│   ├── services/           # Frontend service layer (invoke wrappers)
│   ├── hooks/              # Custom React hooks
│   ├── types/              # TypeScript type definitions
│   ├── i18n/               # Internationalization
│   ├── lib/                # Shared frontend helpers
│   └── utils/              # Utility functions
│
├── src-tauri/              # Tauri Rust backend
│   └── src/
│       ├── commands/        # Tauri IPC command handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuxiran/cc-pane](https://github.com/wuxiran/cc-pane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
