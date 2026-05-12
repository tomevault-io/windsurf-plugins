---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Coasty** is a full-stack AI collaboration platform with computer automation capabilities. It features a Next.js frontend with a FastAPI Python backend that orchestrates multi-agent AI systems capable of browser automation, terminal operations, and desktop control through containerized virtual machines. A cross-platform Electron desktop app provides a lightweight overlay that executes AI agent commands directly on the user's local machine.

## Architecture

### Frontend (Next.js 15 + React 19)
- **Framework**: Next.js 15 with App Router, TypeScript, Tailwind CSS
- **State Management**: Zustand stores for chat, models, user, and sessions
- **Key Libraries**:
  - Vercel AI SDK (`ai`) for streaming LLM responses
  - Radix UI for accessible components
  - Supabase for authentication and database
  - Stripe for billing/subscriptions
- **Provider System**: Multi-provider AI support (OpenAI, Anthropic, Azure, Google, Mistral, xAI, OpenRouter, Perplexity)

### Backend (Python FastAPI)
- **Framework**: FastAPI with async/await patterns
- **Key Services**:
  - `multi_agent_executor.py`: Orchestrates multi-agent task execution with browser, terminal, and desktop agents
  - `vm_control.py`: WebSocket-based VM control with persistent connections and auto-reconnection
  - `database.py`: Supabase integration for user data, chats, and billing
  - `agent_billing.py`: Tracks usage and credits for agent sessions
  - `search.py`: Google Custom Search API integration
- **API Routes**: `/api/chat`, `/api/models`, `/api/search`, `/api/vm`, `/api/billing`, `/api/files`

### VM Agent System
- **Architecture**: Docker containers running Ubuntu 22.04 with XFCE desktop
- **Agent Types**:
  - **Browser Agent**: Web automation using Chrome with remote debugging (search-first strategy)
  - **Terminal Agent**: Command execution and file operations
  - **Desktop Agent**: UI automation with screenshot analysis
- **Communication**: WebSocket protocol on port 8080 (8081 for localhost)
- **Tools**: Each agent has specialized tools (browser navigation, terminal commands, desktop controls)

### Electron Desktop App (`electron/`)

A cross-platform Electron app (v40.6.0) that runs as a floating overlay on the user's desktop, executing AI agent commands locally instead of in a remote VM.

- **Build System**: electron-vite + electron-builder, React 19 + Tailwind CSS renderer
- **Version**: 1.5.0 (`com.coasty.desktop`)
- **Key Dependencies**: `puppeteer-core` (browser automation), `ws` (WebSocket client)

#### Main Process (`electron/src/main/`)

- **`index.ts`**: App entry — creates frameless transparent window, system tray, registers IPC handlers, auto-updater
- **`auth.ts`**: Google OAuth via Supabase implicit flow — spins up local HTTP server on random port for callback, extracts tokens from URL fragment via HTML redirect trick, auto-refreshes tokens 5min before expiry
- **`ws-bridge.ts`**: Persistent WebSocket to backend `/api/electron/ws` — sends system info as URL params, auth credentials in first message body (not URL), auto-reconnect with exponential backoff (max 15s), 30s heartbeat
- **`window-manager.ts`**: Three modes — `auth` (400x500 centered), `compact` (360x56 top-center pill), `expanded` (400x520 chat panel). Smooth animation (320ms quintic ease-out), always-on-top management, opacity control (0.15–1.0), hides before screenshots
- **`local-executor.ts`**: Command handler registry (50+ commands) — maps backend command names to local handlers, normalizes params (filepath→path, find→old_text), auto-hides overlay during UI interactions
- **`desktop-automation.ts`**: Platform-specific mouse/keyboard/scroll/drag operations
- **`browser-automation.ts`**: Puppeteer-core controlling installed Chrome/Edge/Brave with isolated temp user-data-dir
- **`terminal.ts`**: Session-based shell execution (PowerShell on Windows, bash on Unix), 30s timeout
- **`file-ops.ts`**: File system CRUD (read, write, edit, append, delete, directory listing)
- **`screenshot.ts`**: Electron `desktopCapturer` API, resized to max 1280px, JPEG 70% quality
- **`permissions.ts`**: macOS-only — checks Screen Recording and Accessibility permissions
- **`auto-updater.ts`**: Generic update provider at `https://updates.coasty.ai`, checks every 4 hours

#### Platform-Specific Implementations

**Windows:**

- **Desktop Automation**: PowerShell + user32.dll P/Invoke (`mouse_event`, `keybd_event`, `SendKeys`)
  - Click/double-click via `System.Windows.Forms.Cursor` + `mouse_event` DLL calls
  - Typing via `SendKeys::SendWait()` with special character escaping
  - Key combos via `keybd_event` with virtual key codes (supports Win key, modifiers)
  - Scroll via `MOUSEEVENTF_WHEEL` (120 units per notch)
  - Drag via cursor position + mousedown/mouseup sequence
- **Browser Discovery**: Checks Program Files, Program Files (x86), LocalAppData for Chrome/Edge/Brave; falls back to `where.exe` PATH search
- **Terminal**: Uses `powershell.exe -Command` for all shell execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coasty-ai/open-computer-use](https://github.com/coasty-ai/open-computer-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
