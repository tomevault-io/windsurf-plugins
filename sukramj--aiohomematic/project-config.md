---
trigger: always_on
description: Transforms paramset descriptions into typed data points; supports generic, custom, calculated
---

# CLAUDE.md - AI Assistant Guide for aiohomematic

This document is the entry point for AI assistants (like Claude) working on the aiohomematic codebase.
It intentionally stays **concise** and delegates deep details to `docs/` — treat the linked documents
as the source of truth and only fall back to this file for orientation.

## Table of Contents

1. [CUxD / CCU-Jack Special Handling](#-critical-cuxdccu-jack-special-handling) (critical)
2. [Project Overview](#project-overview)
3. [Codebase Structure](#codebase-structure)
4. [Development Environment](#development-environment)
5. [Code Quality & Standards](#code-quality--standards)
6. [Testing Guidelines](#testing-guidelines)
7. [Architecture & Design Patterns](#architecture--design-patterns)
8. [Common Development Tasks](#common-development-tasks)
9. [Git Workflow](#git-workflow)
10. [Key Conventions](#key-conventions)
11. [Implementation Policy](#implementation-policy)
12. [Interaction Protocol](#interaction-protocol)
13. [Tips for AI Assistants](#tips-for-ai-assistants)

---

## ⚠️ CRITICAL: CUxD/CCU-Jack Special Handling

> **STOP AND READ THIS BEFORE ANY REFACTORING**
>
> CUxD and CCU-Jack are **NOT** normal Homematic interfaces. They require special handling that has been broken multiple times during AI-assisted refactoring. Before making any changes, run:
>
> ```bash
> pytest tests/contract/test_cuxd_ccu_jack_contract.py -v
> ```

### What Makes CUxD/CCU-Jack Different

| Aspect            | Standard Interfaces (HmIP-RF, BidCos-RF) | CUxD / CCU-Jack                             |
| ----------------- | ---------------------------------------- | ------------------------------------------- |
| **Protocol**      | XML-RPC                                  | JSON-RPC                                    |
| **Ports**         | 2001, 2010, 2000, 2002                   | 80 (HTTP) / 443 (HTTPS)                     |
| **Events**        | XML-RPC Server (push)                    | Polling (default) or MQTT (optional via HA) |
| **Ping/Pong**     | ✅ Yes                                   | ❌ No                                       |
| **Backend Class** | `CcuBackend` or `HomegearBackend`        | `JsonCcuBackend`                            |
| **XML-RPC Proxy** | Required                                 | **NOT used**                                |
| **Capabilities**  | `CCU_CAPABILITIES`                       | `JSON_CCU_CAPABILITIES`                     |

### Key Constants to Check

```python
# In aiohomematic/const.py:

# CUxD/CCU-Jack MUST be in this set:
INTERFACES_REQUIRING_JSON_RPC_CLIENT = frozenset({Interface.CUXD, Interface.CCU_JACK, ...})

# CUxD/CCU-Jack MUST NOT be in these sets:
INTERFACES_REQUIRING_XML_RPC = frozenset({Interface.HMIP_RF, Interface.BIDCOS_RF, ...})
INTERFACES_SUPPORTING_RPC_CALLBACK = frozenset({...})  # Same as XML_RPC
```

### Critical Capability Flags

```python
# In aiohomematic/client/backends/capabilities.py:

JSON_CCU_CAPABILITIES = BackendCapabilities(
    ping_pong=False,      # ❌ NO ping/pong - events via MQTT
    rpc_callback=False,   # ❌ NO XML-RPC callback server
    push_updates=True/False,    # ✅, if Events arrive via MQTT, otherwise ❌
    # All other feature flags = False (no programs, sysvars, etc.)
)
```

### Common Regression Patterns

- **❌ Adding CUxD to XML-RPC interfaces** — breaks CUxD (tries to create XML-RPC proxies)
- **❌ Enabling `ping_pong=True` for JSON backends** — causes false disconnects after 180 s without events
- **❌ Requiring XML-RPC proxy in the factory** — fails for CUxD (no proxy exists)
- **❌ Using a dedicated port** — CUxD/CCU-Jack use JSON-RPC on 80/443, **not** a dedicated port
- **✅ Correct**: check `INTERFACES_REQUIRING_JSON_RPC_CLIENT` first; for those interfaces return a `JsonCcuBackend` and do not require a proxy.

### Refactoring Checklist for CUxD/CCU-Jack

Before committing any changes that touch:

- `aiohomematic/const.py` (interface constants)
- `aiohomematic/client/backends/` (backend factory or capabilities)
- `aiohomematic/client/interface_client.py` (connection checks)
- `aiohomematic/central/coordinators/connection_recovery.py` (recovery logic)

**Run these commands:**

```bash
# 1. CUxD/CCU-Jack contract tests
pytest tests/contract/test_cuxd_ccu_jack_contract.py -v

# 2. Capability contract tests
pytest tests/contract/test_capability_contract.py -v

# 3. Verify interface classification
python -c "from aiohomematic.const import *; print('JSON-RPC only:', INTERFACES_REQUIRING_JSON_RPC_CLIENT - INTERFACES_REQUIRING_XML_RPC)"
# Expected output: JSON-RPC only: frozenset({<Interface.CUXD: 'CUxD'>, <Interface.CCU_JACK: 'CCU-Jack'>})
```

See also: `docs/architecture/protocol_selection_guide.md`.

---

## Project Overview

**aiohomematic** is a modern, async Python library for controlling and monitoring Homematic and HomematicIP
devices. It powers the Home Assistant integration "Homematic(IP) Local".

### Key Characteristics

- **Language**: Python 3.14+
- **Framework**: AsyncIO-based
- **Status**: Production/Stable
- **Type Safety**: Fully typed, mypy strict mode
- **License**: MIT
- **Version**: single source of truth is `aiohomematic/const.py:VERSION` (matches the top entry of `changelog.md`)

### Core Dependencies

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SukramJ/aiohomematic](https://github.com/SukramJ/aiohomematic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
