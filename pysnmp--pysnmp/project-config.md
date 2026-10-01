---
trigger: always_on
description: When generating code for this repository:
---

# GitHub Copilot Instructions

## Priority Guidelines

When generating code for this repository:

1. **Version Compatibility**: Always detect and respect the exact versions of languages, frameworks, and libraries used in this project
2. **Context Files**: Prioritize patterns and standards defined in the `.github/copilot` directory
3. **Codebase Patterns**: When context files don't provide specific guidance, scan the codebase for established patterns
4. **Architectural Consistency**: Maintain the layered, RFC-mirrored architectural style and established module boundaries
5. **Code Quality**: Prioritize maintainability, performance, security, and testability in all generated code

## Technology Version Detection

Before generating code, scan the codebase to identify:

1. **Language Versions**: Detect the exact versions of programming languages in use
   - Examine `pyproject.toml` for the Python version constraint (`requires-python = ">=3.10"`)
   - Python Support from 3.10 to 3.14 is allowed, but do not use features introduced in versions later than the detected version or earlier than 3.10
   - The codebase uses f-strings (e.g. `pysnmp/error.py`, `pysnmp/entity/engine.py`) — f-strings are permitted
   - The `match` statement (3.10+) is permitted; do not introduce `type` aliases (3.12+) or other post-3.10 syntax

2. **Framework Versions**: Identify the exact versions of all frameworks
   - Build system: uv with hatchling (`hatchling>=1.0.0`, declared in `pyproject.toml` `[build-system]`)
   - Package metadata lives in `pyproject.toml` under the standard PEP 621 `[project]` table; the distribution name is `pysnmplib`. Read the current version from `version` in that table rather than assuming one
   - The in-package version is duplicated in `pysnmp/__init__.py` as `__version__` — keep both in sync when bumping versions
   - Never suggest features not available in the detected framework versions

3. **Library Versions**: Note the exact versions of key libraries and dependencies
   - Runtime dependencies (from `pyproject.toml` `[project].dependencies`): `pycryptodomex >=3.11.0,<4.0.0`, `pysnmp-pyasn1 >=2.0.2,<3.0.0`. pysmi is **not** one of them — the engine path never imports it (`tests/test_no_pysmi.py` holds that line)
   - Optional dependencies (from `pyproject.toml` `[project.optional-dependencies]`): the `compile` extra, `pysnmp-pysmi >=4.0.0,<5.0.0`, which is what `pysnmp.smi.compiler` needs and nothing else does
   - Dev dependencies (from `pyproject.toml` `[dependency-groups]`, PEP 735): `pysnmp-pysmi >=4.0.0,<5.0.0`, `sphinx >=7.0.0,<9.0.0`, `myst-parser >=4.0.0,<5.0.0`, `pytest >=9.0.3,<10.0.0`, `coverage[toml] >=7.2.0,<8.0.0`, `mypy >=1.15.0,<3.0.0`, `ruff >=0.4.0,<1.0.0`
   - Every one of those floors names a final release, and `[tool.uv] prerelease = "disallow"` keeps it that way: never propose a requirement that names a prerelease (`4.0.0rc4`, `2.0.1rc1`), and never relax that setting to make one resolve
   - Generate code compatible with these specific versions
   - The project depends on the **pysnmp-pyasn1** fork, not upstream pyasn1. Import from `pyasn1.type`, `pyasn1.codec.ber`, `pyasn1.error`, and `pyasn1.compat.octets` as seen throughout `pysnmp/proto/` and `pysnmp/smi/`
   - Cryptography uses **pycryptodomex** (the `Cryptodome` namespace), imported under `pysnmp/proto/secmod/` for USM auth/priv protocols
   - Never use APIs or features not available in the detected versions

## Context Files

Prioritize the following files in `.github/copilot` directory (if they exist):

- **architecture.md**: System architecture guidelines
- **tech-stack.md**: Technology versions and framework details
- **coding-standards.md**: Code style and formatting standards
- **folder-structure.md**: Project organization guidelines
- **exemplars.md**: Exemplary code patterns to follow

## Codebase Scanning Instructions

When context files don't provide specific guidance:

1. Identify similar files to the one being modified or created
2. Analyze patterns for:
   - Naming conventions
   - Code organization
   - Error handling
   - Logging approaches
   - Documentation style
   - Testing patterns

3. Follow the most consistent patterns found in the codebase
4. When conflicting patterns exist, prioritize patterns in newer files or files with higher test coverage
5. Never introduce patterns not found in the existing codebase

## Architecture

This is a pure-Python SNMP v1/v2c/v3 engine. The package layout mirrors the SNMP RFC architecture and must be respected when adding or modifying code:

- `pysnmp/proto/` — Protocol layer: ASN.1/BER type definitions (`rfc1155.py`, `rfc1902.py`, `rfc1905.py`), message processing models (`mpmod/`), security models (`secmod/`), access control (`acmod/`), PDU API (`api/v1.py`, `api/v2c.py`), and proxy conversion (`proxy/rfc2576.py`). Files are named after the RFC they implement.
- `pysnmp/smi/` — SMI layer: MIB builder/compiler (`builder.py`, `compiler.py`), instrumentation (`instrum.py`), MIB view (`view.py`), and the high-level SMI types in `smi/rfc1902.py` (`ObjectIdentity`, `ObjectType`, `NotificationType`). Bundled MIB modules live in `smi/mibs/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pysnmp/pysnmp](https://github.com/pysnmp/pysnmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
