---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Pwndbg is a GDB and LLDB plugin that enhances debugging for low-level software developers, reverse engineers, and exploit developers. It's written in Python and supports both GDB (mature, battle-tested) and LLDB (experimental, early-stage).

**Important**: Read the developer documentation in `docs/contributing/` for comprehensive guidance. This file highlights critical information for AI-assisted development.

## Quick Reference

### Essential Commands

```bash
# Setup
./setup.sh              # Install pwndbg
./setup-dev.sh          # Setup dev environment with pre-push hooks

# Testing (CRITICAL: use dbg tests for dual-debugger support!)
./tests.sh -d gdb -g dbg              # Run dual-debugger tests with GDB
./tests.sh -d lldb -g dbg             # Run dual-debugger tests with LLDB
./tests.sh -d gdb -g gdb              # GDB-specific tests 
./tests.sh -d gdb -g cross-arch-user  # Cross-architecture tests
./kernel-tests.sh                     # Kernel tests
./unit-tests.sh                       # Unit tests (no debugger)

# Linting
./lint.sh              # Run all checks
./lint.sh -f           # Fix formatting + run checks
./lint.sh -fo          # Fix formatting only (faster)

# Documentation
./scripts/generate-docs.sh    # Update auto-generated docs (requires GDB + LLDB)
./scripts/verify-docs.sh      # Check docs match source code
./scripts/docs-live.sh        # Preview docs at http://127.0.0.1:8000/pwndbg/

# Run custom Python code wit Pwndbg+GDB
./.venv/bin/gdb /bin/ls --ex 'entry' --ex 'source code.py'  # if Pwndbg is sourced in ~/.gdbinit
./.venv/bin/pwndbg /bin/ls --ex 'entry' --ex 'source code.py'  # if Pwndbg is not sourced
```

### Running Single Tests

```bash
# Use ./tests.sh with a filter (matches test names by regex)
./tests.sh -d gdb -g dbg test_config           # Run tests matching "test_config"
./tests.sh -d gdb -g dbg heap                  # Run all heap-related tests
./tests.sh -d gdb -g gdb test_symbol           # Run GDB-specific tests matching "test_symbol"

# Other useful options: -v (verbose), -s (serial), -p (pdb on failure)
# See ./tests.sh --help for all options

# For debugging cross-arch: see docs/contributing/testing-crossarch.md
```

## Architecture Overview

**Read `docs/contributing/dev-notes.md` and `docs/contributing/common-pitfalls.md` for detailed architecture information.**

### Module Hierarchy (with strict import rules)

```
pwndbg/commands/     -> User-facing commands (NOT an API, don't import!)
    ↓ (can import from)
pwndbg/aglib/        -> Debugger-agnostic library (complex operations)
    ↓ (can import from)
pwndbg/dbg_mod/      -> Debugger abstraction layer (provides pwndbg.dbg)
    |                   ├── gdb/  (GDB implementations)
    |                   └── lldb/ (LLDB implementations)
    ↓ (can import from)
pwndbg/lib/          -> Generic utilities, NO debugger dependencies
                        (lib can ONLY import from lib)
```

### Critical Import Rules

From `docs/contributing/common-pitfalls.md`:

1. **`pwndbg/lib/` only imports `pwndbg/lib/`** - No debugger dependencies allowed
2. **`pwndbg/dbg_mod/` never imports `aglib`** - Dependency goes the other way
3. **`pwndbg/dbg_mod/__init__.py` never imports debugger-specific code** - Breaks abstraction
4. **Never import commands** - Refactor shared logic into `aglib/`
5. **Import at top of file** - Function-level imports suggest refactoring needed
6. **Don't do `from pwndbg.aglib import arch/regs`** - These are runtime-swapped objects
   - Always use: `pwndbg.aglib.arch.whatever()` and `pwndbg.aglib.regs.whatever()`

### Key Architectural Concepts

- **Debugger API**: Terse set of debugging primitives in `pwndbg/dbg_mod/`
  - Get Process object: `pwndbg.dbg.selected_inferior()`
  - Think of it as debugger-agnostic version of `gdb`/`lldb` Python modules
- **aglib**: Builds complex functionality on debugger primitives (vmmap, heap, disasm, etc.)
- **gdblib**: Legacy GDB code being phased out - avoid touching, use `aglib` instead
- **Events**: Custom system with `@pwndbg.dbg.event_handler` decorator
- **Caching**: Memoization via `@cache_until` decorators in `pwndbg/lib/cache.py`
  - Be careful when adding caching since it may introduce subtle bugs (when cache is not cleared as often as it should be)

## Adding Commands

**Read `docs/contributing/adding-a-command.md` for full details.**

Create `pwndbg/commands/my_command.py`, then import in `pwndbg/commands/__init__.py`:

```python
import argparse
import pwndbg.commands

parser = argparse.ArgumentParser(description="Brief description.")
parser.add_argument("arg", type=int, help="Argument help")

@pwndbg.commands.Command(
    parser,
    category=pwndbg.commands.CommandCategory.MISC,
    aliases=["alias"],
    examples="usage examples here",
    notes="additional notes",
    # For debugger-specific commands:
    # only_debuggers={pwndbg.dbg_mod.DebuggerType.GDB},
)
@pwndbg.commands.OnlyWhenRunning  # Use appropriate decorators
def my_command(arg: int) -> None:
    """Command implementation"""
    print(f"Got: {arg}")
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pwndbg/pwndbg](https://github.com/pwndbg/pwndbg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
