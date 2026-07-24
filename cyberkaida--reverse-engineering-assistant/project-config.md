---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the decompiler tools package.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the decompiler tools package.

## Package Overview

The `reva.tools.decompiler` package provides MCP tools for function decompilation and variable manipulation. It handles Ghidra's decompiler interface and implements read-before-modify tracking to ensure safe variable operations.

## Registered Tools

### Core Decompilation Tools
- `get-decompilation` - Get decompiled function with line range support, optional assembly sync, comments, incoming references, and caller/callee lists
- `search-decompilation` - Search regex patterns across all function decompilations
- `rename-variables` - Rename variables in decompiled functions (with diff)
- `change-variable-datatypes` - Change variable data types (with diff)
- `set-decompilation-comment` - Set comment at specific decompilation line

### Bulk Decompilation Tools
- `get-callers-decompiled` - Bulk decompile all functions that call a target (with pagination and call site highlighting)
- `get-referencers-decompiled` - Bulk decompile all functions that reference an address/symbol (handles code and data refs)

## Critical Implementation Patterns

### Decompiler Interface Management

**Use helper methods for consistent decompiler lifecycle**:
```java
// Create and configure decompiler (returns null on failure)
DecompInterface decompiler = createConfiguredDecompiler(program, "tool-name");
if (decompiler == null) {
    return createErrorResult("Failed to initialize decompiler");
}

try {
    // Use decompiler
    DecompilationAttempt attempt = decompileFunctionSafely(decompiler, function, "tool-name");
    if (!attempt.success()) {
        return createErrorResult(attempt.errorMessage());
    }
    // Process results...
} finally {
    decompiler.dispose(); // CRITICAL - prevents memory leaks
}
```

### Timeout Management
Use configured timeouts via helper methods:
```java
// Create timeout monitor from config
private TaskMonitor createTimeoutMonitor() {
    ConfigManager configManager = RevaInternalServiceRegistry.getService(ConfigManager.class);
    int timeoutSeconds = configManager.getDecompilerTimeoutSeconds();
    return TimeoutTaskMonitor.timeoutIn(timeoutSeconds, TimeUnit.SECONDS);
}

// Check if timed out
private boolean isTimedOut(TaskMonitor monitor) {
    return monitor.isCancelled();
}
```

### Safe Decompilation Pattern
**Use DecompilationAttempt wrapper for consistent error handling**:
```java
// Returns success/failure with error message
private record DecompilationAttempt(
    DecompileResults results,
    String errorMessage,
    boolean success
) {
    static DecompilationAttempt success(DecompileResults results) {
        return new DecompilationAttempt(results, null, true);
    }

    static DecompilationAttempt failure(String message) {
        return new DecompilationAttempt(null, message, false);
    }
}

// Use in tool handlers
DecompilationAttempt attempt = decompileFunctionSafely(decompiler, function, "tool-name");
if (!attempt.success()) {
    return createErrorResult(attempt.errorMessage());
}
```

### Read-Before-Modify Pattern
**Enforces that functions must be read via get-decompilation before modification**:
```java
// Tracking map with 30-minute expiry
private final Map<String, Long> readDecompilationTracker = new ConcurrentHashMap<>();
private static final long READ_TRACKING_EXPIRY_MS = TimeUnit.MINUTES.toMillis(30);

// Track when function is read (done automatically in get-decompilation)
String functionKey = programPath + ":" + AddressUtil.formatAddress(function.getEntryPoint());
readDecompilationTracker.put(functionKey, System.currentTimeMillis());

// Check before allowing modification
private boolean hasReadDecompilation(String functionKey) {
    Long lastReadTime = readDecompilationTracker.get(functionKey);
    if (lastReadTime == null) {
        return false;
    }
    long expiryThreshold = System.currentTimeMillis() - READ_TRACKING_EXPIRY_MS;
    return lastReadTime > expiryThreshold;
}

// Validate in modify tools
if (!hasReadDecompilation(functionKey)) {
    return createErrorResult("You must read the decompilation for function '" +
        function.getName() + "' using get-decompilation tool before making changes.");
}
```

**Cleanup on program close**:
```java
@Override
public void programClosed(Program program) {
    super.programClosed(program);
    String programPath = program.getDomainFile().getPathname();
    // Remove entries for closed program using thread-safe removeIf
    readDecompilationTracker.entrySet().removeIf(
        entry -> entry.getKey().startsWith(programPath + ":")
    );
}
```

## Variable Manipulation Patterns

### Using HighFunctionDBUtil for Persistence
**ALWAYS use HighFunctionDBUtil.updateDBVariable() for persistent variable changes**:
```java
// Get the high-level representation
HighFunction highFunction = attempt.results().getHighFunction();
if (highFunction == null) {
    return createErrorResult("Could not get high-level function representation");
}

// Update variable in database (name and/or datatype)
HighFunctionDBUtil.updateDBVariable(symbol, newName, newDataType, SourceType.USER_DEFINED);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyberkaida/reverse-engineering-assistant](https://github.com/cyberkaida/reverse-engineering-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
