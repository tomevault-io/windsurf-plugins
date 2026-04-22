---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

VTest2 is a standalone HTTP testing program, originally derived from Varnish's varnishtest. It's designed to test HTTP clients, servers, and proxies using a domain-specific scripting language (`.vtc` files). This is the second iteration that maintains better synchronization with Varnish-Cache while being usable independently.

## Build Commands

### Basic Build
```bash
make vtest          # Build without Varnish support
make test           # Build and run all tests
make clean          # Remove build artifacts
```

### Build with Varnish Support
```bash
make varnishtest VARNISH_SRC=/path/to/varnish-cache
```

### Generated Files
The build process generates two files that must exist before compilation:
- `src/teken_state.h` - Generated from `src/sequences` using awk
- `src/vtc_h2_dectbl.h` - Generated from `src/tbl/vhp_huffman.h` using Python

### Syncing with Varnish-Cache
```bash
make update         # Pull updates from Varnish-Cache repository
                    # Set VARNISHSRC env var to use local repo instead of cloning
```

## Code Architecture

### Directory Structure
- **lib/** - Shared utility library code from Varnish-Cache (vsb, vqueue, vtcp, etc.)
- **src/** - Core VTest implementation
  - `vtc_main.c` - Entry point, test execution orchestration, parallel test runner
  - `vtc.c` - Core test execution, macro expansion, command dispatch
  - `vtc_http.c/vtc_http.h` - HTTP/1.1 client/server implementation with session operations abstraction
  - `vtc_http2.c` - HTTP/2 support
  - `vtc_tls.c` - TLS/SSL support (OpenSSL-based)
  - `vtc_client.c` - Client command implementation
  - `vtc_server.c` - Server command implementation
  - `vtc_process.c` - Process spawning and management with terminal emulation
  - `vtc_barrier.c` - Synchronization primitives for tests
  - `vtc_logexp.c` - Log expectation matching (conditional, Varnish-specific)
  - `vtc_varnish.c` - Varnish-specific testing (conditional compilation)
  - `vtc_haproxy.c` - HAProxy-specific testing
  - `builtin_cert.h` - Default self-signed certificate (CN=example.com)
  - `vtc_asn_gentm.c` - OpenSSL compatibility layer for older versions
  - `tbl/` - Table-driven code generation (H2 frames, settings, errors, TLS alerts/protocols)
- **tests/** - Test suite (`.vtc` files)
- **tools/sync/** - Scripts for syncing code with Varnish-Cache

### Command System
Commands are registered via `cmds.h` using X-macros:
- **CMD_GLOBAL**: Available everywhere (barrier, delay, shell, include)
- **CMD_TOP**: Top-level test commands (client, server, process, varnish, haproxy, etc.)

The extensible command system allows loading shared libraries via `-E extension_shlib` to add custom commands at runtime using `add_cmd()`.

### Core Concepts

**Test Execution Model:**
Tests are executed in forked child processes with their own temporary directories. The main process (`vtc_main.c`) orchestrates parallel execution (configurable via `-j`), manages timeouts, and collects results via pipes.

**Macro System:**
VTest has a powerful macro expansion system supporting:
- Static macros defined via `-D name=val`
- Dynamic macros defined in test scripts
- Function macros (e.g., `${date}`, `${string,repeat,N,text}`)
- Automatic macros for servers/clients (sNAME_addr, sNAME_port, sNAME_sock)

**Session Abstraction:**
HTTP clients and servers share common session handling code (`vtc_sess.c`, `sess_process()`) with pluggable connect/disconnect callbacks. This enables code reuse between client, server, and tunneling modes.

**Session Operations Pattern:**
VTest2 uses a session operations abstraction (`struct sess_ops` in `vtc_http.h`) that enables transparent switching between plain TCP and TLS connections. The `struct http` contains operation pointers (`poll`, `read`, `write`, `close`) that dispatch to either plain file descriptor operations (`http_fd_so`) or TLS operations (`tlsconn_so`). This allows all HTTP I/O code to remain unchanged whether using plain or encrypted connections.

**Terminal Emulation:**
Process commands include a full terminal emulator (`teken.c`) supporting VT220 escape sequences for testing interactive programs and screen output.

## Test File Format

Tests are `.vtc` files that must start with `vtest` or `varnishtest` followed by a description:
```vtc
vtest "Test description"

server s1 {
    rxreq
    txresp
} -start

client c1 -connect ${s1_sock} {
    txreq
    rxresp
    expect resp.status == 200
} -run
```

### TLS Test Example
```vtc
vtest "Basic TLS test"

server s1 {
    tls_config {
        cert = "/path/to/cert.pem"
        version = TLSv1.2 TLSv1.3
    }
    tls_handshake
    rxreq
    txresp -status 200
} -start

client c1 {
    tls_config {
        servername = "example.com"
        verify_peer = false
    }
    tls_handshake
    txreq -url "/"
    rxresp
    expect resp.status == 200
    expect tls.version == "TLSv1.3"
} -run
```

## Build Dependencies

### Linux
- libpcre2-dev
- zlib
- libssl-dev (OpenSSL 1.0.x or later, 1.1.0+ recommended)

### macOS
- Libraries typically available via Homebrew
- OpenSSL 3.x (via `brew install openssl@3`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/varnish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
