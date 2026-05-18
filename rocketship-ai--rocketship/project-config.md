---
trigger: always_on
description: This file provides guidance to AI coding agents (including this Codex CLI assistant) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (including this Codex CLI assistant) when working with code in this repository.

> **Versioning note:** Rocketship is still pre-1.0. There is **no backwards-compatibility requirement** for any interface, schema, or behaviour. Optimise for the current epic even if it means breaking past behaviour; do not preserve legacy code paths for compatibility unless the user explicitly asks.

## Rocketship Cloud v1 Snapshot

- Product focus: hosted cloud with GitHub SSO (device flow for CLI, OAuth for web) backed by our controlplane that mints Rocketship JWTs. Engine + worker still run tests via Temporal.
- Tenancy: **Org → Project**. Projects reference repo URL, default branch, and `path_scope` globs for mono-repo isolation. No “workspace” layer.
- Roles: project-level **Read** (view only) and **Write** (run/edit). Org Admins inherit Write on all projects. Tokens must carry explicit roles; missing roles are rejected.
- Git-as-SoT: UI/CLI can run uncommitted edits immediately (flagged as `config_source=uncommitted`). Approvals/merges happen in GitHub; Rocketship can optionally open PRs or commits if the user has push rights.
- Tokens: user JWT + refresh issued by controlplane; CI tokens are opaque secrets scoped per project with explicit permissions + TTL. Engine tags runs with `initiator`, `environment`, `config_source`, `commit_sha`/`bundle_sha` for auditability.
- Controlplane persists orgs/users/memberships in Postgres. Fresh logins return `pending` roles until the user creates or joins an org via `POST /api/orgs`.
- Guardrails: enforce path scopes, reject unknown RPCs in auth, clarify uncommitted runs, prefer minikube Helm flow for reproducible clusters.

## Architecture Overview

Rocketship is an open-source testing framework for browser and API testing that uses Temporal for durable execution. The system is built with Go and follows a plugin-based architecture.

There are 3 "server" components that make up the Rocketship system: Temporal, Engine, and Worker. The CLI is meant to communicate with the engine. There are three ways to run Rocketship:

1. **Minikube stack**: `scripts/install-minikube.sh` provisions Temporal + Rocketship inside an isolated cluster per branch.
2. **Self-hosted cluster**: Deploy the Helm charts to your own Kubernetes environment and connect the CLI remotely.
3. **Local processes**: Use `rocketship start server` / `rocketship run -af` for quick experiments without Kubernetes.

**Key Components:**

- **CLI (`cmd/rocketship/`)**: Main entry point that wraps the engine and worker binaries
- **Engine (`cmd/engine/`)**: gRPC server that orchestrates test execution via Temporal workflows
- **Worker (`cmd/worker/`)**: Temporal worker that executes test workflows using plugins
- **Plugins (`internal/plugins/`)**: Extensible system for different protocols (HTTP, delay, AWS services)
- **DSL Parser (`internal/dsl/`)**: Parses YAML test specifications into executable workflows
- **Orchestrator (`internal/orchestrator/`)**: Engine implementation that manages test runs and streaming logs

**Test Flow:**

1. YAML spec is parsed by DSL parser
2. Engine creates Temporal workflows for each test
3. Worker executes test steps using appropriate plugins
4. Results are streamed back via gRPC to CLI

## Development Commands

### Build and Install

```bash
make install        # Build CLI with embedded binaries and install CLI to $GOPATH/bin
```

### Testing and Quality

```bash
make lint && make test    # lint and test
```

### Embedded Binaries

The CLI embeds engine and worker binaries. Always run `make install` after modifying engine/worker code.

### Protocol Buffers

```bash
make proto          # Regenerate protobuf code from proto/engine.proto
```

### Documentation

```bash
make docs-serve     # Start local documentation server
make docs           # Build documentation
```

## Debugging and Logging

### Debug Logging

All processes (CLI, engine, worker) use unified structured logging from `internal/cli/logging.go`:

```bash
rocketship run --debug -af test.yaml    # Full debug output
rocketship run -af test.yaml            # Info level (default)
ROCKETSHIP_LOG=ERROR rocketship run -af test.yaml    # Errors only
```

Debug logging shows:

- Process lifecycle (start, stop, cleanup)
- Temporal connections and workflow execution
- Plugin execution details
- gRPC server initialization

DEBUG LOGGING IS EXTREMELY USEFUL DURING DEVELOPMENT.

### Advanced Debugging Techniques

When debugging complex issues with plugins or workflow state:

```bash
# Run with debug logging and save to file for analysis
rocketship run --debug -af test.yaml --env-file .env 2>&1 > /tmp/debug.log

# Search for specific plugin activity logs
cat /tmp/debug.log | grep -A 10 "SUPABASE Activity"

# Find logs for a specific step by Activity ID
cat /tmp/debug.log | grep -A 5 "ActivityID 47"

# Search for save/state-related logs
cat /tmp/debug.log | grep -E "(Processing save|saved values|State after step)"

# Find all logs for a specific workflow step
cat /tmp/debug.log | grep -A 20 "step 2:"

# Check for variable replacement issues
cat /tmp/debug.log | grep -E "(undefined variables|failed to parse template)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocketship-ai/rocketship](https://github.com/rocketship-ai/rocketship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
