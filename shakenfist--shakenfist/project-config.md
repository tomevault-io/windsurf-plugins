---
trigger: always_on
description: Shaken Fist is an opinionated, minimal cloud orchestration platform for VM and
---

# CLAUDE.md - Shaken Fist Development Guide

## Project Overview

Shaken Fist is an opinionated, minimal cloud orchestration platform for VM and
network management. It prioritizes simplicity and maintainability over feature
completeness, designed to be understood in its entirety by a single developer.

- **Homepage**: https://shakenfist.com/
- **Repository**: https://github.com/shakenfist/shakenfist

## Development goals

There is some discussion of overall project goals in `GOALS.md`, and roadmap
items are documented in `docs/plans/*.md`. However, we should always:

* Ensure objects are correctly cleaned up in `hard_delete()` when appropriate.
* Seek to improve mypy coverage, especially for code we've added.
* Ensure there is unit test coverage, and preferably functional test coverage
  as well (see `shakenfist/deploy/shakenfist_ci` for functional coverage). We
  prefer functional tests to unit tests if we can only have one.
* Preserve event logging -- the exact logging can change if required, but we
  like having good coverage of events for audit and debugging.
* Object or attribute filtering should be "pushed down" to the mariadb SQL
  layer where indexes can improve its performance where possible. Also ensure
  that we have the appropriate indicies for query efficiency when adding
  queries.
* Perform a code review a the end of a plan and address any concerns raised.

## Quick Reference

### Running Tests

```bash
tox                              # Run all tests
stestr run {test_name}           # Run specific test
tox -eflake8 -- -HEAD            # Style check on changed files only
tox -ecover                      # Generate coverage report
```

### Pre-commit Hooks

```bash
pip install pre-commit           # Install pre-commit
pre-commit install               # Set up git hooks
pre-commit run --all-files       # Run all hooks manually
```

The repository uses `actionlint` to validate GitHub Actions workflow files.
Install `actionlint`:
- macOS: `brew install actionlint`
- Linux: Download from https://github.com/rhysd/actionlint/releases

### Key Entry Points (CLI Tools)

- `sf-ctl` - Control CLI
- `sf-api` - REST API server
- `sf-database` - Database microservice daemon (runs on database-tier nodes)
- `sf-cleaner`, `sf-cluster`, `sf-net`, `sf-queues`, `sf-resources` - Daemons

### Generating gRPC Stubs from Proto Files

Proto files are in `protos/` and generated Python stubs go to `shakenfist/protos/`.
To regenerate after modifying `.proto` files or Python enum definitions:

```bash
tox -e genprotos
```

This tox environment:
1. Uses the exact library versions from `pyproject.toml` (avoiding version mismatch
   errors at runtime)
2. Generates protobuf enum definitions from Python source files (using AST parsing
   of `schema/object_types.py` and `schema/ipam_reservation.py`)
3. Compiles all `.proto` files to Python code and type stubs
4. Fixes import statements in generated code (e.g., `import foo_pb2` becomes
   `from shakenfist.protos import foo_pb2`)

**IMPORTANT:** Always use `tox -e genprotos` - never run `grpc_tools.protoc`
directly. The tox environment ensures correct versions and the underlying script
performs essential post-processing.

**Note:** The underlying script works on both macOS and Linux.

## Code Style and Conventions

### Python Style

- Use **single quotes** for strings, **double quotes** for docstrings
- Wrap lines at **120 characters**
- Trim trailing whitespace
- Follow existing import ordering (stdlib, third-party, shakenfist)

### Import Organization

**All imports must be at the top of the file.** Late imports inside functions
or methods should only be used when absolutely necessary to avoid circular
import issues, and must include a comment explaining why. If you see a late
import without such a comment, move it to the top of the file.

```python
# 1. Standard library
import base64
import json
import os

# 2. Third-party
import jinja2
from shakenfist_utilities import logs  # noreorder

# 3. Shaken Fist internal
from shakenfist import artifact
from shakenfist import baseobject
from shakenfist.config import config
```

### Abbreviated Imports (Common Pattern)

```python
from shakenfist.baseobject import DatabaseBackedObject as dbo
from shakenfist.baseobject import DatabaseBackedObjectWithOperations as dbowo
from shakenfist.baseobject import DatabaseBackedObjectIterator as dbo_iter
```

### Logging Pattern

Every module should set up logging like this:

```python
from shakenfist_utilities import logs

LOG, _ = logs.setup(__name__)

# Usage with structured fields:
log = LOG.with_fields({'instance_uuid': uuid})
log.info('Instance created')
```

### Copyright Header

All files should start with:

```python
# Copyright 2019 Michael Still and contributors
```

## Architecture Overview

### Directory Structure

```
shakenfist/
├── shakenfist/               # Core package
│   ├── instance.py          # VM instance management (~2000 lines)
│   ├── network/             # Virtual networking (VXLAN)
│   │   ├── network.py       # Network implementation
│   │   └── interface.py     # Network interfaces
│   ├── artifact.py          # Disk images and artifacts
│   ├── blob.py              # Content-addressable blob storage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shakenfist/shakenfist](https://github.com/shakenfist/shakenfist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
