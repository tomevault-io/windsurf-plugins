---
trigger: always_on
description: Guidelines for AI coding agents working on the Masterblaster codebase.
---

# Agents

Guidelines for AI coding agents working on the Masterblaster codebase.

## Project overview

Masterblaster (`mb`) is a Go CLI and daemon for managing StereOS-based
AI agent sandbox VMs. It targets aarch64 guests using HVF acceleration on
Apple Silicon Macs with two backends: QEMU and Apple Virtualization.framework.

The tool communicates with `stereosd` inside StereOS guests over vsock for
secret injection, shared directory mounting, health monitoring, and graceful
shutdown. Agent harnesses (Claude Code, Claude Code, Gemini CLI) are managed by
`agentd` inside the guest.

See SPEC.md for the full RFC specification.

## Build and test

```bash
make build        # Produces ./build/mb binary
make check        # Runs go tests in Dagger
make clean        # Removes the build/ directory
make fmt          # Formats code
make vet          # Runs go vet
```

Go 1.25+ is required. The Nix flake (`flake.nix`) provides a reproducible dev
environment with QEMU, Go, and build tools. Use `nix develop` or direnv.

## Architecture

The codebase follows the daemon + CLI client + vmhost child process pattern:

```
mb CLI  --[JSON-RPC over $config-dir/mb.sock]--> mb daemon (mb serve)
  daemon --[JSON-RPC over vmhost.sock]---------> mb vmhost (one per VM)
    vmhost --[QMP unix socket]----------------> QEMU process (child of vmhost)
    vmhost --[in-process]---------------------> Apple Virt VM (Vz framework)
    vmhost --[vsock/tcp]----------------------> stereosd (guest)
      stereosd --[tmpfs/unix socket]----------> agentd (guest)
```

Each VM gets a dedicated `vmhost` child process (`mb vmhost --name <name>
--backend <qemu|applevirt>`) that holds the hypervisor handle and exposes a
control socket (`vmhost.sock`). The daemon spawns vmhost processes, monitors
their health via PID liveness checks, and routes CLI requests to them. This
architecture enables crash isolation (one VM crashing doesn't affect others),
daemon restart survival (vmhost processes outlive the daemon), and mixed
backends (QEMU and Apple Virt VMs running simultaneously).

The default config directory is `$XDG_CONFIG_HOME/mb` (falls back to
`~/.config/mb`). It can be overridden via `--config-dir` flag, `MB_CONFIG_DIR`
env var, or `config-dir` in `config.toml`. Precedence: flag > env > config
file > default.

### Key packages

- **`main.go`** -- Thin shim entrypoint. Creates the root command (`NewMbCmd()`),
  registers persistent flags (`--verbose`, `--config-dir`), and wires in all
  subcommands. A `PersistentPreRunE` hook calls `mbconfig.Init()` to
  initialize Viper before any subcommand runs.

- **`pkg/mbconfig/`** -- Centralizes config directory resolution using Viper.
  `Init(cmd)` binds CLI flags, sets the `MB_` env prefix (so `MB_CONFIG_DIR`
  and `MB_VERBOSE` work), applies XDG defaults, and reads an optional
  `config.toml` from the resolved config dir. Exposes `ConfigDir()` and
  `Verbose()` for use by subcommands.

- **`cmd/<name>/`** -- Each subcommand gets its own package directory following
  the `New<Name>Cmd()` factory pattern. Packages: `serve`, `init`, `up`,
  `down`, `status`, `destroy`, `ssh`, `list`, `mixtapes`, `vmhost`. Commands
  are thin wrappers that delegate to the daemon via `pkg/client/`.

- **`cmd/vmhost/`** -- Hidden `mb vmhost` subcommand. The daemon spawns one
  vmhost process per VM; each holds the hypervisor handle and exposes a
  control socket. `vmhost.go` defines `NewVMHostCmd()`, the `runVMHost()`
  entrypoint, the `qemuController` adapter (implements `vmhost.VMController`),
  and the `bootVM()` dispatcher. Platform-specific files provide
  `bootAppleVirt()` and `getPlatformConfig()`:
  - `platform_darwin_arm64.go` -- Apple Virt controller + QEMU HVF config.
  - `platform_linux.go` -- QEMU KVM config.
  - `platform_darwin_amd64.go` -- Stub (unsupported).
  - `platform_other.go` -- Stub (unsupported).

- **`pkg/config/`** -- jcard.toml config parsing, validation, and defaults.
  `config.go` defines `JcardConfig` and `Load()`. `expand.go` handles
  `${ENV_VAR}` expansion. `marshal.go` provides TOML serialization and the
  default jcard.toml template. Always use `Load()` to get a validated config.

- **`pkg/daemon/`** -- The long-lived Masterblaster daemon. `daemon.go` defines
  the `Daemon` struct with a `sync.RWMutex`-protected VM map and unix socket
  listener. Each VM is tracked as a `managedVM` struct containing the vmhost
  client connection. The daemon no longer holds a `Backend` directly; it
  spawns vmhost child processes and delegates all VM operations to them via
  `pkg/vmhost.Client`. `rpc.go` defines the JSON-RPC wire format (`Request`,
  `Response`, `SandboxInfo`). The daemon manages `$config-dir/mb.sock` for CLI
  communication and `$config-dir/daemon.pid` for liveness.

- **`pkg/daemon/client/`** -- Thin JSON-RPC client for CLI commands to talk to
  the daemon. Provides `Up()`, `Down()`, `Status()`, `Destroy()`, `List()`.
  Also provides `EnsureDaemon(baseDir)` which auto-starts the daemon if not
  running (fork + setsid + poll). All CLI commands (`up`, `status`, `list`,
  `ssh`, `down`, `destroy`) call `EnsureDaemon` automatically so the daemon
  can rediscover running vmhost processes that may have survived a daemon
  restart.

- **`pkg/vm/`** -- VM lifecycle management.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [papercomputeco/masterblaster](https://github.com/papercomputeco/masterblaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
