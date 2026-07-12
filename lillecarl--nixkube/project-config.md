---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Version Control

**Always use Jujutsu (jj) instead of Git** for all version control operations in this repository.

**Before committing**: Run `just precommit` (formats, lints, tests, and regenerates docs). This is the single command to run before every `jj commit`. Do not run `just fmt`, `just lint`, `just test`, or `just gendoc` individually when preparing a commit — use `just precommit` instead.

**Running `just` recipes**: You do NOT need `direnv exec . ` to run `just` recipes—they're already part of the dev environment. Only use `direnv exec . ` when running Python introspection code (e.g., `direnv exec . python3 -c ...` or `direnv exec . python3 << 'EOF'`) that depends on rebuilt dependencies in Nix.

## Project Overview

nixkube is a Kubernetes plugin system that injects Nix stores into pods using two complementary protocols:

1. **CSI (Container Storage Interface)** - Ephemeral volumes with explicit pod requests via volumeAttributes
2. **NRI (Node Resource Interface)** - Container runtime hooks for automatic injection via pod annotations

The system consists of:

1. **Node DaemonSet** - Runs on each Kubernetes node, implements both CSI and NRI protocols to mount Nix stores into pods
2. **Cache StatefulSet** - Central cache/coordinator that manages distributed builds and binary substitution
3. **Optional Builder Pods** - For offloading builds to dedicated builder nodes

The CSI layer supports two driver names for backwards compatibility:
- **nixkube** - Primary driver name (recommended for new deployments)
- **nix.csi.store** - Legacy driver name (enabled via `cfg.node.compat`)

## Architecture

### Build System

The project uses Nix with flake-compatish for backwards compatibility. Key build outputs are defined in `default.nix`:

- **Environments** (`environments/`): Separate Nix environments for cache and node, built using dinix (a service manager). Each environment:
  - Shares common services (openssh, nix-daemon, shared-setup)
  - Has role-specific services defined in separate modules
  - Builds for both x86_64-linux and aarch64-linux architectures
  - Is deployed as a minimal container with services managed by dinit

- **Kubernetes Deployment** (`kubenix/`): Uses easykubenix to generate Kubernetes manifests
  - `options.nix`: Defines module options and builds separate `cachePackage` and `nodePackage`
  - `cache.nix`: Cache StatefulSet with initContainer that copies environment artifacts
  - `daemonset.nix`: Node DaemonSet with CSI driver registration
  - SSH keys from `./keys/*.pub` are automatically imported as authorized keys

### Communication Flow

**Cache → Nodes**: The cache service watches for pods labeled `app.kubernetes.io/component=node` and updates `/etc/machines` with builder DNS names (`pod.name.nixkube-builders.namespace.svc.cluster.local`). This enables distributed builds.

**Nodes → Cache**: Node pods use the cache as a binary substitute via `ssh-ng://nix@pynixd?trusted=1` configured in `kubenix/config.nix`.

**CSI Protocol**: When a pod requests a volume with `storePath` or `nixExpr` or `flakeRef` in volumeAttributes, the node CSI driver:
1. Builds/fetches the requested Nix store path
2. Copies artifacts to the cache (if configured)
3. Mounts the store path into the pod using bind mounts

### Python Service Architecture

The nixkube service is a single multi-protocol daemon running both CSI and NRI servers concurrently:

**CSI Server** (`src/csi/`):
- `grpclib` for async gRPC protocol implementation
- `csi-proto-python` for CSI protobuf definitions (upstream spec)
- Handles explicit ephemeral volume requests via volumeAttributes
- Supports both nixkube and nix.csi.store driver names for backwards compatibility

**NRI Plugin** (`src/nri/`):
- ttrpc (transport-agnostic RPC) for NRI protocol
- Pod annotation parsing for automatic store injection
- OCI hook invocation for container initialization
- ZeroMQ-based coordination with build tasks

**Shared Infrastructure**:
- `kr8s` for Kubernetes API interactions and event reporting
- `nri-wait` for container initialization coordination
- Subprocess orchestration for Nix operations

Entry point (`src/cli.py`) starts both servers and features:
- Configurable logging via `/etc/nix/logging.json` ConfigMap
- Environment variable controls for feature flags (ENABLE_COMPAT_DRIVER, NRI_ENABLED)
- Graceful multi-server error handling

## Common Commands

Use the `justfile` for common development tasks. Run `just --list` to see all available recipes.

### Code Quality

**Format code before committing:**
```bash
just fmt
```

**Check formatting without changes:**
```bash
just check-fmt
```

**Run Python type checker:**
```bash
just lint
```

### Testing

**Run Python tests:**
```bash
just test
```

The integration test (runs in CI via `.github/workflows/integration-test.yaml`):
- Checks `/nix/store` is accessible in test pods
- Validates CSI driver registration
- Confirms cache and node pods are operational

**Build job** (runs once, pushes to cachix and container registry):
1. Builds and pushes Nix image
2. Builds and pushes cache/node environments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lillecarl/nixkube](https://github.com/Lillecarl/nixkube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
