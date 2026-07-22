---
trigger: always_on
description: This file provides guidance to AI agents (such as Gemini CLI, Claude Code, etc.) when working with code in
---

# AGENTS.md

This file provides guidance to AI agents (such as Gemini CLI, Claude Code, etc.) when working with code in
this repository. Treat it as the canonical reference for workflows, tooling, and expectations; update it
directly whenever new agent instructions are required.

## Mission
- Pull accurate information and project facts for users by driving available tools rather than reasoning from
  scratch.
- Keep workflow transparent: explain what you did, why it matters, and what the user should consider next.
- Leave the repository tidy; only touch files that advance the request. If you find other opportunities for
  improvement along the way, mention them in the form of a prompt the user can use later.

## Running External Commands

- When running shell commands on this project, use a Bash login shell where possible: run commands via `bash
  -lc '<command>'` (or the platform's equivalent) rather than relying on the default shell.
- If practical, use workdir features of the agent rather than relying on `cd` in the command string.

## Environment & Execution

Prefer project-local tools and scripts (for example, `bin/` scripts, `package.json` scripts, Makefile targets)
instead of ad-hoc one-off commands when building, testing, or running the project.

- Prefer `rg`/`rg --files` for searches; switch only if ripgrep is unavailable.
- Planning tool: when your AI client provides a planning tool, skip it for trivial chores; otherwise create a
  multi-step plan and keep it updated as you work (max one `in_progress` step).
- When moving or renaming tracked files, use `git mv` (or `git mv -k`) instead of plain `mv` so history stays
  intact.
- If `git mv` is blocked by the sandbox or `.git/index.lock` is not writable, stop and leave the tracked file
  in place. Do not simulate the move with delete-plus-add; ask the user to run the `git mv` command outside
  the sandbox.

## Ruby Guidelines
- Use the project's Ruby version.
- Prefer `bundle exec` for project tools.
- Prefer binstubs when present (e.g., `bin/rspec`, `bin/rubocop`).
- Choose the control-flow form that is clearest on first read.
- For simple single-condition methods, especially readers, predicates, and manifest/file-loading methods, an
  `if` block is often clearer than `return unless` / `return if`.
- Avoid early returns for simple Ruby branches when a direct conditional expression is clearer.
- If two early-return orderings are equally complex, prefer an `if` statement
  instead of structuring the branch around early returns.
- Do not introduce early returns inside simple value-producing branches. For methods that choose between
  strings, symbols, booleans, or small result objects, prefer a direct `if` / `elsif` / `else` expression
  unless an early return clearly isolates an exceptional case or simplifies a complex main path.
- Prefer early returns when they isolate exceptional cases, invalid input, or meaningfully simplify the main
  path.
- After editing Ruby, do a quick readability pass on any guard clauses you introduced and rewrite them if
  another form is clearer.
- If readability and RuboCop conflict, use the clearest lint-compliant form.
- Before editing, inspect:
  - `Gemfile`
  - `Gemfile.lock`
  - `*.gemspec`
  - `.rubocop.yml`
  - `.rspec`
- Run the smallest relevant test first.
- Preserve existing Ruby idioms.
- Do not add gems unless necessary.

## Project Overview

**WifiWand** (gem: `wifi-wand`) is a Ruby gem that provides cross-platform WiFi management for Mac and Ubuntu systems.
It operates through both command-line interface and interactive shell modes, 
using OS-specific utilities under the hood while presenting a unified API.

## Development Commands

### Testing

Rake tasks select the test scope:
```bash
bundle exec rake test:safe          # default safe suite (CI-safe)
bundle exec rake test:read_only     # + real-env read-only tests
bundle exec rake test:all           # + real-env read-write tests
```

Env vars are orthogonal modifiers — combine them with any rake task or plain rspec:
```bash
WIFIWAND_VERBOSE=true bundle exec rake test:safe   # show underlying OS commands

# Run a specific file directly
bundle exec rspec spec/wifi_wand/platforms/ubuntu/model_spec.rb

# Run a targeted real-env rake task
bundle exec rake 'test:real[./spec/wifi_wand/platforms/mac/model_spec.rb]'
```

When using bracketed Rake task arguments such as `test:real[spec/foo_spec.rb]`, make the shell rule explicit
in documentation and examples: quotes are optional in `bash` but required in `zsh`. `zsh` treats unquoted
brackets as glob syntax and fails before Rake runs. Quoting remains the portable default.

### Development Setup
```bash
# Install dependencies
bundle install

# Set up git hooks (run once after cloning)
bin/setup-hooks

# Build gem locally
bundle exec rake build

# Before any gem release, inspect the built artifact's file list and confirm it still includes every
# required runtime file, executable, helper asset, and user-facing document while excluding
# maintainer-only tooling. Do not assume the gemspec allowlist is correct without verifying the built gem.
# Prefer an explicit payload check such as:
#   tar -xOf pkg/wifi-wand-<version>.gem data.tar.gz | tar -tz
# or:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keithrbennett/wifiwand](https://github.com/keithrbennett/wifiwand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
