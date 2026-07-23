---
trigger: always_on
description: SDB is an extensible post-mortem and live debugger for Linux kernels and
---

# SDB - The Slick/Simple Debugger

SDB is an extensible post-mortem and live debugger for Linux kernels and
userland programs, built on top of [drgn](https://github.com/osandov/drgn/).
It provides a pipeline-based command architecture (similar to Unix pipes)
where typed `drgn.Object` instances flow between commands.

## Quick Reference

```bash
# Live kernel debugging
sudo sdb

# Debug a crash dump
sdb <vmlinux> <crash_dump>

# Debug a core dump
sdb <binary> <core_dump>

# Evaluate a single command and exit
sdb <vmlinux> <dump> -e "spa | member spa_name"

# Run multiple commands in sequence (stops on first error)
sdb <vmlinux> <dump> -e "spa | count" -e "threads | count"

# Read commands from stdin (one per line, # comments allowed)
echo -e "spa | count\nthreads | count" | sdb <vmlinux> <dump> -

# List all commands as JSON (for tooling/AI agents)
sdb --list-commands

# Machine-readable JSON output
sdb <vmlinux> <dump> -e "spa" --json
```

## Architecture

### Pipeline Model

Commands are composed using `|` pipes. Objects flow left-to-right through
the pipeline. Each command receives an iterable of `drgn.Object` and yields
an iterable of `drgn.Object`:

```
sdb> find_task 1 | member comm
(char [16])"systemd"
```

Shell pipes are supported with `!`:
```
sdb> addr modules | lxlist "struct module" list | member name ! sort | head -n 3
```

### Command Type Hierarchy

All commands inherit from `sdb.Command` (defined in `sdb/command.py`):

- **Command** - Base class. Implements `_call(objs) -> Iterable[drgn.Object]`.
- **SingleInputCommand** - Processes each input object independently; errors
  on one object don't stop processing of subsequent objects.
- **Walker** - Iterates over container data structures (linked lists, trees,
  etc.). Implements `walk(obj) -> Iterable[drgn.Object]`. Registered by
  `input_type` in `Walker.allWalkers`.
- **PrettyPrinter** - Formats objects for human-readable display. Implements
  `pretty_print(objs) -> None`. Registered by `input_type` in
  `PrettyPrinter.all_printers`.
- **Locator** - Finds objects of a given type. Can start a pipeline (via
  `no_input()`) or accept input (via `@InputHandler` decorated methods).
  Often also a PrettyPrinter (hybrid pattern).

### Key Attributes on Command Subclasses

```python
class MyCommand(sdb.Command):
    names = ["mycmd", "mc"]           # CLI names/aliases
    load_on = [sdb.Kernel()]          # When to register: All(), Kernel(),
                                      #   Userland(), Module("zfs"), Library("libc")
    input_type = "struct foo *"       # Expected input C type (optional)
```

### Writing a New Command

Create a `.py` file in `sdb/commands/` (or a subdirectory). The command is
auto-discovered via `__init_subclass__`:

```python
import sdb
from typing import Iterable
import drgn

class MyLocator(sdb.Locator, sdb.PrettyPrinter):
    """
    One-line summary of what this command does

    DESCRIPTION
        Detailed description here.

    EXAMPLES
        sdb> mycmd
        ...
    """
    names = ["mycmd"]
    input_type = "struct foo *"
    output_type = "struct foo *"
    load_on = [sdb.Kernel()]

    def no_input(self) -> Iterable[drgn.Object]:
        # Find all foo objects when used at pipeline start
        ...

    def pretty_print(self, objs: Iterable[drgn.Object]) -> None:
        for obj in objs:
            print(f"foo at {hex(obj.value_())}")
```

External commands can be loaded from arbitrary paths via:
- `--load-commands PATH` CLI flag
- `SDB_COMMANDS_PATH` environment variable (colon-separated)
- `%load-commands PATH` in the REPL

## Project Layout

```
sdb/
├── __init__.py          # Public API exports
├── command.py           # Command base classes (Command, Walker, PrettyPrinter, Locator)
├── pipeline.py          # Pipeline execution engine
├── parser.py            # Pipeline tokenizer/parser
├── target.py            # drgn.Program wrapper and type utilities
├── loader.py            # External command loading
├── error.py             # Exception hierarchy
├── session.py           # Session recording/replay
├── mdb_compat.py        # mdb syntax preprocessing
├── internal/
│   ├── cli.py           # CLI entry point (main())
│   └── repl.py          # Interactive REPL
└── commands/            # Built-in commands
    ├── *.py             # Core commands (echo, cast, filter, member, etc.)
    ├── linux/           # Linux kernel commands
    ├── zfs/             # ZFS-specific commands
    └── spl/             # SPL commands
```

## Development

```bash
# Install in dev mode
pip install -e ".[dev]"

# Run unit tests (no crash dumps needed)
pytest -v tests/unit

# Lint
pylint -d duplicate-code -d invalid-name sdb
ruff check sdb tests

# Type check
mypy --strict --show-error-codes -p sdb

# Format check
yapf --diff --style google --recursive sdb
```

## Running Integration/Regression Tests Locally

Integration tests require crash dumps and additional native libraries not
needed by unit tests. The GitHub Actions workflow (`.github/workflows/main.yml`,
job `pytest-integration`) is the authoritative reference for how to set this
up. The helper scripts live in `.github/scripts/`:

```bash
# 1. Install native dependencies for drgn and libkdumpfile
./.github/scripts/install-drgn.sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delphix/sdb](https://github.com/delphix/sdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
