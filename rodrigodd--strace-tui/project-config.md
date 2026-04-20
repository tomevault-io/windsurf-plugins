---
trigger: always_on
description: strace-tui is a terminal user interface (TUI) for visualizing and exploring strace output. It parses strace output files and provides an interactive interface for browsing syscalls, viewing details, and resolving backtraces.
---

# Copilot Instructions for strace-tui

## Project Overview

strace-tui is a terminal user interface (TUI) for visualizing and exploring strace output. It parses strace output files and provides an interactive interface for browsing syscalls, viewing details, and resolving backtraces.

**Status**: ✅ Complete - Full TUI implementation with interactive navigation  
**Language**: Rust (Edition 2024)  
**Main Dependencies**: nom (parsing), ratatui (TUI), crossterm (terminal), serde (JSON), clap (CLI)

## Quick Commands

```bash
# Build
cargo build --release

# Run tests
cargo test

# TUI mode (default)
./target/release/strace-tui parse trace.txt
./target/release/strace-tui trace ls -la

# JSON mode
./target/release/strace-tui parse trace.txt --json --pretty
./target/release/strace-tui trace --json echo "test"

# Generate test trace
cargo build --example syscall_test
strace -o trace.txt -t -k -f -s 1024 ./target/debug/examples/syscall_test
```

## Architecture

### Overview
The project has three main components:

1. **Parser** (`src/parser/`) - Parses strace output into structured data
2. **TUI** (`src/tui/`) - Interactive terminal interface for exploration
3. **CLI** (`src/main.rs`) - Command-line interface with parse/trace subcommands

### Mode Selection
- **Default**: Interactive TUI for visual exploration
- **JSON mode**: Use `--json` flag for programmatic access

### Module Structure

```
src/
├── main.rs              # CLI entry point, subcommand handling
├── lib.rs               # Library interface
├── parser/
│   ├── mod.rs           # Parser orchestration, multi-line handling
│   ├── types.rs         # Data structures (SyscallEntry, etc.)
│   ├── line_parser.rs   # nom-based line parsing
│   ├── backtrace_parser.rs  # Backtrace parsing
│   └── resolver.rs      # addr2line integration
└── tui/
    ├── mod.rs           # TUI entry point, terminal setup
    ├── app.rs           # Application state and event handling
    ├── ui.rs            # Rendering logic
    └── events.rs        # Keyboard input handling
```

## Key Features

### Parser
- **Input format**: `strace -o file.txt -t -k -f -s 1024 <cmd>`
- **Supports**: All syscall patterns, signals, exits, multi-process traces, kernel backtraces
- **Parser**: nom combinator-based, handles `<unfinished ...>` and `<... resumed>` patterns
- **Output**: Structured `SyscallEntry` objects with full metadata

### TUI
- **Navigation**: Arrow keys, vim-style (j/k), page up/down, home/end
- **Expansion**: Enter to expand syscall details and toggle backtraces
- **Colors**: Red (errors), yellow (signals), cyan (exits), green (resolved addresses)
- **Performance**: Lazy backtrace resolution (only on expand), smooth scrolling
- **Help**: Press `?` for keybindings overlay

### addr2line Integration
- **On-demand**: Backtraces resolved when expanded in TUI
- **Batch mode**: Pre-resolve all with `-r` in JSON mode
- **Caching**: Addresses cached to avoid redundant lookups
- **Method**: Shells out to system `addr2line` binary

## Data Structures

```rust
// Main entry type
pub struct SyscallEntry {
    pub pid: u32,
    pub timestamp: String,
    pub syscall_name: String,
    pub arguments: String,
    pub return_value: Option<String>,
    pub errno: Option<ErrnoInfo>,
    pub duration: Option<f64>,
    pub backtrace: Vec<BacktraceFrame>,
    pub is_unfinished: bool,
    pub is_resumed: bool,
    pub signal: Option<SignalInfo>,
    pub exit_info: Option<ExitInfo>,
}

// Backtrace frame
pub struct BacktraceFrame {
    pub binary: String,
    pub function: Option<String>,
    pub offset: Option<String>,
    pub address: String,
    pub resolved: Option<ResolvedLocation>,
}

// TUI state
pub struct App {
    pub entries: Vec<SyscallEntry>,
    pub resolver: Addr2LineResolver,
    pub selected_index: usize,
    pub scroll_offset: usize,
    pub expanded_items: HashSet<usize>,
    pub expanded_backtraces: HashSet<usize>,
    pub should_quit: bool,
    pub show_help: bool,
}
```

## strace Output Patterns

The parser handles all major strace patterns:

### Regular syscall
```
12345 10:20:30 write(1, "hello\n", 6) = 6
```

### With error
```
12345 10:20:30 open("/no/file", O_RDONLY) = -1 ENOENT (No such file or directory)
```

### Unfinished/resumed (multi-line async)
```
12345 10:20:30 read(3, <unfinished ...>
12346 10:20:31 write(1, "test", 4) = 4
12345 10:20:32 <... read resumed> "data", 1024) = 4
```

### Signal
```
12345 10:20:30 --- SIGSEGV {si_signo=SIGSEGV, si_code=SEGV_MAPERR, ...} ---
```

### Exit
```
12345 10:20:30 +++ exited with 0 +++
12346 10:20:31 +++ killed by SIGKILL +++
```

### Backtrace (multi-line)
```
12345 10:20:30 write(1, "test", 4) = 4
 > /usr/lib/libc.so.6(__write+0x14) [0x10e53e]
 > /home/user/program(main+0x2a) [0x401234]
```

## Parser Flow

1. **Line-by-line parsing** (`src/parser/mod.rs:parse_lines()`)
   - Read each line
   - Check if it's a backtrace line (starts with ` > `)
   - If backtrace: add to pending backtrace buffer
   - If syscall: parse with `line_parser.rs`, attach pending backtrace

2. **Syscall parsing** (`src/parser/line_parser.rs:parse_strace_line()`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rodrigodd/strace-tui](https://github.com/Rodrigodd/strace-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
