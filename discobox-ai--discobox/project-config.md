---
trigger: always_on
description: provides an actual functional runtime benefit inherent to the system, such as
---

# Repository Guidelines

## Project Structure

- Root module `github.com/discobox-ai/discobox`: stable contracts/API module.
  `go.work` joins it and the nested modules below.
- `base-image`: the shared Debian/Docker/systemd image the pool-agent and
  sandbox-agent images are built FROM (and, through the sandbox-agent image,
  every harness image).
- `vm-image`: the pool VM guest image every VM-backed provider boots, plus the
  libkrunfw-patched kernel libkrun alone needs. Both are released on their own
  lines and pulled from a registry at run time.
- `api`: the Server REST API contract: canonical OpenAPI documents
  (`api/openapi`), ogen-generated scaffolds (`api/gen`, `api/sandboxgen`),
  model aliases (`api/model`), and their generators (`api/internal`).
- `cli`: nested Go module for the `discobox` CLI.
- `cli/cmd/discobox`: CLI entrypoint.
- `cli/internal/cli`: CLI command implementation.
- `server`: nested Go module for the control plane implementation.
- `server/cmd/discobox-server`: HTTP server entrypoint.
- `server/internal/server`: server startup and HTTP router wiring.
- `server/internal/handlers`: HTTP API handlers.
- `server/internal/service`: aggregates the resource services and owns process-level service startup, shutdown, and default data.
- `server/internal/resources`: per-resource API services, lifecycle intent, and the reconcilers that converge it.
- `server/internal/reconcile`: level-triggered reconciliation engine (dirty set, lease-based claiming).
- `server/internal/sandbox`: sandbox provider Go contract, provider manager, and shared provider types.
- `server/internal/store`: database access, split by resource.
- `server/internal/database`: database setup and resolution.
- `server/internal/auth`: API authentication and authorization; `auth/sandbox` issues sandbox access tokens, `auth/poolagent` signs control-plane requests to pool agents.
- `server/providers`: Docker, VM, cloud, and pool-backed provider implementations.
- `pool-agent`: nested Go module for the pool agent implementation.
- `pool-agent/cmd/discobox-pool-agent`: pool agent entrypoint.
- `sandbox-agent`: nested Go module and image context for the sandbox agent runtime environment.
- `access`: nested Go module for `discobox-access`, the in-sandbox client of the agent credentials protocol.
- `termpane`: nested Go module; a reusable Bubble Tea component that draws a live terminal from any stream. No dependency on the rest of the repository.
- Other root-module packages (`execstream`, `harness`, `proxy`, `agentcreds`,
  `sandboxconfig`, `sandboxuser`, `runcca`, `endpoint`, and more): shared
  cross-module contracts and components; see the package map in `DESIGN.md`.
- `internal/cmd`: development and build tools, under `internal` because they are
  this repository's own and nothing outside it should build them:
  `discobox-docker-image-watch` (local base, pool-agent, sandbox-agent, and
  harness image rebuild watcher), `discobox-dev-lock` (one `task dev` loop per checkout),
  `discobox-gzip` (portable file compression for the build),
  `discobox-server-manifest` (the server manifest a release CLI is linked with),
  `discobox-installers` (stamps the install scripts a release uploads with that
  release and its binaries' digests), `discobox-installer-logo` (draws the TUI's
  mark into those scripts, from the TUI's own cell data), `discobox-winres` (the
  Windows version resource a release executable links, and the check that it did).
- `scripts`: shell and Node helpers the Taskfile and hooks run.
- `docs`: user/developer documentation and ADRs (`docs/adr`).
- `test`: Bats integration tests, the test-only harness stub image, and terminal performance tests.
- `DESIGN.md` / `REVIEW.md`: package-local design and review notes. Read the closest files in the current package and its parents before making design-sensitive changes.

## Git Workflow

Work directly on whatever branch is already checked out. If the session starts
on `main`, commit to `main`; if it starts on a feature branch, keep committing
to that branch.

Do not create new branches or worktrees unless explicitly told to.

## Commands

The toolchain comes from the Nix flake. Enter it with `nix develop`, or let
direnv do it via `.envrc`; every command below assumes that shell.

Use Taskfile targets through the Go tool-managed `task` binary:

```bash
go tool task --list
```

Common targets:

```bash
go tool task test       # root module tests
go tool task test:all   # root and nested module tests
go tool task check      # static checks
go tool task check-hooks # wait for background hook work and report what failed
go tool task rerun-hooks # re-run failed or never-run hooks
go tool task generate   # regenerate generated files
go tool task build      # build all binaries and local Docker images
```

What CI runs, and what to run before pushing something build-related:

```bash
go tool task ci:check   # check, the windows/amd64 cross type-check, and Dockerfile COPY paths
go tool task ci:test    # every module's tests, the way CI runs them
go tool task verify     # fmt, go.mod, generated files, and Mermaid are current
```

At the end of a code-changing task, run `go tool task check-hooks` before

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [discobox-ai/discobox](https://github.com/discobox-ai/discobox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
