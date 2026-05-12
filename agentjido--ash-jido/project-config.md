---
trigger: always_on
description: - Build: `mix compile`
---

# AGENTS.md

## Commands

- Build: `mix compile`
- Quality: `mix quality` (format check, compile warnings, credo, dialyzer, doctor)
- Test all: `mix test`
- Test single: `mix test test/path/to/test.exs:line_number`
- Format: `mix format`
- Deps: `mix deps.get`
- Shell: `iex -S mix`

## Architecture

AshJido is a thin bridge library (~400 LOC) that generates `Jido.Action` modules from Ash resources at compile time.

### Core Modules

| Module | Purpose |
|--------|---------|
| `AshJido` | Spark DSL extension entry point |
| `AshJido.Resource.Dsl` | DSL section definition (`jido do ... end`) |
| `AshJido.Resource.Transformers.GenerateJidoActions` | Compile-time transformer |
| `AshJido.Generator` | Generates Jido.Action module AST (internal) |
| `AshJido.Mapper` | Converts Ash structs to maps (internal) |
| `AshJido.TypeMapper` | Maps Ash types to NimbleOptions (internal) |
| `AshJido.Error` | Converts Ash errors to Jido errors (public) |

### Key Behaviors

- Domain is **required** in context - raises `ArgumentError` if missing
- Update/destroy actions require `id` parameter
- Default naming: `create_user`, `list_users`, `update_user`, `delete_user`
- `output_map?: true` (default) converts Ash structs to plain maps

### Dependencies

- `{:ash, "~> 3.12"}` - Ash Framework
- `{:jido, "~> 1.1"}` - Jido agent framework
- `{:jido_action, "~> 1.3"}` - Jido action system
- `{:splode, "~> 0.2"}` - Error handling

## Code Style

- Use `mix format` for formatting
- Internal modules use `@moduledoc false`
- Public API: `AshJido` extension + `AshJido.Error`
- Tests in `test/` using ExUnit

## Documentation

- `README.md` - Quick start
- `guides/getting-started.md` - Comprehensive usage
- `guides/ash-jido-demo.livemd` - Interactive demo
- `usage-rules.md` - AI/LLM patterns



<!-- usage-rules-start -->
<!-- usage-rules-header -->
# Usage Rules

**IMPORTANT**: Consult these usage rules early and often when working with the packages listed below. 
Before attempting to use any of these packages or to discover if you should use them, review their 
usage rules to understand the correct patterns, conventions, and best practices.
<!-- usage-rules-header-end -->

<!-- usage_rules-start -->
## usage_rules usage
_A dev tool for Elixir projects to gather LLM usage rules from dependencies
_

## Using Usage Rules

Many packages have usage rules, which you should *thoroughly* consult before taking any
action. These usage rules contain guidelines and rules *directly from the package authors*.
They are your best source of knowledge for making decisions.

## Modules & functions in the current app and dependencies

When looking for docs for modules & functions that are dependencies of the current project,
or for Elixir itself, use `mix usage_rules.docs`

```
# Search a whole module
mix usage_rules.docs Enum

# Search a specific function
mix usage_rules.docs Enum.zip

# Search a specific function & arity
mix usage_rules.docs Enum.zip/1
```


## Searching Documentation

You should also consult the documentation of any tools you are using, early and often. The best 
way to accomplish this is to use the `usage_rules.search_docs` mix task. Once you have
found what you are looking for, use the links in the search results to get more detail. For example:

```
# Search docs for all packages in the current application, including Elixir
mix usage_rules.search_docs Enum.zip

# Search docs for specific packages
mix usage_rules.search_docs Req.get -p req

# Search docs for multi-word queries
mix usage_rules.search_docs "making requests" -p req

# Search only in titles (useful for finding specific functions/modules)
mix usage_rules.search_docs "Enum.zip" --query-by title
```


<!-- usage_rules-end -->
<!-- usage_rules:elixir-start -->
## usage_rules:elixir usage
# Elixir Core Usage Rules

## Pattern Matching
- Use pattern matching over conditional logic when possible
- Prefer to match on function heads instead of using `if`/`else` or `case` in function bodies

## Error Handling
- Use `{:ok, result}` and `{:error, reason}` tuples for operations that can fail
- Avoid raising exceptions for control flow
- Use `with` for chaining operations that return `{:ok, _}` or `{:error, _}`

## Common Mistakes to Avoid
- Elixir has no `return` statement, nor early returns. The last expression in a block is always returned.
- Don't use `Enum` functions on large collections when `Stream` is more appropriate
- Avoid nested `case` statements - refactor to a single `case`, `with` or separate functions
- Don't use `String.to_atom/1` on user input (memory leak risk)
- Lists and enumerables cannot be indexed with brackets. Use pattern matching or `Enum` functions
- Prefer `Enum` functions like `Enum.reduce` over recursion
- When recursion is necessary, prefer to use pattern matching in function heads for base case detection
- Using the process dictionary is typically a sign of unidiomatic code
- Only use macros if explicitly requested
- There are many useful standard library functions, prefer to use them where possible

## Function Design
- Use guard clauses: `when is_binary(name) and byte_size(name) > 0`
- Prefer multiple function clauses over complex conditional logic
- Name functions descriptively: `calculate_total_price/2` not `calc/2`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentjido/ash_jido](https://github.com/agentjido/ash_jido) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
