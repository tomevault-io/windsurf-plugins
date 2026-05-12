---
trigger: always_on
description: **GettextSigils** provides a `~t` sigil for Elixir's Gettext, replacing verbose `gettext`/`dgettext`/`pgettext` calls with concise, interpolation-aware syntax.
---

# Agent Instructions

## Library Overview

**GettextSigils** provides a `~t` sigil for Elixir's Gettext, replacing verbose `gettext`/`dgettext`/`pgettext` calls with concise, interpolation-aware syntax.

### Core Modules

- **`GettextSigils`** — Main entry point. `use GettextSigils, backend: MyApp.Gettext` imports the `~t` sigil and configures domain, context, and modifiers.
- **`GettextSigils.Sigil`** — Implements `sigil_t/2`. Parses interpolation, resolves modifiers, and emits `dpgettext/4` calls at compile time.
- **`GettextSigils.Interpolation`** — Converts `#{expr}` to `%{key}` placeholders with automatic key derivation (e.g. `user.name` → `user_name`, `String.upcase(x)` → `string_upcase`). Supports explicit keys via `key = expr` syntax.
- **`GettextSigils.Options`** — Validates compile-time configuration: `:domain`, `:context`, and `:modifiers` (single lowercase letter keys mapping to domain/context overrides).

### Key Features

- **Automatic interpolation**: `~t"Hello, #{user.name}!"` → `dpgettext(backend, "default", nil, "Hello, %{user_name}!", user_name: user.name)`
- **Modifiers**: Single-letter suffixes override domain/context per-call (e.g. `~t"Error"e` uses the `errors` domain)
- **Modifier composition**: Multiple modifiers apply left-to-right, last wins (`~t"msg"em` ≠ `~t"msg"me`)
- **Compile-time only**: All transformation happens via macros — zero runtime overhead
- **Pluralization**: `~t"#{count} item(s)"N` — the `N` modifier uses the message as both `msgid` and `msgid_plural`, emits `dpngettext/6`

### Architecture

All processing is compile-time macro expansion. No processes, no state. Each module using `GettextSigils` stores its configuration in the `@__gettext_sigils__` module attribute. The sigil macro reads this attribute to resolve domain, context, and modifiers when generating the Gettext call.

## Commits

Use [Conventional Commits](https://www.conventionalcommits.org/) — release-please automates releases from them.

- **`feat:`** and **`fix:`** trigger a release and appear in the changelog (patch bump while pre-1.0)
- `docs:`, `chore:`, `ci:`, `test:`, `refactor:` — no release triggered
- **Breaking changes**: add `!` after type (e.g. `feat!:`) or a `BREAKING CHANGE:` footer — bumps minor while pre-1.0

# Guidelines

- Always run `mix precommit` before committing
- For callouts in documentation (README, guides, moduledocs), use [ExDoc admonitions](https://hexdocs.pm/ex_doc/readme.html#admonition-blocks) instead of plain blockquotes or bold "Note:" prefixes. Example:

  ```markdown
  > #### Reserved letters {: .info}
  >
  > Custom modifiers must use lowercase letters (`a`–`z`).
  ```

  Supported classes: `.info`, `.tip`, `.warning`, `.error`, `.neutral`.

# Usage Rules

**IMPORTANT:** Prefer Tidewave MCP tools for reading documentation if possible.

<!-- usage-rules-start -->
<!-- usage_rules-start -->
## usage_rules usage
_A config-driven dev tool for Elixir projects to manage AGENTS.md files and agent skills from dependencies_

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
- `%{}` matches ANY map, not just empty maps. Use `map_size(map) == 0` guard to check for truly empty maps

## Error Handling
- Use `{:ok, result}` and `{:error, reason}` tuples for operations that can fail
- Avoid raising exceptions for control flow
- Use `with` for chaining operations that return `{:ok, _}` or `{:error, _}`

## Common Mistakes to Avoid
- Elixir has no `return` statement, nor early returns. The last expression in a block is always returned.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zebbra/gettext_sigils](https://github.com/zebbra/gettext_sigils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
