---
trigger: always_on
description: - **Name**: ve — minimal ncurses-based text editor (C++17)
---

# Cursor Rules for ve Editor Project

## Project Overview
- **Name**: ve — minimal ncurses-based text editor (C++17)
- **Binary**: `ve` (two letters, simple)
- **Build system**: CMake (>= 3.15) with version 0.1.0
- **Libraries**: ncurses, GoogleTest
- **Platform**: macOS primary (AppleClang); portable C++17
- **Main header**: `editor.h` — contains the `Editor` class definition
- **Core sources**: Layer-based organization:
  - Presentation: `core.cpp`, `display.cpp`
  - Input: `key_bindings.cpp`
  - Business Logic: `ops.cpp`, `clipboard.cpp`, `buffer.cpp`
  - Data: `file.cpp`, `workspace.cpp`, `segment.cpp`, `tempfile.cpp`
  - Infrastructure: `session.cpp`, `help.cpp`
  - Entry: `main.cpp`
- **Test directory**: `tests/` — comprehensive test suite using GoogleTest

## Core Architecture Patterns

### Dual-Workspace Model
- **Primary Workspace** (`wksp_`): Main editing workspace for file content
- **Alternative Workspace** (`alt_wksp_`): Accessible via `^N` and `F3`
- Both workspaces share the same tempfile for efficient memory usage

### Dual-Mode Data Model
1. **Segment Chain Model**: Linked list of `Segment` objects with metadata (no raw byte data in memory)
   - Efficient for large file handling with on-demand loading
2. **In-Memory Current Line** (`std::string current_line_`): Primary editing interface
   - Data read from segment when line loaded with `get_line()`
   - Data written back to temp file when saved with `put_line()`

### Current Line Buffer Pattern
- **During editing**: Use `get_line(line_no)` to load line into `current_line_` buffer
- **After editing**: Call `put_line()` to write buffer back to segment chain if modified
- **State tracking**: `current_line_no_`, `current_line_modified_` flags

### Component Architecture
- **Editor class**: Singleton pattern (signal handlers need static instance pointer)
- **Workspace switching**: Tab-like switching between primary and alternative workspaces
- **Session management**: Automatic save/restore of position and state
- **Journaling**: Keystroke recording for debugging and replay

## Key Data Structures

### Editor State
```cpp
int ncols_, nlines_;           // Terminal dimensions
int cursor_col_, cursor_line_; // Cursor position in view
std::string status_;           // Status message
std::string filename_;         // Current filename
bool cmd_mode_;                // Command mode active
bool quit_flag_;               // Exit requested
bool insert_mode_;             // Insert vs overwrite
bool quote_next_;              // ^P literal insert mode
bool filter_mode_;             // External filter entry mode
bool area_selection_mode_;     // Rectangular selection active
```

### Current Line Buffer
```cpp
std::string current_line_;        // Editing buffer
int current_line_no_;             // Which line is buffered (-1 = invalid)
bool current_line_modified_;      // Buffer needs writing back
```

### Segment Chain
```cpp
struct Segment {
    unsigned line_count;                   // Number of lines in segment
    int file_descriptor;                   // File containing data, or -1 for empty lines
    long file_offset;                      // Offset in file where data begins
    std::vector<unsigned short> line_lengths; // Byte length of each line (including \n)
};
```

### Workspaces
```cpp
class Workspace {
    Segment *head_;         // Segment chain head
    Segment *cursegm_;      // Current segment pointer
    int topline_;           // Viewport top line
    int basecol_;           // Horizontal scroll offset
    bool modified_;         // Change tracking
    Tempfile &tempfile_;    // Shared temp file
};

// Editor maintains two:
std::unique_ptr<Workspace> wksp_;      // Primary workspace
std::unique_ptr<Workspace> alt_wksp_;  // Alternative workspace
```

## Key Implementation Patterns

### Main Loop (core.cpp)
```cpp
timeout(200);  // 200ms non-blocking input
for (;;) {
    check_interrupt();
    move(cursor_line_, cursor_col_);
    int ch = journal_read_key();
    if (ch == ERR) {
        draw();  // Redraw even without input
    } else {
        journal_write_key(ch);
        handle_key(ch);
        draw();
    }
    if (quit_flag_) break;
}
```

### Current Line Editing Pattern
```cpp
// To edit line N:
get_line(N);              // Load into current_line_
current_line_ = "...";    // Edit buffer contents
current_line_modified_ = true;
put_line();               // Write back to segment chain
```

## Code Style & Conventions

### Naming
- **Private members**: underscore suffix (`ncols_`, `wksp_`, `cursor_line_`)
- **Public API**: no suffix (`get_lines()`, `topline()`)
- **Methods**: snake_case (`handle_key()`, `draw()`)
- **Fields and locals**: snake_case (`start_line`, `first_seg`)
- **Classes/Structs**: PascalCase (`Editor`, `Clipboard`, `Workspace`)

### Formatting
- **Indentation**: 4 spaces, no tabs (C++ and CMake)
- **Formatting**: `make reindent` for clang-format

### File Organization
- **Headers**: Classes, structures, constants in `.h` files
- **Implementation**: Single-responsibility `.cpp` files
- **Test files**: Comprehensive unit and integration tests in `tests/`

## Testing Framework
- **Unit tests**: Direct Editor/Workspace instantiation
- **Integration tests**: tmux-based UI testing using TmuxDriver

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sergev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
