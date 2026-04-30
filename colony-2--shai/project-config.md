---
trigger: always_on
description: Sandbox orchestration system for vibethis "boxes". It provisions ephemeral Docker containers using `.shai/config.yaml`, applies selective read-write overlays, and exposes both a CLI (`cmd/shai`) and a reusable API (`pkg/shai`) for launching workloads with strict workspace isolation.
---

# server/container

## Overview

Sandbox orchestration system for vibethis "boxes". It provisions ephemeral Docker containers using `.shai/config.yaml`, applies selective read-write overlays, and exposes both a CLI (`cmd/shai`) and a reusable API (`pkg/shai`) for launching workloads with strict workspace isolation.

## Architecture

### Container Internals (`internal/container/`)
- **Manager Interface**: Public contract kept for historical compatibility; production paths go through the sandbox API instead.
- **Info / Status / Mount**: Shared type definitions for callers that still reason about docker-like lifecycle states.

### Sandbox Runtime (`internal/shai/runtime/`)
- **EphemeralRunner**: Core engine that loads `.shai/config.yaml`, resolves resource sets, builds Docker options, and supervises containers (`ephemeral_runner.go`).
- **Config Loader (`internal/shai/runtime/config/`)**: Parses YAML, performs template expansion, validates resource/call definitions, and resolves apply rules.
- **MountBuilder**: Implements selective read-write overlays (base repo at `/src` read-only; provided paths become writable, with optional whole-workspace mode guarded by `.shai` dir protection).
- **Alias + Bootstrap**: Auxiliary services that expose host side commands inside the sandbox and generate the setup script executed before user processes run.
- **Docker Integration**: Lazy client detection (Desktop sockets on macOS, env overrides, rootless paths), image validation/pulls, and auto-removal semantics when containers exit.

### Shai CLI (`pkg/shai/` & `cmd/shai/`)
- **Sandbox API**: `pkg/shai.Sandbox` exposes `Run`, `Start`, and `Close`, delegating to the internal runtime.
- **SandboxConfig**: Public struct accepted by both CLI and library consumers (working dir, `.shai` config path, read-write overlays, resource sets, image overrides, verbosity, writers, exec specification).
- **CLI Wiring**: Cobra-based command that normalizes legacy flags, optionally collects read-write paths, captures template variables, and forwards everything to the sandbox runtime.

## Programmatic Usage

Shai exposes a generic API to run any process inside an ephemeral sandbox after loading `.shai/config.yaml`. Consumers provide a working directory, optional overlays, and (optionally) a `SandboxExec` describing the command to run post-setup.

### Key Types (pkg/shai)
```go
type SandboxExec struct {
    Command []string          // argv to exec after setup
    Env     map[string]string // extra env vars
    Workdir string            // container cwd; default: workspaceFolder from config
    UseTTY  bool              // default true; false => demuxed logs
}

type SandboxConfig struct {
    WorkingDir          string
    ConfigFile          string
    TemplateVars        map[string]string
    ReadWritePaths      []string
    ResourceSets        []string
    Verbose             bool
    PostSetupExec       *SandboxExec
    Stdout              io.Writer
    Stderr              io.Writer
    GracefulStopTimeout time.Duration
    ImageOverride       string
}

type Sandbox interface {
    Run(ctx context.Context) error
    Start(ctx context.Context) (*SandboxSession, error)
    Close() error
}

type SandboxSession struct {
    ContainerID string
    Wait(ctx context.Context) error
    Stop(ctx context.Context) error
    Close() error
}
```

### Example: Run a custom entrypoint
```go
import (
  "context"
  shai "github.com/colony-2/shai/pkg/shai"
)

func startProcess(ctx context.Context, repoRoot string, logs io.Writer) (*shai.SandboxSession, error) {
  cfg := shai.SandboxConfig{
    WorkingDir:     repoRoot,
    ReadWritePaths: []string{".vibethis", ".cache"},
    Stdout:         logs,
    Stderr:         logs,
    PostSetupExec: &shai.SandboxExec{
      Command: []string{"./bin/worker", "--name", "example"},
      UseTTY: false, // structured logs
    },
  }

  sandbox, err := shai.NewSandbox(cfg)
  if err != nil { return nil, err }
  sess, err := sandbox.Start(ctx)
  if err != nil { _ = sandbox.Close(); return nil, err }
  return sess, nil
}
```

## Behavior
- `.shai/config.yaml` fully defines image, workspace path, container user, resource sets, host-call shims, HTTP allow lists, and optional image overrides via path-based apply rules.
- Devcontainer lifecycle hooks are no longer processed. Instead, Shai emits a bootstrap script that wires environment variables, resolves host-call aliases, applies resource mounts, and then hands control to the requested command (or login shell).
- If `PostSetupExec` is nil: the runner switches to the configured user and launches an interactive login shell attached to the caller’s TTY.
- If `PostSetupExec` is set: the runner exports the provided environment, optionally `cd`s, and executes the supplied command with or without a TTY.
- Mounting: `/src` is read-only by default; entries in `ReadWritePaths` become writable bind mounts. Passing `.` switches the entire workspace to RW mode and then immediately remounts `.shai/` as read-only to guard configs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colony-2/shai](https://github.com/colony-2/shai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
