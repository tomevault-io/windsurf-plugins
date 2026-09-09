---
trigger: always_on
description: This file guides Claude Code (claude.ai/code) or other AI agents
---

# Readymax (the Ready Room) — agent guidance

This file guides Claude Code (claude.ai/code) or other AI agents
working with this repo's environment, standalone or aboard a Basilisk
stack.  It is deliberately mixed-register: lore nouns are bound to
their referents in parentheses at first use, then used freely;
commands, identifiers, and warnings never take the voice.

## Overview

This setup provides a complete Lisp development environment with:
- **The ready room** (this repo's container, compose service
  `ready-room`): the Captain (a long-running Emacs daemon with the
  Readymax configuration) attended by the Protocol Officer (the
  lisply-mcp layer that receives arriving agents)
- **The Gendl rooms** (`bridge`, `engine-room`): 3D CAD/modeling
  residents with Common Lisp REPLs, answering the same lisply dialect
- **Network Integration**: the rooms share the ship's Docker network
  (its name minted fresh at each raising, kept in `basilisk/.ship`),
  which carries SLIME connections
- **MCP Services**: every room exposes its resident via the Model
  Context Protocol for external tool integration

## MCP Integration

The containers are now wrapped as MCP (Model Context Protocol) services, providing seamless integration with Claude Code and other MCP-enabled tools.

### Available MCP Services

MCP server names are the ROOM slugs.  The containers themselves wear
minted crew names that change at every full raising — read
`basilisk/.muster` or `docker ps -f label=basilisk.module=<room>`
fresh each session, never from memory.

**The ready room (this repo, Emacs Lisp):**
- **Service Name**: `mcp__ready-room__ready-room__lisp_eval`
- **Purpose**: Evaluate Emacs Lisp code remotely
- **Usage**: `mcp__ready-room__ready-room__lisp_eval(code="(+ 1 2 3)")`

**Gendl rooms (included, free and open-source):**
- `mcp__bridge__bridge__lisp_eval` — Gendl on Clozure CL (port 9080)
- `mcp__engine-room__engine-room__lisp_eval` — Gendl on SBCL (port 9090)

**Guild workshops (commercial Genworks GDL, via supplemental overlay repos):**
- `mcp__guild-workshop__guild-workshop__lisp_eval` — GDL with NURBS, SMP (port 9098)
- `mcp__guild-workshop-2__guild-workshop-2__lisp_eval` — non-SMP variant (port 9088)
- These are not included here. Licensed users receive a supplemental
  repo to clone as a sibling directory, then run its `./install` script to
  add Docker Compose and MCP config overlays into the stack.

**Ping Services:**
- `mcp__ready-room__ready-room__ping_lisp` - Check Emacs availability
- `mcp__bridge__bridge__ping_lisp` - Check Gendl CCL availability
- `mcp__engine-room__engine-room__ping_lisp` - Check Gendl SBCL availability

### MCP vs Raw HTTP

**Previous Approach (Deprecated):**
```bash
# Raw HTTP calls (no longer recommended)
curl -X POST http://localhost:7080/lisply/lisp-eval -d '{"code": "(+ 1 2 3)"}'  # only from inside container
```

**Current Approach (Recommended):**
```python
# Through MCP services (seamless with Claude Code)
mcp__ready-room__ready-room__lisp_eval(code="(+ 1 2 3)")
```


### Verification Commands

Check if workarounds are active:
```bash
# Test current environment via MCP
mcp__ready-room__ready-room__lisp_eval(code='(list (getenv "SHELL") shell-file-name (getenv "PATH"))')

# Test native compilation settings
mcp__ready-room__ready-room__lisp_eval(code='native-comp-jit-compilation')

# Test assembler accessibility
mcp__ready-room__ready-room__lisp_eval(code='(shell-command-to-string "which as")')
```

## Quick Start

### 1. Start the Environment

```bash
# The stack lives in the Basilisk repo, not this one.
cd ~/projects/basilisk

# Start the full container stack
./basilisk up

# Verify services are running via MCP
mcp__ready-room__ready-room__ping_lisp()      # Should return "pong"
mcp__bridge__bridge__ping_lisp()            # Should return "pong"
```

### 2. Connect to Development Environment

```bash
# Connect to Emacs in the ready room (the container's name is minted
# per raising, so look it up by room label)
docker exec -it $(docker ps -qf label=basilisk.module=ready-room | head -1) emacsclient -t

# From within Emacs, connect to Gendl SLIME
# M-x slime-connect RET bridge RET 4200 RET
```

## Container Details

### The Ready Room (compose service `ready-room`, this repo's image)
- **Base**: the Readymax Emacs configuration
- **Network Name**: `ready-room` (accessible as `ready-room:7080` from
  other containers; the container's own name is its keeper's minted
  name — read it fresh from `basilisk/.muster`)
- **Host Ports**: 
- `6942` → `6942` (web terminal — the gangway)
- **MCP Service**: Available via `mcp__ready-room__*` functions
- **Mount**: `~/projects` → `/projects`

### Gendl/GDL Containers

**Included in the base articles** (free, open-source Gendl kernel):

| Room (service) | Image | HTTP Port | Swank Port |
|----------------|-------|-----------|------------|
| `bridge` | `gornskew/gendl:devo-ccl` | 9080 | 4200 |
| `engine-room` | `gornskew/gendl:devo-sbcl` | 9090 | 4210 |

**Available via supplemental overlay repos** (licensed, commercial GDL with NURBS):

| Room (service) | HTTP Port | Swank Port |
|----------------|-----------|------------|
| `guild-workshop` (SMP) | 9098 | 4218 |
| `guild-workshop-2` (non-SMP) | 9088 | 4208 |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gornskew/readymax](https://github.com/gornskew/readymax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
