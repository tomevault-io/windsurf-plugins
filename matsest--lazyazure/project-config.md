---
trigger: always_on
description: This document provides guidelines and lessons learned for agents working on the LazyAzure codebase.
---

# LazyAzure - Agent Guidelines

This document provides guidelines and lessons learned for agents working on the LazyAzure codebase.

## Project Overview

LazyAzure is a TUI (Terminal User Interface) application for Azure resource management, inspired by lazydocker. It uses:
- **gocui** for the terminal interface
- **Azure SDK for Go** for Azure API interactions
- **DefaultAzureCredential** for authentication (supports multiple auth methods)

## Critical Guidelines

### 1. TUI Development with gocui

#### Event Loop and Threading
- **CRITICAL**: Never hold locks (mutex) when calling gocui operations
- **CRITICAL**: Never call `gui.g.Update()` or `gui.g.UpdateAsync()` while holding a lock
- Use `go func()` for async operations, then use `gui.g.UpdateAsync()` for UI updates
- `UpdateAsync()` is safer than `Update()` as it doesn't spawn a goroutine

#### Cursor Handling
- Use `v.SetCursor(x, y)` directly instead of `v.TextArea.MoveCursorDown/Up`
- Get cursor position with `v.Cursor()` which returns (x, y)
- Don't mix cursor tracking with origin/offset calculations unless necessary

#### View Management
- Use `gocui.IsUnknownView(err)` to check if a view already exists
- Views are identified by string names (e.g., "subscriptions", "resourcegroups")
- Set current view with `gui.g.SetCurrentView("viewname")`

### 2. Mutex Best Practices

#### Deadlock Prevention
```go
// ❌ WRONG: Holding lock while calling another function that needs lock
func (gui *Gui) nextLine() {
    gui.mu.Lock()
    defer gui.mu.Unlock()
    gui.selectItem()  // selectItem() also needs Lock = DEADLOCK!
}

// ✅ CORRECT: Release lock before calling other methods
func (gui *Gui) nextLine() {
    gui.mu.RLock()
    count := len(gui.items)
    gui.mu.RUnlock()
    
    // ... do work ...
    
    gui.selectItem()  // Now safe to acquire Lock
}
```

#### Lock Hierarchy
1. `RLock()` for reading data length/slices
2. Do work without locks
3. `Lock()` only for updating state
4. Never hold locks during I/O or UI operations

### 3. Debug Logging

The application supports opt-in debug logging:

```bash
# Enable debug logging
LAZYAZURE_DEBUG=1 ./lazyazure

# View logs
cat ~/.lazyazure/debug.log
```

**When to add logging:**
- Entry/exit of complex functions
- Before/after async operations
- When acquiring/releasing locks
- Error conditions
- User actions (key presses, selections)

**Implementation:**
```go
import "github.com/matsest/lazyazure/pkg/utils"

// Logging is a no-op if LAZYAZURE_DEBUG is not set
utils.Log("message: %s", value)
```

**Debug Logging Privacy:**

When `LAZYAZURE_DEBUG=1` is enabled, logs are written to `~/.lazyazure/debug.log`. 
The logs are designed to be safe to share for debugging while protecting sensitive information:

**What IS logged:**
- Application flow and lifecycle events (initialization, view setup, etc.)
- UI interactions (panel switches, keybindings, clicks)
- Performance metrics (operation timing, result counts)
- Error messages and types
- Authentication success/failure (without identity details)
- Search activity (character counts, not content)

**What is NOT logged:**
- User Principal Names (UPN/email addresses)
- Display names
- Tenant IDs or Object IDs
- Resource IDs (full Azure resource IDs, subscription IDs, resource group IDs)
- Search query content
- Full JWT tokens or credentials

**Safe alternatives for debugging:**
- Use presence indicators: `hasSub := savedSubID != ""` then log `hasSub=true/false`
- Log indices/positions instead of IDs: `found at index 5` instead of `found ID xyz`
- Log counts and lengths: `loaded 20 subscriptions` instead of listing them
- Use anonymized identifiers: `resource-1`, `rg-A` if item identification is needed

The goal is to provide enough context to diagnose issues without exposing 
personally identifiable information or organizational data.

### 3.1 Performance Metrics

The application includes comprehensive performance metrics for validating optimizations:

**Metrics Tracked:**
- **Cache hits/misses**: Track cache efficiency with hit rate percentage
- **API call timing**: Average duration of Azure API calls
- **Cache size**: Current and maximum sizes for all cache tiers
- **Evictions**: Count of cache evictions due to size limits

**Implementation:**
```go
import "github.com/matsest/lazyazure/pkg/utils"

// Record cache hit/miss (automatic in PreloadCache)
utils.GetMetrics().RecordCacheHit()
utils.GetMetrics().RecordCacheMiss()

// Record API call timing
record := utils.StartAPITimer("ListSubscriptions")
defer record(err)  // Records duration and error status

// Get current stats
stats := utils.GetMetrics().GetStats()
fmt.Printf("Hit rate: %.1f%%\n", stats.CacheHitRate)

// Log metrics to debug output
utils.LogMetrics()
```

**Viewing Metrics:**
When `LAZYAZURE_DEBUG=1` is enabled, metrics are automatically logged:
```
[2026-04-02 10:15:30.123] [METRICS] Cache: 150 hits, 50 misses (75.0% hit rate), 10 evictions | API: 25 calls, 0 errors, avg 145ms | Size: RG=10/20, Res=50/100, FullRes=25/50
```

The metrics are stored in a global singleton (`utils.GetMetrics()`) and updated automatically by:
- `PreloadCache` for cache operations (hit/miss, size, evictions)
- Azure client methods for API call timing (via `StartAPITimer`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matsest/lazyazure](https://github.com/matsest/lazyazure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
