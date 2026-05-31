---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solo for Laravel is a developer tool package that runs multiple development commands simultaneously in a tabbed terminal interface. It enables developers to consolidate all local development processes (Vite, logs, queues, tests, etc.) into a single `php artisan solo` command.

**Key Characteristics:**
- Laravel package (requires Laravel 10-12, PHP 8.2+)
- Unix-only (requires ext-pcntl and ext-posix)
- Uses GNU Screen as an intermediary for PTY and ANSI handling
- Built on Laravel Prompts and Chewie for TUI rendering
- Configuration-driven command setup via `config/solo.php`

## Common Commands

### Development
```bash
# Run all tests
./vendor/bin/phpunit

# Run specific test suite
./vendor/bin/phpunit --testsuite=unit
./vendor/bin/phpunit --testsuite=integration

# Run single test file
./vendor/bin/phpunit tests/Unit/AnsiAwareTest.php

# Run single test method
./vendor/bin/phpunit --filter testMethodName

# Run the demo (requires testbench)
composer dev
# or: ./vendor/bin/testbench solo

# Install dependencies
composer install
```

### Testing the Package
Since this is a Laravel package (not an application), it uses Orchestra Testbench for testing. Tests are in the `tests/` directory with two suites:
- `tests/Unit/` - Unit tests for individual components
- `tests/Integration/` - Integration tests for full features

## Architecture Overview

### Core Architectural Pattern: Event Loop with Process Management

Solo uses a **render loop pattern** where the main Dashboard continuously:
1. Collects output from all running processes (via process output callbacks)
2. Renders the current state to the terminal
3. Handles keyboard input
4. Updates process states
5. Repeats at ~40 FPS (25ms intervals)

### Component Hierarchy

```
php artisan solo
    ↓
Console\Commands\Solo
    ↓
Prompt\Dashboard (extends Laravel Prompt)
    ├── Manages Command[] array
    ├── Runs event loop (Loops trait from Chewie)
    └── Handles keyboard input via KeyPressListener
        ↓
    Commands\Command (implements Loopable)
        ├── ManagesProcess trait - process lifecycle
        ├── Screen (from soloterm/screen) - virtual terminal
        └── onTick() - called each loop iteration
```

### Key Classes and Their Roles

**Manager** (`src/Manager.php`)
- Singleton service registered in SoloServiceProvider
- Loads commands from config
- Manages themes and hotkeys
- Central configuration point

**Dashboard** (`src/Prompt/Dashboard.php`)
- Main TUI controller extending Laravel Prompt
- Owns the event loop via Chewie's Loops trait
- Manages tab switching and keyboard routing
- Coordinates rendering and input handling
- Handles terminal resize events (SIGWINCH)

**Command** (`src/Commands/Command.php`)
- Base class for all runnable commands
- Implements Loopable interface (onTick method)
- Uses ManagesProcess trait for process lifecycle
- Maintains a virtual Screen for output buffering
- Supports two modes: MODE_PASSIVE (read-only) and MODE_INTERACTIVE (input forwarded)
- Can be created from strings, custom classes, or via Command::from() static constructor

**ManagesProcess** (`src/Commands/Concerns/ManagesProcess.php`)
- Critical trait handling subprocess lifecycle
- Wraps user commands in GNU Screen for PTY emulation
- Manages incremental output collection without blocking
- Handles graceful shutdown (SIGTERM) and force kill (SIGKILL)
- Uses ProcessTracker to kill child processes on exit

**Hotkeys & KeyHandler**
- `Hotkeys\DefaultHotkeys.php` and `Hotkeys\VimHotkeys.php` define keybindings
- `Hotkeys\KeyHandler.php` is an enum mapping actions to closures
- Hotkeys are dynamically bound based on current tab and command state
- Each Command can define custom hotkeys() method for command-specific keys

### Process Wrapping Strategy

Solo wraps user commands in GNU Screen to solve PTY and ANSI rendering challenges:

```bash
bash -c 'export LC_ALL=en_US.UTF-8 && stty cols 120 rows 40 && screen -U -q sh -c "printf MARKER; user-command; sleep 0.25; printf MARKER"'
```

This provides:
- Proper PTY allocation for interactive programs
- ANSI code handling via Screen's terminal emulation
- Output markers to filter Screen's own output
- Small delay before exit to capture all output

### Virtual Terminal (Screen class)

Commands write output to a `soloterm/screen` Screen object, which is a virtual terminal buffer that:
- Handles ANSI codes (colors, cursor movement, etc.)
- Wraps lines to terminal width
- Maintains a buffer of printable output
- Supports querying (for responsive applications)

### Differential Rendering

Recent optimization uses DiffRenderer (`src/Support/DiffRenderer.php`) to:
- Compare previous and current screen states
- Generate minimal ANSI sequences to update only changed cells
- Reduce terminal I/O for performance

### Configuration Flow

1. `config/solo.php` defines commands array
2. Manager->loadCommands() reads config
3. Manager->addCommand() converts strings/objects to Command instances
4. Dashboard constructor calls ->setDimensions() and ->autostart() on each
5. Commands marked ->lazy() skip autostart

### Lazy Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soloterm/solo](https://github.com/soloterm/solo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
