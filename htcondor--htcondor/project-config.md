---
trigger: always_on
description: This file provides guidance to GitHub Copilot when assisting with HTCondor development. It follows best practices for modern C++ projects and Copilot instruction files.
---

# HTCondor GitHub Copilot Instructions

This file provides guidance to GitHub Copilot when assisting with HTCondor development. It follows best practices for modern C++ projects and Copilot instruction files.

## Quick Reference Card

| **Aspect** | **HTCondor Convention** |
|------------|------------------------|
| **Language** | C++20 standard |
| **Build System** | CMake (out-of-tree builds only) |
| **Logging** | `dprintf(D_LEVEL, fmt, ...)` not `printf()` |
| **File I/O** | `safe_open_wrapper()` / `safe_fopen_wrapper()` not `open()` / `fopen()` |
| **Errors** | `EXCEPT("message")` for fatal errors |
| **Member vars** | Prefix with `m_` (e.g., `m_count`) |
| **Indentation** | Tabs (size 4), not spaces |
| **Warnings** | Aim for zero warnings; some CI/build configs may enable `-Werror` |
| **Namespaces** | ClassAd uses `classad::` namespace |
| **Config** | Use `param()` family, never hardcode paths |

## Project Overview

HTCondor is a **distributed high-throughput computing system** written in C++20. The codebase consists of multiple daemons communicating via ClassAd messages over network sockets to schedule and execute jobs across distributed resources.

**Key Characteristics:**
- **Event-driven architecture** using DaemonCore framework
- **Mature codebase** with 30+ years of history (respect legacy patterns)
- **Security-critical** (runs with elevated privileges, handles user data)
- **Cross-platform** (Linux, Windows, macOS)
- **Large scale** (manages 100,000+ compute nodes in production)

## Architecture: Core Daemons

HTCondor uses a daemon-based architecture. The five essential daemons are:

1. **condor_master** (`src/condor_master.V6/`) - Root daemon, starts/monitors all others
2. **condor_schedd** (`src/condor_schedd.V6/`) - Job scheduler, manages job queue
3. **condor_startd** (`src/condor_startd.V6/`) - Worker node manager, executes jobs
4. **condor_collector** (`src/condor_collector.V6/`) - Central information repository
5. **condor_negotiator** (`src/condor_negotiator.V6/`) - Matchmaker (jobs ↔ resources)

Supporting daemons:
- **condor_shadow** (`src/condor_shadow.V6.1/`) - Schedd-side job proxy
- **condor_starter** (`src/condor_starter.V6.1/`) - Worker-side job executor

*Note: `.V6` suffixes are historical, ignore them.*

## Core Frameworks & Patterns

### DaemonCore Framework

All daemons are built on **DaemonCore** (`src/condor_daemon_core.V6/condor_daemon_core.h`), an event-driven framework providing:

- **Command registration**: `daemonCore->Register_Command(CMD_*, handler, ...)`
- **Timer registration**: `daemonCore->Register_Timer(interval, handler, ...)`
- **Signal handling**: `daemonCore->Register_Signal(SIGTERM, handler)`
- **Process spawning**: `daemonCore->Create_Process(...)`
- **Socket/stream management**: Inter-daemon communication primitives

**Golden Rule**: In daemon code, ALWAYS use DaemonCore for timers/signals/sockets. Never use raw POSIX calls (`timer_create()`, `signal()`, etc.) as they bypass the event loop. Note that the Register_* functions have a std::function overload for their callback.  Prefer the std::function overload for all new code, and refactoring of old code where possible.

### ClassAd System

**ClassAds** (`src/classad/`) are HTCondor's universal data structure—attribute-value pairs with expression evaluation. Everything is a ClassAd: jobs, machines, policies.

**Key Points:**
- **Namespace**: Lives in `classad::` namespace (`classad::ClassAd`)
- **Don't forward-declare**: Never `class ClassAd;` — include `condor_classad.h` instead
- **Case-insensitive**: Attribute names are case-insensitive
- **Use macros**: Reference attributes via `ATTR_*` constants from `condor_attributes.h`

### Logging with dprintf

Use `dprintf()` (from `condor_debug.h`) **instead of `printf()`** for all output:

**Debug Categories:** For daemons, enable fine-grained logging via `ALL_DEBUG`, `DEFAULT_DEBUG`, and `<SUBSYS>_DEBUG` config parameters; for command-line tools, use `TOOL_DEBUG`. Common categories: `D_ALWAYS`, `D_ERROR`, `D_FULLDEBUG`, `D_NETWORK`, `D_SECURITY`, `D_DAEMONCORE`, `D_COMMAND`, `D_MATCH`.

**Best Practices:**
- Use `D_ERROR` for errors (not `D_ALWAYS`)
- Use `D_FULLDEBUG` for verbose tracing (not `D_ALWAYS`)
- Use category-specific levels (`D_NETWORK`, `D_SECURITY`) when appropriate
- Always include newlines (`\n`)

### Error Handling with EXCEPT

Use `EXCEPT()` macro for **fatal errors** (invariant violations, impossible conditions):

**When to use:**
- Assertion-like checks that should never fail
- Unrecoverable errors (corrupted data structures, etc.)
- Programming errors (wrong API usage)

**When NOT to use:**
- Expected failures (file not found, network timeout)
- User errors (bad input, permission denied)
- Use regular error handling (return codes, exceptions) for these

### Network I/O Patterns

Communication uses **Stream** objects from `condor_io.h`:

**Stream Types:**
- **ReliSock** - Reliable TCP connections
- **SafeSock** - UDP datagrams
-- Use ReliSock and SafeSock where possible instead of raw sockets.

**Critical**: ALWAYS check return values—network operations are async and can fail.

## Build System & Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [htcondor/htcondor](https://github.com/htcondor/htcondor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
