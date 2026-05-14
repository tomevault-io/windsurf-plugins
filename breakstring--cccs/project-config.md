---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Tauri application with a TypeScript/Vite frontend. The project combines a Rust backend (Tauri) with a web frontend built using Vite and TypeScript.

**CCCS (Claude Code Configuration Switcher)** is a system tray application that allows users to quickly switch between different Claude Code configuration profiles.

## Architecture

### Frontend (src/)
- **Entry point**: `src/main.ts` - Sets up the basic HTML structure and initializes the counter component
- **Components**: `src/counter.ts` - Simple counter functionality with click handlers
- **Styling**: `src/style.css` - Application styles
- **Build tool**: Vite with TypeScript support

### Backend (src-tauri/)
- **Entry point**: `src-tauri/src/main.rs` - Launches the Tauri application
- **Core logic**: `src-tauri/src/lib.rs` - Contains the main Tauri application setup with logging configuration
- **Build configuration**: `src-tauri/Cargo.toml` - Rust dependencies and build settings
- **Tauri configuration**: `src-tauri/tauri.conf.json` - App window settings, build commands, and bundle configuration

## Tauri Backend Architecture

### Core Modules
- **`app.rs`** - Main application state and initialization
- **`claude_detector.rs`** - Detects Claude Code installation and configuration files
- **`config_service.rs`** - Handles profile management and configuration file operations
- **`tray_service.rs`** - System tray icon and menu management
- **`monitor_service.rs`** - File monitoring for configuration changes
- **`settings_service.rs`** - Application settings management
- **`i18n_service.rs`** - Internationalization support (Chinese/English)
- **`validation.rs`** - JSON configuration validation framework
- **`types.rs`** - Common data structures and types
- **`error.rs`** - Error handling definitions

### Tauri Commands (API)
Available commands that can be called from frontend JavaScript:

#### Profile Management
- **`get_profiles_info()`** - Get summary information about profiles
- **`get_profiles_list()`** - Get detailed list of all profiles
- **`load_profile_content(profile_id: String)`** - Load content of a specific profile
- **`save_profile(profile_id: String, content: String)`** - Save changes to a profile
- **`create_new_profile(profile_name: String, content: String)`** - Create a new profile
- **`validate_json_content(content: String)`** - Validate JSON configuration

#### Field Exclusion Settings (Dynamic Ignored Fields)
- **`get_ignored_fields()`** - Get current list of ignored fields for profile comparison
- **`update_ignored_fields(fields: Vec<String>)`** - Update the ignored fields list
- **`get_default_ignored_fields()`** - Get default ignored fields (model, feedbackSurveyState)
- **`reset_ignored_fields_to_default()`** - Reset ignored fields to default values

#### Window Management
- **`close_settings_window()`** - Close the settings window

### Permissions Configuration
**Important**: `src-tauri/capabilities/default.json` defines Tauri permissions:
- Window operations: close, minimize, maximize, set-size, inner-size
- File system access for configuration files
- Dialog access for save/load operations

## Development Commands

### Frontend Development
- `npm run dev` - Start development server (runs Vite dev server on localhost:5173)
- `npm run build` - Build frontend for production (TypeScript compilation + Vite build)
- `npm run preview` - Preview production build

### Tauri Development
- `npm run tauri:dev` or `cargo tauri dev` - Run full application in development mode
- `npm run tauri:build` or `cargo tauri build` - Build production application
- Development is handled through the Tauri configuration in `tauri.conf.json`
- Frontend dev server runs on `http://localhost:5173`
- Build process: `npm run build` creates the `dist` directory for Tauri

### Debugging and Testing

#### Recommended Debugging Workflow (TabbyMCP + tmux)

**IMPORTANT**: 使用以下推荐的调试流程避免阻塞主工作流程：

##### 1. 创建专用的 tmux 调试会话
```bash
# 创建新的 tmux 会话用于调试
tmux new-session -d -s cccs-debug
```

##### 2. 在 tmux 会话中运行应用程序
```bash
# 切换到项目目录
tmux send-keys -t cccs-debug 'cd /Users/kenn/Projects/cccs' Enter

# 启动应用程序（开发模式）
tmux send-keys -t cccs-debug 'npm run tauri:dev' Enter
```

##### 3. 查看应用程序日志
```bash
# 捕获 tmux 会话的屏幕内容查看日志
tmux capture-pane -t cccs-debug -p

# 如果需要查看更多历史输出
tmux capture-pane -t cccs-debug -S -1000 -p
```

##### 4. 在会话中执行其他调试命令
```bash
# 向 tmux 会话发送任意命令
tmux send-keys -t cccs-debug 'echo "Debug message"' Enter

# 停止应用程序（如果需要）
tmux send-keys -t cccs-debug 'C-c'

# 重新启动应用程序
tmux send-keys -t cccs-debug 'npm run tauri:dev' Enter
```

##### 5. 清理调试会话
```bash
# 结束调试会话
tmux kill-session -t cccs-debug
```

##### TabbyMCP 工具使用
- **获取终端会话列表**: `mcp__tabbymcp__get_ssh_session_list()`
- **执行命令**: `mcp__tabbymcp__exec_command({tabId, command, commandExplanation})`
- **查看终端缓冲区**: `mcp__tabbymcp__get_terminal_buffer({tabId, startLine, endLine})`

##### 调试优势
1. **非阻塞**：不会阻塞主要的工作流程
2. **持久化**：可以随时查看应用程序状态和日志
3. **灵活性**：可以在不中断应用的情况下执行其他命令
4. **隔离性**：调试环境与主工作环境分离

##### 注意事项
- 使用 `tmux send-keys` 而不是直接 `tmux attach`，避免阻塞
- 定期使用 `tmux capture-pane` 查看最新日志
- 调试完成后记得清理 tmux 会话


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breakstring/cccs](https://github.com/breakstring/cccs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
