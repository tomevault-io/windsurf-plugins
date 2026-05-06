---
trigger: always_on
description: This plugin exposes JetBrains IDE debugger capabilities through MCP (Model Context Protocol), enabling AI agents to programmatically debug applications.
---

# JetBrains Debugger MCP Plugin

This plugin exposes JetBrains IDE debugger capabilities through MCP (Model Context Protocol), enabling AI agents to programmatically debug applications.

## Quick Start

```
IMPORTANT: When debugging, prefer using jetbrains-debugger MCP tools to interact with the IDE debugger.
```

## Typical Debugging Workflow

1. **Discover configurations**: `list_run_configurations` to see available run/debug configs
2. **Start debugging**: `start_debug_session` with a configuration that has `can_debug: true`
3. **Set breakpoints**: `set_breakpoint` at locations of interest
4. **Resume and wait**: `resume_execution` then `wait_for_pause` with a timeout to block until a breakpoint is hit
5. **Inspect state**: `wait_for_pause` returns variables, stack trace, and source context automatically. Use `get_debug_session_status` if you need to re-inspect without waiting.
6. **Evaluate expressions**: `evaluate_expression` to test hypotheses
7. **Step through code**: `step_over`, `step_into`, `step_out` as needed
8. **Repeat steps 5-7** until the issue is found

## Tool Reference

### Session Management
| Tool | Description |
|------|-------------|
| `list_run_configurations` | List available run configurations |
| `execute_run_configuration` | Run a configuration (debug or run mode) |
| `list_debug_sessions` | List active debug sessions |
| `start_debug_session` | Start a new debug session |
| `stop_debug_session` | Stop a debug session |
| `get_debug_session_status` | Get comprehensive session state (variables, stack, source) |

### Breakpoints
| Tool | Description |
|------|-------------|
| `list_breakpoints` | List all breakpoints |
| `set_breakpoint` | Set a line breakpoint (supports conditions, log messages) |
| `remove_breakpoint` | Remove a breakpoint by ID |

### Execution Control
| Tool | Description |
|------|-------------|
| `resume_execution` | Resume from paused state |
| `pause_execution` | Pause running execution |
| `step_over` | Step to next line (over function calls) |
| `step_into` | Step into function call |
| `step_out` | Step out of current function |
| `run_to_line` | Run until specific line is reached |
| `wait_for_pause` | Wait for session to pause (breakpoint, exception, manual). Returns full status. |

### Inspection
| Tool | Description |
|------|-------------|
| `get_stack_trace` | Get call stack for a thread |
| `select_stack_frame` | Change stack frame context |
| `list_threads` | List all threads |
| `get_variables` | Get variables in current frame |
| `set_variable` | Modify variable value at runtime |
| `evaluate_expression` | Evaluate expression or code fragment |
| `get_source_context` | Get source code around a location |

## Best Practices

### Use `get_debug_session_status` First
When paused at a breakpoint, call `get_debug_session_status` before other inspection tools. It returns:
- Current location (file, line, method)
- Variables in scope
- Stack trace summary
- Source code context
- Breakpoint that was hit

This reduces round-trips compared to calling `get_variables`, `get_stack_trace`, and `get_source_context` separately.

### Conditional Breakpoints for Loops
When debugging loops or frequently-called functions, use conditional breakpoints:
```json
{
  "file_path": "/src/Calculator.java",
  "line": 42,
  "condition": "i == 100"
}
```

### Log Breakpoints for Tracing
Use log breakpoints (tracepoints) to trace execution without stopping:
```json
{
  "file_path": "/src/Calculator.java",
  "line": 42,
  "log_message": "Value of x: {x}, items.size(): {items.size()}",
  "suspend_policy": "none"
}
```

**Log Message Syntax**: Use `{expression}` placeholders in your log message. They are automatically transformed to language-specific expressions:

| Language | Input | Transformed To |
|----------|-------|----------------|
| Java | `"x={x}, y={y}"` | `"x=" + (x) + ", y=" + (y)` |
| Kotlin | `"x={x}, y={y}"` | `"x=$x, y=$y"` |
| Python | `"x={x}, y={y}"` | `f"x={x}, y={y}"` |
| JavaScript/TS | `"x={x}, y={y}"` | `` `x=${x}, y=${y}` `` |

You can also pass raw language-specific expressions directly (e.g., `"\"x=\" + x"` for Java) - they pass through unchanged if no `{...}` placeholders are detected.

### Evaluate Before Modifying
Before using `set_variable`, use `evaluate_expression` to preview the change:
```json
{
  "expression": "calculateNewValue(currentValue)",
  "allow_side_effects": false
}
```

### Handle Multiple Projects
When multiple projects are open in the IDE, always specify `projectPath`:
```json
{
  "projectPath": "/Users/dev/my-project",
  "file_path": "/src/Main.java",
  "line": 10
}
```

If omitted with multiple projects, tools return an error listing available projects.

## Common Patterns

### Find Why a Value is Wrong
```
1. set_breakpoint at the line where the wrong value is used
2. start_debug_session
3. resume_execution (wait for breakpoint)
4. get_debug_session_status (see all variables)
5. evaluate_expression to test the calculation
6. step_over/step_into to trace the logic
```

### Debug a Specific Iteration
```
1. set_breakpoint with condition "i == 50"
2. start_debug_session
3. resume_execution
4. Debugger stops only when i equals 50
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hechtcarmel/jetbrains-debugger-mcp-plugin](https://github.com/hechtcarmel/jetbrains-debugger-mcp-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
