---
trigger: always_on
description: Claude Code Web is a web-based remote development environment that allows users to access and control Claude Code and VS Code (Cursor) on remote servers through their browser.
---

# Claude Code Web Project Memory

## Project Overview

Claude Code Web is a web-based remote development environment that allows users to access and control Claude Code and VS Code (Cursor) on remote servers through their browser.

## Technology Stack Decisions

- **Architecture**: Self-built PTY solution (node-pty + WebSocket)
- **Frontend**: React + TypeScript + xterm.js + Material-UI
- **Backend**: Fastify + WebSocket + node-pty
- **Database**: PostgreSQL (user authentication)
- **IDE Integration**: code-server
- **Deployment**: Docker + Docker Compose
- **Package Management**: pnpm (monorepo)

## Core Requirements

1. Launch Claude Code on remote server
2. Users access terminal via web, view history, real-time interaction
3. Integrate VS Code (Cursor), view code changes on web

## Development Requirements

- All feature requirements recorded in FEATURE.md
- Continuously save project memory to CLAUDE.md
- Must write test cases, continuous regression testing

## Project Status

- ✅ Technical research completed
- ✅ Architecture solution determined (self-built PTY solution)
- ✅ Requirements document created (FEATURE.md)
- ✅ Project structure setup completed
- ✅ Backend framework implemented (Fastify + WebSocket + node-pty)
- ✅ Test framework configuration completed
- ✅ CI/CD pipeline configured
- ✅ Docker deployment configured
- ✅ Frontend interface development completed
- ✅ User authentication system implemented
- ✅ WebSocket connection successful
- ✅ xterm.js dimensions error fixed (StableTerminal)
- ✅ UI style optimization (VS Code style)
- ✅ WebSocket error handling improved
- ✅ Session management features implemented (create, switch, delete, rename)
- ✅ Session management implemented (in-memory)
- ✅ Terminal height adaptive fix (FitAddon)
- ✅ Session output buffering implemented (in-memory)
- ✅ Real-time session management features
- ✅ Mobile responsive design implemented
- ✅ Session List real-time updates (WebSocket)
- ✅ Smart CWD detection system implemented
- ✅ Path smart abbreviation display
- ✅ Execution status real-time detection
- ✅ Mobile keyboard adaptation optimization
- ✅ iPhone login interface keyboard handling
- ✅ Mobile debug interface implemented
- ✅ Mobile virtual keyboard toolbar (ESC, page navigation, arrow keys, etc.)
- ✅ Fixed mobile terminal refresh issues (optimized ANSI escape sequence handling)
- ✅ Advanced ANSI sequence optimization (mobile performance improvement)
- ✅ Virtual keyboard toolbar auto-hide (synchronized with iOS native keyboard)
- ✅ Enhanced virtual keyboard (Shift+Tab, nano editor shortcuts)
- ✅ Mobile terminal gesture support Hooks
- ✅ Smart Tab auto-completion Hook
- ✅ Mobile virtual keyboard toolbar optimization (single-row scrolling layout)
- ✅ Auto-scroll fix (terminal scrolls to bottom when keyboard appears)
- ✅ Focus management optimization (buttons don't retain selected state after click)
- ✅ iOS Chinese input method compatibility fix (space, numbers, symbols input)
- ✅ Smart cursor tracking scroll system (Claude Code mid-input scenarios)
- ✅ Container isolation mode implemented (Docker/Podman)
- ✅ Container lifecycle management improved (auto-cleanup, session recovery)
- ✅ xterm.js race condition thoroughly fixed (delayed initialization + WebSocket timing optimization)
- ✅ GitHub OAuth integration implemented (OAuth authentication, repository management, Token refresh)
- ✅ VS Code Remote-SSH integration solution implemented (SSHpiper workingDir mode)
- ✅ SSH public key authentication system implemented (removed password authentication, public key only)
- ✅ SSHpiper workingDir configuration automation
- ✅ SSH public key drag-and-drop upload and smart parsing
- ✅ Multi-IDE support (VS Code, Cursor, Windsurf)
- ✅ One-click IDE open feature implemented
- ✅ Dockerode integration implemented (replacing node-pty + docker exec)
- ✅ Invite code registration restriction system implemented
- ✅ Dynamic configuration management system implemented
- ✅ Inlets tunnel feature implemented (similar to ngrok, expose container services to public internet)

## Key Decision Records

1. **2025-06-14**: Initially chose ttyd hybrid solution, later decided to use node-pty self-built solution
2. **2025-06-14**: Decided to use Node.js ecosystem for rapid development and maintenance
3. **2025-06-14**: Chose Fastify over Express for performance improvement
4. **2025-06-14**: Used pnpm as package manager, supporting monorepo structure
5. **2025-06-14**: Fixed xterm.js dimensions error, downgraded to xterm@4.19.0 stable version
6. **2025-06-14**: Created multiple terminal component attempts, finally used StableTerminal
7. **2025-06-14**: Implemented session management, maintaining session state and output buffer in memory
8. **2025-06-14**: Used singleton pattern to ensure SessionManager consistency
9. **2025-06-14**: Implemented output buffer chunked storage, preserving ANSI escape sequences
10. **2025-06-14**: Implemented mobile responsive design, unified left-side Drawer usage
11. **2025-06-14**: Implemented smart CWD detection, using lsof (macOS) and /proc (Linux)
12. **2025-06-14**: Adopted event-driven CWD detection strategy replacing scheduled checks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fbzhong/claude-code-web](https://github.com/fbzhong/claude-code-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
