---
trigger: always_on
description: Preferred: **Chinese (Simplified)**. Fallback: English. Never use other languages.
---

# mma-mcp — Wolfram Engine MCP Server

## Working Language

Preferred: **Chinese (Simplified)**. Fallback: English. Never use other languages.

## Project Overview

A Model Context Protocol (MCP) server that wraps a local Wolfram Engine, enabling AI assistants (Claude, ChatGPT, etc.) to invoke Wolfram Language computation — symbolic math, numerical analysis, data visualization, and more.

## Tech Stack

- **Language:** Python 3.11+
- **MCP framework:** `mcp[cli]` (official Python SDK, FastMCP)
- **Wolfram bridge:** `wolframclient` (local kernel via `WolframLanguageSession`)
- **HTTP:** Starlette + uvicorn (via `mcp[cli]` transitive deps)
- **Package manager:** `uv`

## Project Structure

```
mma-mcp/
├── src/
│   └── mma_mcp/
│       ├── __init__.py
│       ├── server.py              # App class + CLI entry point (argparse subcommands)
│       ├── config.py              # TOML config loading, dataclasses, validation
│       ├── kernel.py              # KernelSession: single kernel lifecycle, auto-restart, timeout
│       ├── pool.py               # KernelPool: worker pool for cross-client isolation
│       ├── auth.py                # BearerAuthMiddleware, ClientIdentity contextvar
│       ├── oauth.py               # Minimal OAuth 2.1 server (DCR + PKCE + AuthCode)
│       ├── passwords.py           # scrypt hash/verify (stdlib only)
│       ├── logging_config.py      # Structured logging with per-request ID
│       ├── stdio_transport.py     # Custom stdio transport (fixes SDK pipe hang)
│       ├── caddyfile.py           # Caddyfile generator for HTTPS deployment
│       ├── setup_groups.py        # Generate security group JSONs from local kernel
│       ├── security/
│       │   ├── __init__.py
│       │   ├── filter.py          # ExpressionFilter: regex symbol extraction + policy check
│       │   ├── registry.py        # CapabilityRegistry: load groups, build filters
│       │   └── groups/            # User-generated JSON symbol lists per group (not committed)
│       │       ├── manifest.json  # Group metadata (29 groups: 22 safe + 7 dangerous)
│       │       ├── math_core.json, algebra.json, ...  # 22 safe groups
│       │       ├── system_exec.json, file_read.json, ...  # 7 dangerous groups
│       │       └── (regenerate via: mma-mcp setup)
│       └── tools/
│           ├── __init__.py        # Tool registry, ToolContext, RoleRuntime, RBAC wrapper
│           └── evaluate.py        # evaluate (text) / evaluate_image (PNG)
├── tests/
│   ├── test_security.py           # Filter + registry unit tests
│   ├── test_config.py             # Config loading/validation tests
│   ├── test_auth.py               # Auth + OAuth + password tests
│   ├── test_tools.py              # Tool registry + RBAC + session isolation tests
│   ├── test_cli.py                # CLI subcommand unit tests
│   ├── test_integration.py        # Real kernel integration tests
│   └── test_mcp_e2e.py            # Full MCP protocol end-to-end tests
├── scripts/
│   └── generate_groups.wl         # Pure WL alternative for group generation
├── pyproject.toml
├── CLAUDE.md
├── LICENSE                        # MIT License
├── README.md                      # English README
├── README-cn.md                   # Chinese README
├── ARCHITECTURE.md                # Architecture documentation (English)
├── ARCHITECTURE-cn.md             # Architecture documentation (Chinese)
├── DEPLOY.md                      # HTTPS deployment guide (English)
├── DEPLOY-cn.md                   # HTTPS deployment guide (Chinese)
├── CONTRIBUTING.md                # Contributing guide (English)
├── CONTRIBUTING-cn.md             # Contributing guide (Chinese)
├── SECURITY.md                    # Security policy (English)
└── SECURITY-cn.md                 # Security policy (Chinese)
```

## Architecture Overview

### Layered security model
```
Layer 1: Authentication (auth.py / oauth.py)
  └─ Bearer token / OAuth 2.1 → client identity

Layer 2: Role-based access control (tools/__init__.py)
  └─ Per-role tool permissions → which MCP tools can be called

Layer 3: Expression filtering (security/)
  └─ Per-role symbol policy → which WL functions can be used
```

### Key design decisions

- **Pre-kernel filtering:** Expressions are filtered in Python (regex symbol extraction) before the kernel sees them. The kernel only receives policy-compliant code.
- **Worker pool isolation:** Each tool call acquires an exclusive kernel worker from a pool (`KernelPool`). A temporary WL context is used per call and cleaned up on release. This provides process-level isolation between concurrent clients. Pool supports lazy creation, idle reclaim, and periodic worker restart.
- **Two-layer timeout:** WL-side `TimeConstrained` (cooperative) + Python-side `ThreadPoolExecutor` hard timeout (force-restart on stuck kernel).
- **Config-driven:** All behavior controlled via `mma_mcp.toml`. Tools, security policy, auth, resource limits — all configurable without code changes.
- **Contextvar-based RBAC:** `current_client` and `_active_filter` contextvars propagate per-request identity and security policy, concurrent-safe.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siqiliu-tsinghua/mma-mcp](https://github.com/siqiliu-tsinghua/mma-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
