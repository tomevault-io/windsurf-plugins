---
trigger: always_on
description: This document provides architectural guidelines and development rules for the darnit project.
---

# Darnit Project Guidelines

This document provides architectural guidelines and development rules for the darnit project.

## Architecture Overview

Darnit is an AI-powered compliance auditing framework with a plugin architecture that separates the core framework from compliance implementations.

### Package Structure

```
packages/
├── darnit/                  # Core framework (MUST NOT import implementations)
│   └── src/darnit/
│       ├── core/            # Plugin system, discovery, logging
│       ├── sieve/           # 4-phase verification pipeline
│       ├── config/          # Configuration loading and merging
│       ├── tools/           # MCP tool implementations
│       └── server/          # MCP server setup
│
├── darnit-baseline/         # OpenSSF Baseline implementation
│   └── src/darnit_baseline/
│       ├── attestation/     # In-toto attestation support
│       ├── config/          # Project context configuration
│       ├── formatters/      # Output formatting (Markdown, JSON, SARIF)
│       ├── remediation/     # Remediation orchestration
│       ├── rules/           # SARIF rule definitions (from TOML)
│       └── threat_model/    # Threat model generation
│
└── darnit-testchecks/       # Test implementation (for testing)
```

## Separation Rules

### Rule 1: Framework MUST NOT Import Implementations

The `darnit` package must never directly import implementation packages.

```python
# ❌ WRONG - Creates hard dependency
import darnit_baseline
from darnit_baseline.controls import level1

# ✅ CORRECT - Use plugin discovery
from darnit.core.discovery import get_implementation
impl = get_implementation("openssf-baseline")
if impl:
    controls = impl.get_all_controls()
```

### Rule 2: Implementations MAY Import Framework

Implementation packages can freely import from the framework.

```python
# ✅ OK - Implementation importing framework
from darnit.core.plugin import ComplianceImplementation, ControlSpec
from darnit.sieve import register_control
```

### Rule 3: Use Protocol Methods for Cross-Package Communication

All framework-to-implementation communication must go through the `ComplianceImplementation` protocol.

```python
# Protocol methods available:
impl.name                        # str: Implementation identifier
impl.display_name                # str: Human-readable name
impl.version                     # str: Implementation version
impl.spec_version                # str: Spec version implemented
impl.get_all_controls()          # List[ControlSpec]: All controls
impl.get_controls_by_level(n)    # List[ControlSpec]: Controls at level n
impl.get_rules_catalog()         # Dict: SARIF rule definitions
impl.get_remediation_registry()  # Dict: Auto-fix mappings
impl.get_framework_config_path() # Path | None: TOML config location
impl.register_controls()         # None: Register TOML controls
```

## Plugin System

### Entry Points

Implementations register via Python entry points in `pyproject.toml`:

```toml
[project.entry-points."darnit.implementations"]
openssf-baseline = "darnit_baseline:register"
```

### Creating a New Implementation

1. Create a new package with the implementation class:

```python
# my_framework/implementation.py
from pathlib import Path
from darnit.core.plugin import ComplianceImplementation, ControlSpec

class MyFrameworkImplementation:
    @property
    def name(self) -> str:
        return "my-framework"

    @property
    def display_name(self) -> str:
        return "My Compliance Framework"

    @property
    def version(self) -> str:
        return "1.0.0"

    @property
    def spec_version(self) -> str:
        return "MySpec v1.0"

    def get_all_controls(self) -> list[ControlSpec]:
        # Return your control definitions
        ...

    def get_framework_config_path(self) -> Path | None:
        return Path(__file__).parent / "my-framework.toml"

    def register_controls(self) -> None:
        pass  # Controls are defined in TOML; no Python registration needed
```

2. Add the registration function:

```python
# my_framework/__init__.py
def register():
    from .implementation import MyFrameworkImplementation
    return MyFrameworkImplementation()
```

3. Register via entry point:

```toml
[project.entry-points."darnit.implementations"]
my-framework = "my_framework:register"
```

## Sieve Pattern

The verification pipeline follows a 4-phase pattern using built-in handlers:

```
file_must_exist → exec/regex → llm_eval → manual
       ↓              ↓           ↓         ↓
  File presence   Commands &   AI-based   Human
  checks          patterns     eval       review
```

Each control can define passes at each phase. The orchestrator stops at the first conclusive result.

## Conservative-by-Default Principles

This is a compliance auditing tool. Incorrect results are worse than incomplete results. Every design decision must follow these rules:

### Never Assume Compliance

- A control that has not been **explicitly verified as passing** is NOT compliant. Period.
- "Needs Verification" / WARN means "we don't know" — treat it the same as FAIL for compliance calculations.
- Never report a level as "Compliant" if any control at that level is unverified, errored, or pending.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kusari-oss/darnit](https://github.com/kusari-oss/darnit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
