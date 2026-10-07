---
trigger: always_on
description: Welcome! This repository contains the Pinpoint PHP and Python agents, a shared C++ core, and a Go-based collector agent.
---

# Pinpoint C Agent AI Instructions

Welcome! This repository contains the Pinpoint PHP and Python agents, a shared C++ core, and a Go-based collector agent.

## Project Architecture

The project follows a tiered architecture:
1.  **Language Agents (PHP/Python)**: Small, high-performance extensions (C++) or SDKs that hook into the runtime to record spans.
2.  **Common C++ Core ([common/](common/))**: Shared logic for span management and serialization used by both agents.
3.  **Collector Agent ([collector-agent/](collector-agent/))**: A Go-based bridge that receives data from agents (via TCP/UDS) and forwards it to the Pinpoint Collector via gRPC.

**Data Flow**: `App` -> `PHP/Python Agent (C++)` -> `Collector Agent (Go)` -> `Pinpoint Collector (Java)`.

## Build & Test Commands

### PHP Extension
- **Build**: `phpize && ./configure && make`
- **Install**: `sudo make install`
- **Test**: `make test` (runs `.phpt` tests in [tests/](tests/))

### Python Agent
- **Build**: `pip install .`
- **Test**: `python -m unittest discover src/PY/test`

### Go Collector Agent
- **Build**: `cd collector-agent && make server`
- **Test**: `cd collector-agent && make test`

### Common C++ Core
- **Build**: `cd common && mkdir build && cd build && cmake .. && make`
- **Test**: `CTEST_OUTPUT_ON_FAILURE=1 make test` (requires `-DWITH_TEST_CASE=ON` in cmake)

## Important Conventions & Pitfalls

- **Shared Core**: Any changes to [common/](common/) impact **both** PHP and Python agents. Ensure both are tested/validated.
- **PHP AOP Cache**: When editing PHP AOP plugins, you may need to clear the cache: `rm /tmp/.cache/.__class_index.php`.
- **Go Collector Protoc**: Compiling protobufs for the Go collector requires `protoc` and `protoc-gen-go` to be installed and in the PATH. See [collector-agent/makefile](collector-agent/makefile) for the `protoc` command.
- **Build Stability**: Recent changes have removed "version passing" features to improve build stability. Avoid re-introducing complex versioning logic into the build scripts without extensive testing.
- **Documentation**: Extensive documentation is available in the [DOC/](DOC/) directory for each component. Refer to them before making architectural changes.
- **Environment Variables**: The Collector Agent is configured primarily via environment variables (see [collector-agent/env.list](collector-agent/env.list)).

## Branch Creation Rules

When creating a new branch, follow these conventions (observed from the repo's
existing branches):

- **Prefix by type** (kebab-case, lowercase):
  - `fix/...` — bug fixes (e.g. `fix/collector-agent-security-panics`)
  - `feat/...` — new features (e.g. `feat/issue-triage-bot`)
  - `ci/...` — CI/workflow changes (e.g. `ci/path-filtering`)
  - `chore/...` — maintenance, docs, build tooling
  - `refactor/...` — code restructuring without behavior change
- **Scope by product** when the change is product-specific: include the product
  name in the branch (e.g. `fix/collector-agent-...`, `fix/python-...`,
  `fix/php-...`, `fix/common-...`).
- **Never branch directly from a version branch** (`main`, `dev`, `0.7`, etc.)
  for feature work — always create a short-lived topic branch from `dev` (the
  default branch) unless told otherwise.
- **Keep names short and descriptive**, separated by `-`; no spaces, no
  uppercase, no underscores.
- **Do not reuse existing branch names**; if a branch already exists for the
  same topic, check with the user before creating a duplicate.
- **Version bumps are their own concern**: a version change is made by editing
  `versions.json` (the single source of truth), not by creating a
  version-named branch. See the "Version Management" section.

## Version Management

- **Single source of truth**: all product versions live in [versions.json](versions.json)
  at the repo root. Never hardcode a version in source files — read it from
  `versions.json` at build time.
- **Products and their keys**: `collector-agent`, `php`, `python`, `common`.
  Each product versions independently (they are NOT kept in sync).
- **How each product consumes the version**:
  - `common` — `common/CMakeLists.txt` reads `versions.json` and injects
    `PINPOINT_C_AGENT_API_VERSION` via `configure_file` from
    `common/include/common.h.in`.
  - `collector-agent` — `make version` generates `server/version_gen.go`
    (gitignored) from `versions.json`; keep the `v` prefix (the client compares
    it lexicographically against `lowest_version="v0.7.0"`).
  - `python` — `setup.py` / `setup_pypi_test.py` read `versions.json`;
    `plugins/PY/pinpointPy/__init__.py` resolves it at runtime with a fallback.
  - `php` — `config.m4` injects `PHP_PINPOINT_PHP_VERSION` via `AC_DEFINE`;
    `php_pinpoint_php.h` keeps a fallback default.
- **Release tags are product-prefixed** (strategy 1): `collector-agent/v*.*.*`,
  `php/v*.*.*`, `python/v*.*.*`, `common/v*.*.*`.
- **Publishing is fully manual** (`workflow_dispatch` only — no `release` or
  `push: tags` triggers). Each product has its own manual release workflow:
  - `collector-agent` → `publish-collector-agent.yml`
  - `php` → `release.yml`
  - `python` → `build-wheels.yml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pinpoint-apm/pinpoint-c-agent](https://github.com/pinpoint-apm/pinpoint-c-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
