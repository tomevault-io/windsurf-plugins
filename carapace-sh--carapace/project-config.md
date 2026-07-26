---
trigger: always_on
description: Carapace is a Go library that provides command argument completion for [spf13/cobra](https://github.com/spf13/cobra)-based CLI applications. It generates shell completion scripts and handles runtime completion callbacks for 11 shells: bash, bash-ble, cmd-clink, elvish, fish, ion, nushell, oil, powershell, tcsh, xonsh, and zsh.
---

# AGENTS.md

## Project Overview

Carapace is a Go library that provides command argument completion for [spf13/cobra](https://github.com/spf13/cobra)-based CLI applications. It generates shell completion scripts and handles runtime completion callbacks for 11 shells: bash, bash-ble, cmd-clink, elvish, fish, ion, nushell, oil, powershell, tcsh, xonsh, and zsh.

This is the **core library** (`github.com/carapace-sh/carapace`). Companion projects:
- `carapace-bin` — pre-built completions for 500+ commands
- `carapace-bridge` — bridges to other completion frameworks
- `carapace-pflag` — forked spf13/pflag with non-POSIX modes

## Build & Test Commands

```sh
# Build
go build ./...

# Run all tests (unit + example-nonposix via go.work)
go test ./...

# Run with coverage (matches CI)
mkdir -p .cover && CARAPACE_COVERDIR="$(pwd)/.cover" go test -v -coverpkg ./... -coverprofile=unit.cov ./... ./example-nonposix/...

# Benchmarks
go test -bench ./...

# Generate
go generate ./...

# Format check (CI uses this; must produce no output)
gofmt -d -s .

# Lint
golangci-lint run
```

## Architecture & Control Flow

### Two-Phase Completion Model

1. **Callback phase** — `Action` objects hold a `CompletionCallback` function. When `Invoke(ctx)` is called, the callback recursively resolves until it produces an `Action` with `rawValues` (no callback). Actions are lazy — callbacks execute only at completion time.

2. **Formatting phase** — `InvokedAction` is passed to the shell-specific formatter (`internal/shell/<shell>/action.go`) which produces the output string the shell consumes.

### Completion Dispatch Flow

```
cobra command invoked with _carapace subcommand
  → complete()                    # shell-specific arg patching
  → traverse()                   # classify args (flag, positional, subcommand, dash)
  → storage.get(cmd)            # resolve Action from per-command entry
  → action.Invoke(context)       # resolve callback chain
  → invokedAction.value(shell)   # format for target shell
```

### Core Types

| Type | File | Purpose |
|------|------|---------|
| `Carapace` | `carapace.go` | Wrapper around `*cobra.Command`; entry via `Gen()` |
| `Action` | `action.go` | Lazy completion definition (callback + meta + rawValues) |
| `InvokedAction` | `invokedAction.go` | Resolved action; supports Filter/Merge/Prefix/Suffix |
| `Context` | `context.go` | Completion context: Value, Args, Parts, Env, Dir |
| `Batch` | `batch.go` | Parallel action invocation via goroutines |
| `entry` | `storage.go` | Per-command storage: flag actions, positional actions, hooks |
| `RawValue` | `internal/common/value.go` | Single candidate: Value, Display, Description, Style, Tag, Uid |
| `Meta` | `internal/common/meta.go` | Metadata: Messages, Nospace, Usage, Queries |
| `SuffixMatcher` | `internal/common/suffix.go` | Controls when shell should NOT add trailing space |
| `pflagfork.FlagSet` | `internal/pflagfork/` | pflag wrapper with POSIX/non-POSIX flag lookup |
| `pflagfork.Flag` | `internal/pflagfork/` | Flag wrapper with Mode, Nargs, OptargDelimiter, Consumes |

## Directory Structure

```
/                           Root package — public API
  carapace.go               Gen(), PreRun(), PreInvoke(), Snippet(), IsCallback(), Test()
  action.go                 Action type, modifiers (Cache, Chdir, Filter, MultiParts, etc.)
  invokedAction.go          InvokedAction type, Merge, Prefix, Suffix, Filter, ToMultiPartsA
  complete.go               complete() — main dispatch with shell patching
  traverse.go               traverse() — argument classification state machine
  storage.go                Global per-command completion storage (mutex-protected)
  command.go                _carapace hidden subcommand + spec/style subcommands
  compat.go                 Cobra bridge: registerValidArgsFunction, registerFlagCompletion
  defaultActions.go         ActionValues, ActionFiles, ActionDirectories, ActionCommands, etc.
  internalActions.go        ActionCallback, ActionExecCommand, ActionImport, ActionMessage
  diff.go                   Diff() action
  log.go                    LOG (conditional logging)
  experimental.go           x.ClearStorage, x.Complete

/internal/
  common/                   RawValue, Meta, SuffixMatcher, Messages, Queries
  shell/                    Per-shell formatters (bash, zsh, fish, nushell, elvish, etc.)
    <shell>/action.go       ActionRawValues() — format completion output
    <shell>/snippet.go      Snippet() — generate shell setup script
    bash/patch.go           Redirect handling
    nushell/patch.go        Open-quote handling
    cmd_clink/patch.go      cmd-clink patching
    zsh/zstyle.go           zstyle integration, named directories
  pflagfork/                Non-POSIX flag mode handling
  spec/                     YAML spec generation from cobra commands
  config/                   Runtime config loading
  env/                      Environment variable accessors (CARAPACE_* constants)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carapace-sh/carapace](https://github.com/carapace-sh/carapace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
