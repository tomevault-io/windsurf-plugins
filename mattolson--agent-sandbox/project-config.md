---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

# Agent Instructions

This file provides guidance to agents when working with code in this repository.

## Project Overview

Agent Sandbox creates locked-down local sandboxes for running AI coding agents with minimal filesystem access and restricted outbound network. Enforcement uses two layers: an mitmproxy sidecar that enforces a domain allowlist at the HTTP/HTTPS level, and iptables rules that block all direct outbound to prevent bypassing the proxy.

Supported agents: `claude`, `codex`, `gemini`, `opencode`, `pi`, `copilot`, `factory`.

**Note**: During development of this project, the agent operates inside a locked-down container using the docker compose method. This means git push/pull and other network operations outside the allowlist will fail from within the container. Handle git operations from the host.

## Development Environment

This project uses layered Docker Compose files with a proxy sidecar. The managed CLI compose files live under `.agent-sandbox/compose/`, the active agent lives in `.agent-sandbox/active-target.env`, and the user-owned layered policy inputs live at `.agent-sandbox/policy/user.policy.yaml` plus `.agent-sandbox/policy/user.agent.<agent>.policy.yaml`.

The container runs Debian bookworm with:
- Non-root `dev` user (uid/gid 500)
- Zsh with minimal prompt
- Network lockdown via `init-firewall.sh` at container start
- SSH disabled (git must use HTTPS, URLs are auto-rewritten)

### Setup (one-time)

Build local images:
```bash
./images/build.sh
```

### Key Paths Inside Container
- `/workspace` - Your repo (bind mount)
- `/home/dev/.codex` - Codex state (named volume, persists per-project)
- `/commandhistory` - Bash/zsh history (named volume)

### Templates vs Runtime Config

`internal/embeddata/templates/` contains the embedded templates used by `agentbox init` to generate per-project files. Editing a template affects all future `init` runs across all projects.

`.agent-sandbox/` at the repo root is the generated runtime config for developing this specific project. Editing files here only affects the local dev environment.

Do not confuse the two. When changing how `init` generates files, edit templates. When adjusting the local dev setup, edit `.agent-sandbox/`.

## Architecture

Three components:

1. **CLI** (`cmd/agentbox`, `internal/`) - The Go `agentbox` command-line tool plus its runtime, scaffold, docker, version, and embedded-template packages.
2. **Images** (`images/`) - Base image, agent-specific images, and proxy image.
3. **Runtime** (`.agent-sandbox/compose/*.yml`) - Layered Docker Compose stack for developing this project.

The base image contains the firewall script and common tools. Agent images extend it with agent-specific software. The proxy image runs mitmproxy with the policy enforcement addon.

### Image Directory Structure

```
images/
├── base/              # Base image (Dockerfile + shell scripts)
│   └── stacks/        # Language stack installers (python, node, go, rust)
├── agents/            # Agent-specific images
│   ├── claude/
│   ├── codex/
│   ├── copilot/
│   ├── factory/
│   └── gemini/
├── proxy/             # mitmproxy + policy enforcement
│   └── addons/        # enforcer.py (domain allowlist enforcement)
└── build.sh           # Image build script
```

### Proxy Internals

The proxy runs `mitmdump` (headless mitmproxy) with `enforcer.py` as an addon. At startup, `images/proxy/entrypoint.sh` invokes `images/proxy/render-policy` (Python) to merge the layered policy files into a single effective policy. `render-policy` merges the active agent's baseline policy with user and devcontainer overlays using `images/proxy/render-policy`.

The proxy Dockerfile installs `PyYAML`. The enforcer addon and render script are both Python.

Available services and their domain allowlists are hardcoded in `images/proxy/addons/enforcer.py`. Adding a new service requires modifying `enforcer.py`. For one-off domains, use the `domains:` list in policy files instead.

## CLI

### Code Conventions

The CLI is written in Go with Cobra entrypoints under `cmd/agentbox` and implementation packages under `internal/`.

Style rules:
- Format Go changes with `gofmt`
- Keep command handlers thin and push behavior into reusable packages such as `internal/runtime`, `internal/scaffold`, and `internal/docker`
- Treat `internal/embeddata/templates/` as the live template source of truth

### Package Structure

```
cmd/
└── agentbox/

internal/
├── cli/
├── docker/
├── embeddata/
├── runtime/
├── scaffold/
├── testutil/
└── version/
```

### Commands

`init`, `switch`, `exec`, `up`, `down`, `logs`, `compose`, `bump`, `edit` (`compose`|`policy`), `policy` (`config`|`render`), `destroy`, `version`, and `completion`.

See `docs/cli.md` for command documentation.

## Network Policy

Two layers of enforcement:

1. **Proxy** (mitmproxy sidecar) - Enforces a domain allowlist at the HTTP/HTTPS level. Blocks non-allowed domains with 403.
2. **Firewall** (iptables) - Blocks all direct outbound. Only the Docker host network (where the proxy lives) is reachable.

### Policy Format

The proxy reads policy from `/etc/mitmproxy/policy.yaml`:

```yaml
services:
  - github  # Expands to github.com, *.github.com, *.githubusercontent.com

domains:
  - api.anthropic.com

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattolson/agent-sandbox](https://github.com/mattolson/agent-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
