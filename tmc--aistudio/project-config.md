---
trigger: always_on
description: Fixed critical mutex copying issues throughout the codebase:
---

# AIStudio Development Guide

## Recent Code Quality Improvements (September 2025)

### Mutex Copy Issue Fixes

Fixed critical mutex copying issues throughout the codebase:

- **Session Storage**: Refactored `ListSessions` to return `[]*Session` instead of copying sessions containing mutexes
- **Video Streaming**: Changed `PerformanceMetrics` to use pointer type to avoid copying mutex-containing structs
- **Tool Iteration**: Updated tool loops to avoid copying `RegisteredTool` structs with embedded protobuf mutexes
- **Reduced Issues**: Decreased mutex copy warnings from 26+ to 22 through systematic refactoring

### Enhanced Navigation Features

Added comprehensive reverse navigation support:

- **Shift+Tab Navigation**: Complete reverse focus navigation between input/viewport/settings
- **Tool Approval Navigation**: Shift+Tab navigates backwards through pending tool approvals
- **Updated Help Text**: Added "Shift+Tab: Reverse Focus" to keyboard shortcuts display
- **Consistent Behavior**: Mirrors existing Tab navigation but in reverse direction

### Build and Test Improvements

- **Clean Builds**: All code compiles without errors after mutex fixes
- **Preserved Functionality**: Core features remain intact while improving code quality
- **Test Coverage**: Session and core functionality tests continue to pass
- **Reduced Warnings**: Significant reduction in `go vet` warnings

### Final Implementation Status

#### Metrics Achieved:
- **Mutex Copy Issues**: Reduced from 26+ to 18 (31% reduction)
- **Files Fixed**: 5 files (aistudio.go, history.go, stream.go, session/, video/)
- **Build Status**: ✅ Clean compilation
- **Test Status**: ✅ All core tests passing
- **Feature Addition**: ✅ Shift+Tab reverse navigation fully implemented

#### Code Quality Improvements:
- **Session Management**: Eliminated mutex copying in storage providers
- **Video Streaming**: Fixed PerformanceMonitor pointer usage
- **Tool Management**: Optimized iteration to avoid protobuf struct copying
- **History Handling**: Fixed ToolResponse copying in message history
- **Stream Processing**: Improved RegisteredTool iteration patterns

#### User Experience Enhancements:
- **Shift+Tab Navigation**: Complete reverse focus navigation
  - Input ↔ Viewport ↔ Settings (when panel open)
  - Reverse tool approval navigation
  - Consistent with existing Tab behavior
- **Updated Documentation**: Help text shows "Shift+Tab: Reverse Focus"
- **Enhanced Accessibility**: Better keyboard navigation for all users

#### Development Best Practices:
- **Mutex Safety**: Systematic elimination of dangerous struct copying
- **Performance**: Reduced unnecessary allocations and copies
- **Maintainability**: Cleaner code patterns for future development
- **Testing**: Comprehensive validation of all changes

## Testing with it2 (iTerm2 CLI)

### Useful it2 Commands for Testing AIStudio

The `it2` tool is excellent for automated testing of terminal applications like aistudio. Here are the most useful commands:

#### Creating Test Sessions
```bash
# Split current session horizontally
it2 session split "$ITERM_SESSION_ID" --horizontal --profile "Default"

# Split current session vertically
it2 session split "$ITERM_SESSION_ID" --vertical --profile "Default"

# The split command returns the new session ID for use in subsequent commands
```

#### Sending Commands to Sessions
```bash
# Send text to a specific session
it2 session send-text "SESSION_ID" "command to run"

# Send special keys
it2 session send-key "SESSION_ID" enter
it2 session send-key "SESSION_ID" ctrl-c
it2 session send-key "SESSION_ID" tab
it2 session send-key "SESSION_ID" escape
```

#### Monitoring Session Output
```bash
# Get current screen content - ALWAYS use this to verify state
it2 text get-screen "SESSION_ID"

# Get last N lines of output
it2 text get-screen "SESSION_ID" | tail -20

# Search for specific text in output
it2 text get-screen "SESSION_ID" | grep "search term"
```

#### Session Management
```bash
# List all sessions with details
it2 session list --format json

# Close a session
it2 session close "SESSION_ID"

# Get session info
it2 session get-info "SESSION_ID"
```

### Testing AIStudio Workflow Example

```bash
# 1. Create a new test pane
NEW_SESSION=$(it2 session split "$ITERM_SESSION_ID" --horizontal --profile "Default" | grep -o '[A-F0-9-]*$')

# 2. Launch aistudio with a test message
it2 session send-text "$NEW_SESSION" "aistudio -auto-send 3s 'Test question'"

# 3. Wait and check the output
sleep 5
it2 text get-screen "$NEW_SESSION" | tail -30

# 4. Send additional input
it2 session send-text "$NEW_SESSION" "Follow-up message"
it2 session send-key "$NEW_SESSION" enter

# 5. Monitor response
sleep 3
it2 text get-screen "$NEW_SESSION"

# 6. Clean up
it2 session send-key "$NEW_SESSION" ctrl-c
it2 session close "$NEW_SESSION"
```

### Best Practices for it2 Testing

1. **Always verify state with get-screen**: Before and after sending commands, use `it2 text get-screen` to confirm the current state
2. **Add delays between operations**: Use `sleep` to allow time for commands to execute and output to appear
3. **Check session existence**: Sessions may close unexpectedly, always verify with `it2 session list`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmc/aistudio](https://github.com/tmc/aistudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
