---
trigger: always_on
description: > **Purpose**: This document provides AI coding agents with comprehensive context to understand, develop with, and contribute to the PyWry library.
---

# AGENTS.md - PyWry Library Guide for AI Coding Agents

> **Purpose**: This document provides AI coding agents with comprehensive context to understand, develop with, and contribute to the PyWry library.

---

## Quick Reference

| Aspect | Details |
|--------|---------|
| **Language** | Python 3.10–3.14 (base); dev tooling requires 3.11+ |
| **Type System** | ty type checker, Pydantic v2 models |
| **Style** | Ruff (line length 100), NumPy docstrings |
| **Testing** | pytest with fixtures, `PYWRY_HEADLESS=1` for CI |
| **Architecture** | Subprocess IPC (native) + FastAPI inline server (notebook / browser) |
| **State** | Memory (default), Redis, or SQLite + SQLCipher for multi-worker / at-rest encryption |
| **Chat** | Streaming chat widget with OpenAI / Anthropic / Magentic / Callback / Stdio-ACP / Deepagent providers |
| **MCP** | FastMCP server (`pywry mcp`) exposing widget, chart, grid, chat, tvchart, and auth tooling |

---

## Project Overview

**PyWry** is a blazingly fast rendering library for generating and managing native desktop windows, iFrames, and Jupyter widgets — with full bidirectional Python ↔ JavaScript communication.

Unlike dashboard libraries that only render output, PyWry provides a complete event system where Python can send events to JavaScript and JavaScript can invoke Python callbacks, enabling truly interactive applications.

Built on [PyTauri](https://pypi.org/project/pytauri/) (which uses Rust's [Tauri](https://tauri.app/) framework), it leverages the OS webview instead of bundling a browser engine—resulting in binaries under 3MB compared to Electron's 150MB+ overhead.

### Core Capabilities

- **Window Modes**: `NEW_WINDOW`, `SINGLE_WINDOW`, `MULTI_WINDOW`, `NOTEBOOK`, `BROWSER`.
- **Notebook Support**: Automatic inline rendering via anywidget or IFrame in Jupyter / VS Code / Colab.
- **Toolbar System**: Declarative Pydantic components (`Button`, `Select`, `MultiSelect`, `TextInput`, `SecretInput`, `SliderInput`, `RangeInput`, `Toggle`, `Checkbox`, `RadioGroup`, `TabGroup`, `Marquee`, `Modal`, …) with top / bottom / left / right / header / footer / inside positions and automatic nested-flexbox layout.
- **Two-Way Events**: Python ↔ JavaScript communication with pre-wired Plotly / AgGrid / TradingView / chat events plus utility events for DOM manipulation.
- **Theming & CSS**: Light / dark / system modes, `--pywry-*` CSS variables, component-ID targeting, hot reload, and dynamic styling via events (`pywry:set-style`, `pywry:inject-css`).
- **Security**: Scoped token auth enabled by default, CSP headers, internal API protection, `SecuritySettings.strict() / .permissive() / .localhost()` presets.
- **AgGrid Tables**: Pandas → AgGrid conversion with pre-wired events, context menus, server-side mode, and persisted column/filter/sort state.
- **Plotly Charts**: Plotly rendering with pre-wired plot events, layout / trace / figure updates, and state round-trips.
- **TradingView Charts**: Extended Lightweight Charts integration — drawing surface (trendlines, fib tools, text, price notes, brushes), pluggable datafeed API, UDF adapter for external quote servers, streaming bar updates, compare overlays, compare-derivative indicators (Spread / Ratio / Sum / Product / Correlation), savable layouts, and a themeable settings panel.
- **Chat Widget**: Streaming chat UI with threads, artifacts, slash commands, plan / todo updates, permission prompts, context sources, and pluggable providers (`OpenAIProvider`, `AnthropicProvider`, `MagenticProvider`, `CallbackProvider`, `StdioProvider` for ACP subprocesses, `DeepagentProvider` for LangChain Deep Agents).
- **MCP Server**: `pywry mcp --transport stdio | http` exposes widget management, components, chart / grid / tvchart control, events, chat-agent driving, auth, and docs skills to any MCP client.
- **Toast Notifications**: Built-in alert system (info, success, warning, error, confirm) with positioning and blocking overlay.
- **Marquee Ticker**: Scrolling text/content with dynamic per-item updates.
- **Secrets Handling**: `SecretInput` stores values server-side; HTML only carries opaque component IDs.
- **Hot Reload**: CSS injection and JS refresh with scroll preservation.
- **Bundled Libraries**: Plotly.js, AgGrid, and TradingView Lightweight Charts are bundled (offline capable) and served from `pywry/frontend/assets/`.
- **Native File Dialogs, Menus, Tray**: Tauri-powered save/open dialogs and filesystem access; `MenuProxy`, `TrayProxy`, and `WindowProxy` wrap the Tauri runtime APIs.
- **Configuration System**: TOML files, `pyproject.toml` `[tool.pywry]`, environment variables (`PYWRY_*`).
- **CLI Tools**: `pywry config`, `pywry init`, `pywry mcp`, `pywry mcp install`.
- **Deploy Mode**: Horizontal scaling with Redis-backed state, or SQLite + SQLCipher for single-node at-rest encryption.
- **Authentication**: OAuth2 (Google / GitHub / Microsoft / OIDC / custom) with PKCE, keyring-backed token storage, automatic refresh, and optional RBAC for deploy-mode routes.
- **Standalone Executables**: `pywry[freeze]` ships a PyInstaller hook — no `.spec` edits or `--hidden-import` flags required.

### Dependencies

Base package (always installed):

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deeleeramone/PyWry](https://github.com/deeleeramone/PyWry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
