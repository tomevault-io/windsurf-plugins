---
trigger: always_on
description: Control mouse, keyboard, and screen for desktop automation tasks
---


# Desktop Control Skill

This skill provides comprehensive desktop automation capabilities through PyAutoGUI, allowing AI agents to control the mouse, keyboard, take screenshots, and interact with the desktop environment.

## How to Use This Skill

As an AI agent, you can invoke desktop automation commands using the `uvx desktop-agent` CLI.

### Command Structure

All commands follow this pattern:

```bash
uvx desktop-agent <category> <command> [arguments] [options]
```

**Categories:**
- `mouse` - Mouse control
- `keyboard` - Keyboard input
- `screen` - Screenshots and screen analysis
- `message` - User dialogs
- `app` - Application control (open, focus, list windows)

## Available Commands

### 🖱️ Mouse Control (`mouse`)

Control cursor movement and clicks.

```bash
# Move cursor to coordinates
uvx desktop-agent mouse move <x> <y> [--duration SECONDS]

# Click at current position or specific coordinates
uvx desktop-agent mouse click [x] [y] [--button left|right|middle] [--clicks N]

# Specialized clicks
uvx desktop-agent mouse double-click [x] [y]
uvx desktop-agent mouse right-click [x] [y]
uvx desktop-agent mouse middle-click [x] [y]

# Drag to coordinates
uvx desktop-agent mouse drag <x> <y> [--duration SECONDS] [--button BUTTON]

# Scroll (positive=up, negative=down)
uvx desktop-agent mouse scroll <clicks> [x] [y]

# Get current mouse position
uvx desktop-agent mouse position
```

**Examples:**
```bash
# Move to center of 1920x1080 screen
uvx desktop-agent mouse move 960 540 --duration 0.5

# Right-click at specific location
uvx desktop-agent mouse right-click 500 300

# Scroll down 5 clicks
uvx desktop-agent mouse scroll -5
```

### ⌨️ Keyboard Control (`keyboard`)

Type text and execute keyboard shortcuts.

```bash
# Type text
uvx desktop-agent keyboard write "<text>" [--interval SECONDS]

# Press keys
uvx desktop-agent keyboard press <key> [--presses N] [--interval SECONDS]

# Execute hotkey combination (comma-separated)
uvx desktop-agent keyboard hotkey "<key1>,<key2>,..."

# Hold/release keys
uvx desktop-agent keyboard keydown <key>
uvx desktop-agent keyboard keyup <key>
```

**Examples:**
```bash
# Type text with natural delay
uvx desktop-agent keyboard write "Hello World" --interval 0.05

# Copy selected text
uvx desktop-agent keyboard hotkey "ctrl,c"

# Open Task Manager
uvx desktop-agent keyboard hotkey "ctrl,shift,esc"

# Press Enter 3 times
uvx desktop-agent keyboard press enter --presses 3
```

**Common Key Names:**
- Modifiers: `ctrl`, `shift`, `alt`, `win`
- Special: `enter`, `tab`, `esc`, `space`, `backspace`, `delete`
- Function: `f1` through `f12`
- Arrows: `up`, `down`, `left`, `right`

### 🖼️ Screen & Screenshots (`screen`)

Capture screenshots and analyze screen content. Supports targeting specific windows.

```bash
# Take screenshot
uvx desktop-agent screen screenshot <filename> [--region "x,y,width,height"] [--window <title>] [--active]

# Locate image on screen or within window
uvx desktop-agent screen locate <image_path> [--confidence 0.0-1.0] [--window <title>] [--active]
uvx desktop-agent screen locate-center <image_path> [--confidence 0.0-1.0] [--window <title>] [--active]

# Locate text using OCR within window
uvx desktop-agent screen locate-text-coordinates <text> [--window <title>] [--active]
uvx desktop-agent screen read-all-text [--window <title>] [--active]

# Utility commands
uvx desktop-agent screen pixel <x> <y>
uvx desktop-agent screen size
uvx desktop-agent screen on-screen <x> <y>
```

**Examples:**
```bash
# Screenshot of active window
uvx desktop-agent screen screenshot active.png --active

# Screenshot of a specific application
uvx desktop-agent screen screenshot chrome.png --window "Google Chrome"

# Locate image within Notepad
uvx desktop-agent screen locate-center button.png --window "Notepad"
```

### 💬 Message Dialogs (`message`)

Display user interaction dialogs.

```bash
# Show alert
uvx desktop-agent message alert "<text>" [--title TITLE] [--button BUTTON]

# Show confirmation dialog
uvx desktop-agent message confirm "<text>" [--title TITLE] [--buttons "OK,Cancel"]

# Prompt for input
uvx desktop-agent message prompt "<text>" [--title TITLE] [--default TEXT]

# Password input
uvx desktop-agent message password "<text>" [--title TITLE] [--mask CHAR]
```

**Examples:**
```bash
# Simple alert
uvx desktop-agent message alert "Task completed!"

# Get user confirmation
uvx desktop-agent message confirm "Continue with operation?"

# Ask for user input
uvx desktop-agent message prompt "Enter your name:"
```

### 📱 Application Control (`app`)

Control applications across Windows, macOS, and Linux.

```bash
# Open an application by name
uvx desktop-agent app open <name> [--arg ARGS...]

# Focus on a window by title/name
uvx desktop-agent app focus <name>

# List all visible windows
uvx desktop-agent app list
```

**Examples:**
```bash
# Windows: Open Notepad
uvx desktop-agent app open notepad

# Windows: Open Chrome with a URL
uvx desktop-agent app open "chrome" --arg "https://google.com"

# macOS: Open Safari
uvx desktop-agent app open "Safari"

# Focus on a specific window
uvx desktop-agent app focus "Untitled - Notepad"

# List all open windows
uvx desktop-agent app list
```

## Common Automation Workflows


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickporto/desktop-agent](https://github.com/patrickporto/desktop-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
