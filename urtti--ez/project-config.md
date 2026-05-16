---
trigger: always_on
description: A macOS CLI tool that stores terminal commands as short aliases. Aliases are stored per-directory in `.ez_cli.json` files. Built with Swift 6 and swift-argument-parser.
---

# ez CLI — Development Guide

## What is ez?

A macOS CLI tool that stores terminal commands as short aliases. Aliases are stored per-directory in `.ez_cli.json` files. Built with Swift 6 and swift-argument-parser.

## Build & Test

```bash
swift build                  # Debug build → .build/debug/ez
swift build -c release       # Release build → .build/release/ez
./acceptance-test.sh         # Run all automated tests (builds first)
./acceptance-test-interactive.sh  # Manual tests for TTY/interactive features
```

## Testing

All testing is done via `acceptance-test.sh` — there are no Swift XCTest files. The script builds the binary, runs it in an isolated temp directory, and asserts on output.

**Running tests:** `./acceptance-test.sh`

**How tests work:**
- The script defines an `ez()` wrapper that calls the built binary and captures stdout+stderr
- Helper functions: `assert_contains` (grep match), `assert_equals` (exact match), `assert_exit_code`
- Tests run in a fresh temp directory (cleaned up on exit via trap)
- Pass/fail counts are tracked; script exits 1 if any test fails

**Adding new tests:** Append to `acceptance-test.sh` before the `# Summary` section. Follow the existing pattern:
```bash
ez add myalias "echo something {1}"
output=$(ez myalias arg1)
assert_contains "$output" "something arg1" "description of what is being tested"
```

**Test categories covered:** version/help, add/remove/list, alias execution, parallel mode, shell expansion, sequential commands, parameter substitution, extra argument overflow, secret placeholders, add-secret validation, error cases.

There is also `acceptance-test-interactive.sh` for manual verification of TTY features (vim, less, signal handling) that can't be automated.

## Project Structure

```
ezcli/
  ez.swift              # Entry point, signal handling, command routing
  Alias.swift           # Alias model: commands, execution type, parameter substitution, secrets
  AliasCollection.swift # Alias storage: JSON persistence, add/remove/lookup
  Keychain.swift        # KeychainManager: Apple Keychain read/write/delete via Security framework
  Scope.swift           # Local vs global scope, file paths, test isolation
  SystemActions.swift   # Process execution via posix_spawn, parallel via TaskGroup
  TerminalOutput.swift  # ANSI formatting, timing, error output
  commands/
    Add.swift           # `ez add` — creates aliases (-p for parallel, -d for description)
    AddSecret.swift     # `ez add-secret` — stores secrets in Keychain
    Remove.swift        # `ez remove` — deletes aliases
    RemoveSecret.swift  # `ez remove-secret` — removes secrets from Keychain
    List.swift          # `ez list` — shows aliases (-v for verbose)
    Execute.swift       # Stub; actual execution is in ez.swift
    InstallCompletions.swift    # Adds zsh completions to ~/.zshrc
    UninstallCompletions.swift  # Removes zsh completions from ~/.zshrc
```

## Key Architecture

- **Process execution** uses `posix_spawn` directly (not Foundation's `Process`) with TTY passthrough via inherited file descriptors. This enables interactive commands (vim, less, ssh).
- **Signal forwarding**: SIGINT, SIGTERM, SIGQUIT, SIGTSTP, SIGCONT are forwarded to child processes via a `@MainActor` PID set.
- **Parallel mode** (`-p` flag) spawns commands concurrently using Swift `TaskGroup`.
- **Command routing**: Subcommands (add/remove/list) go through ArgumentParser. Alias execution is custom-routed in `ez.swift main()`.
- **Shell**: All commands execute via `/bin/zsh -c`.

## Alias Storage Format

```json
{
  "aliases": {
    "deploy": {
      "executionType": "sequential",
      "commands": ["git push origin main && ssh server deploy"],
      "description": "Deploy to production"
    }
  }
}
```

- `executionType`: `"sequential"` (default) or `"parallel"` (`-p` flag)
- `commands`: Array of command strings. Sequential joins with space; parallel runs concurrently.
- `description`: Optional, added via `-d` flag.

## Parameter Substitution

Aliases support `{1}`, `{2}`, ... `{n}` placeholders that are replaced at runtime:

```bash
ez add tag 'git tag -a {1} -m "Release {1}"'
ez tag v2.0.0  # → git tag -a v2.0.0 -m "Release v2.0.0"
```

- Implemented in `Alias.substituting(arguments:)` and `Alias.maxPlaceholderIndex`
- Validated in `ez.swift` before execution — missing args produce an error message

**Extra argument overflow:** Any arguments beyond the highest placeholder index are appended to the end of the last command. This works for both parameterized and non-parameterized aliases:

```bash
# Non-parameterized — all args appended
ez add gs "git stash"
ez gs pop             # → git stash pop

# Parameterized — extra args appended after substitution
ez add greet 'echo hello {1}'
ez greet world foo bar  # → echo hello world foo bar
```

- Implemented in `Alias.appending(extraArguments:)` — shell-escapes each extra arg (preserving spaces via single-quoting) and appends to the last command string
- `ez.swift` splits args: first N go to placeholder substitution, the rest go to `appending(extraArguments:)`

## Scope System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [urtti/ez](https://github.com/urtti/ez) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
