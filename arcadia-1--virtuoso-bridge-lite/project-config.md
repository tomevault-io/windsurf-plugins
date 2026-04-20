---
trigger: always_on
description: Control Cadence Virtuoso via Python — remotely over SSH or locally on the same machine.
---

# AGENTS.md — AI Agent Guide for virtuoso-bridge-lite

Control Cadence Virtuoso via Python — remotely over SSH or locally on the same machine.

## Two modes

| Mode | When | Setup |
|---|---|---|
| **Remote** | Virtuoso on a server, you work locally | Set `VB_REMOTE_HOST` in `.env`, run `virtuoso-bridge start` |
| **Local** | Virtuoso on your own machine | Load `core/ramic_bridge.il` in CIW, use `VirtuosoClient.local()` |

## Prerequisites

1. **SSH**: `ssh my-server` must work without a password prompt.
2. **Virtuoso** (for SKILL execution): a running Virtuoso process on the remote (or local) machine.
3. **Spectre** (for simulation only): `spectre` on PATH, or set `VB_CADENCE_CSHRC` to a cshrc that adds Cadence tools to PATH.

> Virtuoso and Spectre are **independent** — you can run Spectre without the SKILL bridge, and vice versa.

## Install (both modes)

> **Use `uv` + virtual environment** — never install into the global Python.

```bash
uv venv .venv && source .venv/bin/activate   # Windows: source .venv/Scripts/activate
uv pip install -e .
```

## Step-by-step setup (remote mode)

**1. Generate config**

```bash
virtuoso-bridge init        # creates ~/.virtuoso-bridge/.env
```

**2. Edit `.env`**

> **Where to put `.env`:** By default the bridge checks `./.env` first, then `~/.virtuoso-bridge/.env`. For CLI commands such as `virtuoso-bridge start`, `--env FILE` has the highest priority.

```dotenv
VB_REMOTE_HOST=my-server              # SSH host alias from ~/.ssh/config
VB_REMOTE_USER=username               # SSH username on the remote
VB_REMOTE_PORT=65081                  # port for the bridge daemon on remote
VB_LOCAL_PORT=65082                   # local port forwarded via SSH tunnel

# Optional — only needed if `spectre` is not already on PATH in the remote shell.
# VB_CADENCE_CSHRC=/path/to/.cshrc   # cshrc that sets up Cadence tools on the remote
```

**3. Start the bridge**

```bash
virtuoso-bridge start
```

**4. Load SKILL in Virtuoso CIW**

```
load("/path/to/virtuoso-bridge-lite/core/ramic_bridge.il")
```

**5. Verify**

```bash
virtuoso-bridge status
```

**6. Connect from Python**

```python
from virtuoso_bridge import VirtuosoClient
client = VirtuosoClient.from_env()
client.execute_skill("1+2")  # VirtuosoResult(status=SUCCESS, output='3')
```

> **CIW output vs return value**: `execute_skill()` returns the result to Python but does **not** print in the CIW window. To also display in CIW, use `printf` explicitly:
> `client.execute_skill(r'let((v) v=1+2 printf("1+2 = %d\n" v) v)')`.
> See `examples/01_virtuoso/basic/00_ciw_output_vs_return.py`.

### Jump host setup

If you access Virtuoso through a bastion/jump host, set both hosts in `.env`:

```dotenv
VB_REMOTE_HOST=compute-host   # the machine running Virtuoso (NOT the jump host)
VB_JUMP_HOST=jump-host        # the bastion you SSH through
```

Common mistake: setting `VB_REMOTE_HOST` to the jump host. `VB_REMOTE_HOST` must be the machine where Virtuoso is actually running.

### Multi-profile setup

Connect to multiple Virtuoso instances simultaneously with `-p`. Profile names are **case-sensitive** and appended as suffixes to env var names.

```dotenv
# Default (no profile)
VB_REMOTE_HOST=server-a
VB_REMOTE_USER=user1

# Profile "worker1" — used with `-p worker1`
VB_REMOTE_HOST_worker1=server-b
VB_REMOTE_USER_worker1=user2
VB_CADENCE_CSHRC_worker1=/path/to/.cshrc.worker1
```

```bash
virtuoso-bridge start -p worker1
virtuoso-bridge status -p worker1
```

```python
from virtuoso_bridge.spectre import SpectreSimulator
sim = SpectreSimulator.from_env(profile="worker1")
```

> Profile suffixes are case-sensitive. `-p worker1` reads `VB_REMOTE_HOST_worker1`, not `VB_REMOTE_HOST_WORKER1`.

## First-time setup check

When a user first opens this project, run these checks **before anything else**:

### Remote check

**Three-host model** (common in EDA environments):
```
Your machine  ──SSH──►  Jump host (bastion)  ──SSH──►  Compute host (Virtuoso)
              VB_JUMP_HOST                   VB_REMOTE_HOST
```
`VB_REMOTE_HOST` must be the machine running Virtuoso, **not** the jump host. This is the most common misconfiguration.

1. **Check `.env`** — does it exist and have `VB_REMOTE_HOST` set?
   - If not: `pip install -e .` then `virtuoso-bridge init`, ask the user to fill in their SSH host.
   - Verify: `VB_REMOTE_HOST` = compute host (where Virtuoso runs), `VB_JUMP_HOST` = bastion (if any).

2. **Check SSH** — `ssh <VB_REMOTE_HOST> echo ok` (or via jump host if configured)
   - If this fails: tell the user to fix SSH first. The bridge assumes `ssh <host>` already works.

3. **Check Virtuoso** — `ssh <VB_REMOTE_HOST> "pgrep -f virtuoso"`
   - If no process: tell the user to start Virtuoso first.

4. **Start bridge** — `virtuoso-bridge start`
   - If "degraded": tell the user to paste the `load("...")` command in Virtuoso CIW.

5. **Verify** — `virtuoso-bridge status`

6. **Quick test** — `VirtuosoClient.from_env().execute_skill("1+2")`

### Local mode

No tunnel, no `.env`, no SSH. Just load `core/ramic_bridge.il` in Virtuoso CIW and connect directly.

```python
from virtuoso_bridge import VirtuosoClient
bridge = VirtuosoClient.local(port=65432)
bridge.execute_skill("1+2")
```

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arcadia-1/virtuoso-bridge-lite](https://github.com/Arcadia-1/virtuoso-bridge-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
