---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the integrated Skewed Emacs and Gendl development environment.
---

# Skewed Emacs + Gendl Docker Development Environment

This file provides guidance to Claude Code (claude.ai/code) when working with the integrated Skewed Emacs and Gendl development environment.

## Overview

This setup provides a complete Lisp development environment with:
- **Skewed Emacs Container**: Custom Emacs configuration with MCP integration
- **Gendl Container**: 3D CAD/modeling system with Common Lisp REPL and MCP integration
- **Network Integration**: Containers communicate via Docker network for SLIME connections
- **MCP Services**: Both containers expose services via Model Context Protocol for external tool integration

## MCP Integration

The containers are now wrapped as MCP (Model Context Protocol) services, providing seamless integration with Claude Code and other MCP-enabled tools.

### Available MCP Services

**Emacs Lisp Evaluation Service:**
- **Service Name**: `mcp__skewed-emacs__skewed-emacs__lisp_eval`
- **Purpose**: Evaluate Emacs Lisp code remotely
- **Usage**: `mcp__skewed-emacs__skewed-emacs__lisp_eval(code="(+ 1 2 3)")`

**Gendl Common Lisp Services (included with skewed-emacs):**
- `mcp__gendl_ccl__gendl_ccl__lisp_eval` — Gendl on Clozure CL (port 9080)
- `mcp__gendl_sbcl__gendl_sbcl__lisp_eval` — Gendl on SBCL (port 9090)

**Commercial Genworks GDL Services (via supplemental overlay repos):**
- `mcp__genworks_gdl_smp__genworks_gdl_smp__lisp_eval` — GDL with NURBS (port 9098)
- `mcp__genworks_gdl_enterprise_smp__genworks_gdl_enterprise_smp__lisp_eval` — Enterprise variant
- These are not included in skewed-emacs. Licensed users receive a supplemental
  repo to clone as a sibling directory, then run its `./install` script to
  add Docker Compose and MCP config overlays into skewed-emacs.

**Ping Services:**
- `mcp__skewed_emacs__skewed_emacs__ping_lisp` - Check Emacs availability
- `mcp__gendl_ccl__gendl_ccl__ping_lisp` - Check Gendl CCL availability
- `mcp__gendl_sbcl__gendl_sbcl__ping_lisp` - Check Gendl SBCL availability

### MCP vs Raw HTTP

**Previous Approach (Deprecated):**
```bash
# Raw HTTP calls (no longer recommended)
curl -X POST http://localhost:7080/lisply/lisp-eval -d '{"code": "(+ 1 2 3)"}'  # only from inside container
```

**Current Approach (Recommended):**
```python
# Through MCP services (seamless with Claude Code)
mcp__skewed_emacs__skewed_emacs__lisp_eval(code="(+ 1 2 3)")
```


### Verification Commands

Check if workarounds are active:
```bash
# Test current environment via MCP
mcp__skewed_emacs__skewed_emacs__lisp_eval(code='(list (getenv "SHELL") shell-file-name (getenv "PATH"))')

# Test native compilation settings
mcp__skewed_emacs__skewed_emacs__lisp_eval(code='native-comp-jit-compilation')

# Test assembler accessibility
mcp__skewed_emacs__skewed_emacs__lisp_eval(code='(shell-command-to-string "which as")')
```

## Quick Start

### 1. Start the Environment

```bash
# Navigate to skewed-emacs directory
cd ~/projects/skewed-emacs

# Start the full container stack
./compose-dev up

# Verify services are running via MCP
mcp__skewed_emacs__skewed_emacs__ping_lisp()      # Should return "pong"
mcp__gendl_ccl__gendl_ccl__ping_lisp()            # Should return "pong"
```

### 2. Connect to Development Environment

```bash
# Connect to Emacs in the container
docker exec -it skewed-emacs emacsclient -t

# From within Emacs, connect to Gendl SLIME
# M-x slime-connect RET gendl-ccl RET 4200 RET
```

## Container Details

### Skewed Emacs Container (`skewed-emacs`)
- **Base**: Custom Emacs configuration
- **Network Name**: `skewed-emacs` (accessible as `skewed-emacs:7080` from other containers)
- **Host Ports**: 
- `6942` → `6942` (ttyd web terminal)
- **MCP Service**: Available via `mcp__skewed-emacs__*` functions
- **Mount**: `~/projects` → `/projects`

### Gendl/GDL Containers

**Included with skewed-emacs** (free, open-source Gendl kernel):

| Container | Image | HTTP Port | Swank Port |
|-----------|-------|-----------|------------|
| `gendl-ccl` | `genworks/gendl:devo-ccl` | 9080 (host: 19080) | 4200 |
| `gendl-sbcl` | `genworks/gendl:devo-sbcl` | 9090 (host: 29080) | 4210 |

**Available via supplemental overlay repos** (licensed, commercial GDL with NURBS):

| Container | HTTP Port | Swank Port |
|-----------|-----------|------------|
| `genworks-gdl-smp` | 9098 | 4218 |
| `genworks-gdl-non-smp` | 9089 | 4209 |
| `genworks-gdl-enterprise-smp` | 9098 | 4218 |

Licensed users clone their supplemental repo as a sibling to `skewed-emacs/`,
run `./install`, then `./compose-dev up` picks up the overlay automatically.

All containers mount `~/projects` → `/projects` and join `skewed-network`.
Check the Dashboard (`*dashboard*` buffer) for current service health.

### Docker Network
- **Network Name**: `skewed-network`
- **Purpose**: Enables container-to-container communication
- **Key Benefit**: Allows SLIME connection from Skewed Emacs to Gendl Swank server

## Development Workflow

### 1. Basic SLIME Development
```elisp
;; In Skewed Emacs container, after slime-connect to gendl-ccl:4200
;; Load Quicklisp
(load-quicklisp)

;; Add project directories
(pushnew "~/projects/gendl/demos/" ql:*local-project-directories* :test #'equalp)

;; Enable development mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gornskew/skewed-emacs](https://github.com/gornskew/skewed-emacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
