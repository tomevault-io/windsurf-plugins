---
trigger: always_on
description: This file provides instructions for working with this Python CLI project.
---

# CLAUDE.md - Project Instructions for Claude Code

This file provides instructions for working with this Python CLI project.

## Project Overview

- **Project name**: sambaudit
- **Type**: Security / SMB auditing CLI tool
- **Core functionality**: Discovers SMB hosts via LDAP, enumerates shares, BFS-crawls directory trees with threads, indexes everything to SQLite
- **Target users**: Security engineers auditing internal file shares; pentesters (loud)

## Project Structure

```
sambaudit/
├── pyproject.toml       # Package configuration
├── README.md            # Project documentation
├── LICENSE              # MIT License
├── sambaudit/           # Main Python package
│   ├── __init__.py      # Package init
│   ├── run.py           # CLI entry point
│   ├── cli_args.py      # Argument parsing (ArgsParser)
│   ├── core.py          # Main orchestration (SambAudit)
│   ├── db.py            # SQLite operations (DatabaseManager)
│   ├── logger.py        # SQLite-backed logging (AuditLogger)
│   ├── auth.py          # SMB auth — NTLM/Kerberos (Authenticator)
│   ├── ldap_query.py    # LDAP host discovery (LDAPQuerier)
│   ├── smb.py           # Share/directory listing (SMBOperations)
│   ├── crawl.py         # BFS crawler (ShareCrawler)
│   └── throttle.py      # Resource monitor (Throttler)
```

## Key Files

- **pyproject.toml**: Package configuration — dependencies, entry point `sambaudit`
- **sambaudit/run.py**: CLI entry point — calls `parse_args()` then `SambAudit.run()`
- **sambaudit/core.py**: Main orchestration — discover → enumerate → crawl
- **sambaudit/db.py**: All SQLite CRUD — thread-local connections, WAL mode
- **sambaudit/crawl.py**: BFS with two thread pools (coord + dir) to avoid deadlock
- **sambaudit/logger.py**: Custom `SQLiteHandler` — all logs go to DB, optionally to console

## Development Commands

### Installation - IMPORTANT
**NEVER install packages with pip outside of a virtual environment.** Use pipx instead:

```bash
# Install in development mode using pipx
pipx install -e .

# Reinstall to pick up new dependencies
pipx install -e . --force
```

### Building
```bash
# Build package
pip build

# Build source distribution
python -m build --sdist
```

### Testing

**IMPORTANT: Run the full test suite before marking any task complete.**

pytest must run inside the pipx venv so it can import impacket and all other dependencies.
**Do NOT use `pytest`, `python -m pytest`, or `$(pipx environment ...)` — these all fail.**

The concrete pytest path is always:
```
~/.local/pipx/venvs/sambaudit/bin/pytest
```

If it is missing (no output from `ls ~/.local/pipx/venvs/sambaudit/bin/pytest`), inject it first — this only needs to be done once:
```bash
pipx inject sambaudit pytest
```

Then run tests:
```bash
# Run all tests
~/.local/pipx/venvs/sambaudit/bin/pytest tests/

# Run with coverage report
~/.local/pipx/venvs/sambaudit/bin/pytest tests/ --cov=sambaudit

# Run a single test file
~/.local/pipx/venvs/sambaudit/bin/pytest tests/test_db.py -v
```

Tests live in `tests/` — one file per module (`test_db.py`, `test_auth.py`, etc.).

**Rules:**
- Every new module must have a corresponding `tests/test_<module>.py`
- Every bug fix must include a regression test that fails without the fix
- When a method changes behaviour, update its test to match
- Tests mock external services (Impacket SMB, LDAP, psutil) but use real SQLite for `db.py` tests
- Run the full suite after every change — do not mark a task done until it passes

### Running
```bash
# Show help
sambaudit --help

# Single target, anonymous
sambaudit --target 192.168.1.50

# Full LDAP discovery with NTLM auth
sambaudit --dc dc01.corp.local -D CORP -u alice -p 'S3cr3t!'

# LDAP when DNS cannot resolve the DC — --dc for Kerberos SPN, --dc-ip for TCP
sambaudit --dc dc01.corp.local --dc-ip 192.168.1.10 -D corp.local

# Kerberos via ccache (--kerberos is set automatically when --ccache is given)
sambaudit --dc dc01.corp.local --dc-ip 192.168.1.10 --ccache /tmp/user.ccache

# Share discovery only (no crawl)
sambaudit --dc dc01.corp.local -D CORP -u alice -p 'S3cr3t!' -sd

# Hosts file with optional IP override (hostname=ip skips DNS, useful when DNS is wrong)
sambaudit --hosts-file hosts.txt

# Resume an interrupted run
sambaudit --resume

# Debug output to console
sambaudit --target 192.168.1.50 -d
```

## Common Tasks

### Adding a new command
1. Edit `sambaudit/cli_args.py` - add argument in the relevant `_add_*_args()` method
2. Edit `sambaudit/core.py` - add logic in the appropriate workflow method

### Adding dependencies
1. Edit `pyproject.toml` - add to `dependencies` list
2. Confirm with user before adding new packages
3. Reinstall with pipx to load new dependencies: `pipx install -e . --force`

### Adding static files
1. Add files to `sambaudit/static/`
2. Update `pyproject.toml` under `[tool.setuptools.package-data]`

## Important Notes

- Entry point is defined in `pyproject.toml` under `[project.scripts]`
- Logger supports colored output for console and structured format for files
- Debug mode (`-d`) enables verbose logging
- **Always use pipx for installation** - never pip directly
- **Logging output is ignored** - do not commit log files to git

## Authentication Architecture

### Kerberos / ccache

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [d-woosley/SambAudit](https://github.com/d-woosley/SambAudit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
