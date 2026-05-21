---
trigger: always_on
description: **kamal_podman** is a Ruby gem that extends [Kamal](https://kamal-deploy.org) (Basecamp's container deployment tool) to use **Podman** instead of Docker as the container runtime. It works by monkey-patching Kamal's internals at load time, replacing all `docker` CLI calls with `podman` equivalents.
---

# CLAUDE.md — kamal_podman

## Project Overview

**kamal_podman** is a Ruby gem that extends [Kamal](https://kamal-deploy.org) (Basecamp's container deployment tool) to use **Podman** instead of Docker as the container runtime. It works by monkey-patching Kamal's internals at load time, replacing all `docker` CLI calls with `podman` equivalents.

- **Gem name**: `kamal_podman`
- **Current version**: 0.1.0
- **Pinned Kamal version**: 2.10.1 (exact pin — any Kamal update may break overrides)
- **License**: MIT
- **Ruby**: >= 3.0.0

## Architecture

### Core Mechanism: Monkey-Patching via `class_eval`

The gem's architecture revolves around reopening Kamal's classes at runtime:

1. `lib/kamal_podman.rb` loads the `kamal` gem first (all Kamal code)
2. Zeitwerk autoloads `KamalPodman::*` classes, **ignoring** `lib/overrides/`
3. `Kamal::Cli` namespace is **eager-loaded** so all classes exist before patching
4. All files in `lib/overrides/` are loaded via `Dir.glob` + `load()` — these use `class_eval` to patch Kamal classes
5. `override.rb` replaces the global `KAMAL` constant with `KamalPodman::Commander`

**Load order matters.** Kamal must be fully loaded before overrides are applied.

### Override Architecture: Two Layers

**Layer 1 — Binary swap** (`overrides/kamal/commands/base.rb`):
Overrides `docker()` on `Kamal::Commands::Base` to delegate to `podman()`. All subclasses inherit this automatically via Ruby method resolution — no per-class overrides needed for commands where the only difference is `docker` → `podman`.

```ruby
Kamal::Commands::Base.class_eval do
  def docker(*args)
    podman(*args)        # Layer 1: binary swap, inherited by all subclasses
  end

  def podman(*args)
    args.compact.unshift :podman
  end
end
```

**Layer 2 — Method-level overrides** (remaining files in `overrides/`):
For commands where Podman's syntax genuinely differs from Docker, individual methods are overridden on the specific class. Only add a Layer 2 override when the command flags, output format, or behavior differs — not just the binary name.

| Override file | Why it exists |
|---|---|
| `commands/prune.rb` | Podman has no `docker image prune --filter`; rewrites to `podman image ls` + shell piping |
| `commands/app/logging.rb` | Podman `logs` needs `2>&1` and different flag handling |
| `configuration/registry.rb` | Podman requires explicit `docker.io/` prefix (Docker assumes it) |
| `configuration/proxy/boot.rb` | Same `docker.io/` prefix for kamal-proxy image |
| `cli/main.rb` | Replaces server bootstrap to check for Podman, not Docker |

**Safety net:** `test/auto_discovery_test.rb` dynamically iterates all `Kamal::Commands::Base` subclasses and verifies `docker()` returns a podman command. This catches any new Kamal class added in a future version that doesn't work with the base swap.

### Key Classes

| Class | Purpose |
|---|---|
| `KamalPodman::Commander` | Extends `Kamal::Commander`, returns Podman-aware builder/commands |
| `KamalPodman::Commands::Podman` | Podman system commands (`installed?`, `running?`, `create_network`) |
| `KamalPodman::Commands::Builder` | Podman-based builder with validation (rejects remote/multi-arch/cloud) |
| `KamalPodman::Commands::Builder::Local` | `podman build` + `podman push` (stubs out buildx lifecycle) |
| `KamalPodman::Cli::Server` | Podman-aware server bootstrap (checks for Podman, not Docker) |

### Podman-Specific Differences from Docker

- **Registry**: Podman requires explicit registry prefixes (`docker.io/`) — Docker defaults to Docker Hub implicitly
- **Builder**: No buildx/buildkit equivalent — uses `podman build` + `podman push` directly
- **Prune**: Commands rewritten to use `podman image ls`, `podman rmi`, `podman ps`, `podman rm`
- **Network**: Creates `kamal` network via `podman network create`
- **No daemon**: Podman is daemonless — `running?` checks `podman version` instead of daemon status

## Directory Structure

```
lib/
  kamal_podman.rb              # Entry point — loads Kamal, sets up Zeitwerk, applies overrides
  kamal_podman/
    version.rb                 # VERSION constant
    override.rb                # Replaces global KAMAL constant
    commander.rb               # Custom Commander subclass
    cli/
      server.rb                # Podman-aware server bootstrap
    commands/
      podman.rb                # Podman system commands
      builder.rb               # Builder orchestrator
      builder/
        local.rb               # Local build implementation
  overrides/                   # NOT autoloaded by Zeitwerk — loaded manually via Dir.glob
    kamal/
      cli/
        main.rb                # Replaces server subcommand
      commands/
        base.rb                # Layer 1: docker()→podman() + container_id_for
        app/
          logging.rb           # Layer 2: Podman-specific logs/follow_logs
        prune.rb               # Layer 2: Podman-specific prune commands
      configuration/
        registry.rb            # Default registry = docker.io
        proxy/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phoozle/kamal_podman](https://github.com/phoozle/kamal_podman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
