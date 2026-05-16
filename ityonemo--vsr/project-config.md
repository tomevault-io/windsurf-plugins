---
trigger: always_on
description: This document outlines the normative coding standards for the VSR (Viewstamped Replication) codebase.
---

# VSR Coding Standards

This document outlines the normative coding standards for the VSR (Viewstamped Replication) codebase.

## Module Organization

### Structure
Organize modules with clear sections using comments:

```elixir
defmodule Vsr.ModuleName do
  @moduledoc """
  Clear, concise module documentation.
  
  Explain purpose, usage patterns, and important notes.
  Include examples where helpful.
  """
  
  # Section 0: Types and Constants
  @type t :: %__MODULE__{}
  @default_opts [key: :value]
  
  # Section 1: Public API
  def public_function(args), do: implementation
  
  # Section 2: GenServer callbacks (if applicable)
  def init(opts), do: {:ok, state}
  def handle_call(msg, from, state), do: {:reply, :ok, state}
  
  # Section 3: Private helpers
  defp helper_function(args), do: implementation
end
```

### Documentation Standards
- **All public functions** must have `@doc` strings
- **All modules** must have `@moduledoc` with clear purpose explanation
- **Protocol callbacks** should document expected behavior and return values
- **Complex algorithms** require inline comments explaining the logic
- **Include examples** in documentation where helpful

## Naming Conventions

### Modules
- **PascalCase**: `Vsr.StateMachine`, `VsrKv`
- **Descriptive names**: Module name should clearly indicate purpose
- **Namespace properly**: Use `Vsr.` prefix for core protocol modules

### Functions
- **snake_case**: `client_request_impl`, `increment_op_number`
- **Descriptive names**: Function name should clearly indicate what it does
- **Implementation suffix**: Message handlers use `_impl` suffix (`prepare_impl/2`)
- **Protocol callbacks**: Prefix with `_` (`_apply_operation`, `_get_state`)

### Variables and Atoms
- **snake_case**: `view_number`, `op_number`
- **Descriptive**: Avoid abbreviations unless they're domain-specific (VSR terms)
- **Consistent terminology**: Use VSR specification terms exactly

### Constants
- **Uppercase with underscores**: `@DEFAULT_OPTS`, `@FILTER`
- **Module attributes**: Use `@` for compile-time constants

### Module Aliases
- **Individual alias statements**: Use separate `alias` statements for each module in source files
- **❌ Avoid multi-module aliases in source code**: `alias Module.{A, B, C}`
- **✅ Use individual aliases in source files**: 
  ```elixir
  alias Module.A
  alias Module.B
  alias Module.C
  ```
- **✅ Exception**: Bracketed aliases are acceptable for command-line usage (IEx, `elixir -e "..."`)
- **Reason**: Individual aliases are clearer, easier to grep, and avoid merge conflicts in source files

## Function Patterns

### Function Definitions
```elixir
# Good - descriptive name, clear pattern matching
defp client_request_impl(%ClientRequest{operation: op, from: from}, state) do
  # implementation
end

# Bad - abbreviated name, unclear purpose  
defp handle_cr(msg, state) do
  # implementation
end
```

### Error Handling
- **Let it crash**: Follow Erlang/Elixir philosophy - don't code defensively with excessive try/catch
- **No defensive catch-all branches**: Never use `_ ->` branches in case statements to "handle" unexpected input - let it crash so problems surface in logs
- **Explicit error tuples**: `{:error, :reason}` not just `:error` for expected error conditions
- **Consistent patterns**: Always use same error tuple format
- **Meaningful error reasons**: Use descriptive atoms for error reasons
- **Crash on invalid input**: Let functions crash on invalid input rather than handling every edge case
- **No defensive logging**: Don't log warnings for "unhandled" cases - if it's truly unhandled, it should crash

```elixir
# Good - let it crash on invalid input
def fetch_operation(log, op_number) when is_integer(op_number) and op_number > 0 do
  case Log.fetch(log, op_number) do
    {:ok, entry} -> {:ok, entry}
    :error -> {:error, :not_found}
  end
end

# Good - crash immediately on invalid JSON rather than defensive error handling
def stdin_impl(stdin, _from, state) do
  json_map = JSON.decode!(stdin)  # Let it crash if invalid JSON
  message = Message.from_json_map(json_map)  # Let it crash if invalid message
  # ... process message
end

# Bad - overly defensive
def fetch_operation(log, op_number) do
  try do
    case Log.fetch(log, op_number) do
      {:ok, entry} -> {:ok, entry}
      :error -> {:error, :not_found}
    end
  rescue
    _ -> {:error, :unknown}
  end
end
```

**When to let it crash:**
- Invalid JSON input
- Missing required fields in messages  
- Type mismatches (string passed where integer expected)
- Programming errors (calling undefined functions)
- Contract violations (guards should enforce contracts)

**When to use error tuples:**
- Expected business logic failures (key not found, insufficient permissions)
- Network failures that should be retried
- Resource unavailability that can be handled gracefully

**Never add catch-all clauses for operation handlers:**
- **Protocol message handlers** like `handle_commit/2` should NOT have default `_ -> ...` clauses
- **Unknown operations indicate programming errors** - let them crash to surface bugs immediately
- **Defensive catch-alls hide problems** - if an unexpected operation arrives, it's better to crash noisily than silently ignore it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ityonemo/vsr](https://github.com/ityonemo/vsr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
