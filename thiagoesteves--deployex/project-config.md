---
trigger: always_on
description: DeployEx is an Elixir **umbrella application** that manages deployments for BEAM applications (Elixir, Erlang, Gleam).
---

# DeployEx - Agent Code Guide

## Project Overview

DeployEx is an Elixir **umbrella application** that manages deployments for BEAM applications (Elixir, Erlang, Gleam).
It monitors running nodes, performs full deployments and hot upgrades, manages TLS certificates, sends notifications, and exposes a Phoenix LiveView dashboard.
It does **not** use Docker or Kubernetes - it relies purely on OTP distribution.

The project version lives in `mix/shared.exs` (`Mix.Shared.version/0`) - do not hardcode it elsewhere.
Elixir requirement is `~> 1.16`; the exact toolchain is pinned in `.tool-versions` (Erlang 28.5.0.2, Elixir 1.19.5-otp-28).

---

## Umbrella Structure

```
apps/
  foundation/       # Core config, accounts, catalog, certificates, notifications, RPC, YAML parsing
  deployer/         # Release management, deployment engine, hot upgrades
  sentinel/         # Log streaming and watchdog monitoring
  host/             # Host system info, command execution, terminal sessions (tmux)
  deployex_web/     # Phoenix + LiveView web interface
mix/
  shared.exs        # Shared version, Elixir requirement, test coverage config used by all apps
devops/
  installer/        # Installer scripts
  releases/         # Per-OTP release configs (otp-27/, otp-28/ with their own .tool-versions)
  scripts/          # Operational scripts
```

Module naming follows the app prefix: `Foundation.*`, `Deployer.*`, `Sentinel.*`, `Host.*`, `DeployexWeb.*`.

---

## Development Commands

```bash
# Install deps and compile (warnings as errors)
mix do deps.get + compile --warnings-as-errors

# Run locally (needs a named node for OTP distribution)
iex --sname deployex --cookie cookie -S mix phx.server

# Run all tests with coverage
mix test --cover --warnings-as-errors

# Format code
mix format

# Check formatting (CI gate)
mix format --check-formatted

# Credo lint (strict mode)
mix credo --strict

# Check unused dependencies
mix deps.unlock --check-unused

# Dialyzer type checking
mix dialyzer

# Security audit
mix deps.audit
mix sobelow -r apps/deployex_web --exit --threshold medium --skip -i Config.HTTPS

# Generate docs
mix docs
```

Local runtime configuration comes from `deployex.yaml` at the repo root.
In production the path is provided via the `DEPLOYEX_CONFIG_YAML_PATH` environment variable.

---

## Safety and Permissions

Rules for AI agents about which actions may run without asking and which require explicit confirmation first.

**Allowed without asking:**

- Reading and listing files anywhere in the repository
- Compiling the project (`mix compile`)
- Formatting or format-checking specific files (`mix format <files>`)
- Linting (`mix credo`)
- Running a single test file or a focused subset (e.g. `mix test apps/<app>/test/<file>.exs`)

**Ask first:**

- Installing or updating packages (changing `mix.exs` deps, `mix deps.get` / `mix deps.update` after dependency changes)
- `git push` or any other action that leaves the local machine
- Deleting files or changing permissions (`rm`, `chmod`)
- Running the full test suite with coverage, `mix dialyzer`, or other long/expensive whole-project runs
- Building or running releases (`mix release`), or starting the app against real nodes

---

## Testing

- **Framework:** ExUnit - each app has its own `test/` directory and `test_helper.exs`
- **Test support:** `test/support/` per app; includes mocks, stubs, fixture helpers
- **Mocking:** `Mox` (behaviour-based) and `Mock` (module-level) - adapters in test env are all mocked (e.g. `Deployer.ReleaseMock`, `Foundation.RpcMock`)
- **Coverage threshold:** **94%** - enforced in CI via `mix test --cover`
- **Excluded modules:** Application callbacks, Adapter behaviours, Supervisor modules, and fixture helpers are excluded from coverage (see `ignore_modules` in `mix/shared.exs`)

When adding a new module, check if it needs to be added to the `ignore_modules` list in `mix/shared.exs` if it can't be reasonably unit-tested (Application, Supervisor, Adapter behaviour definitions).

---

## Code Patterns

### Adapter Pattern
Behaviour modules with a `Mock` counterpart for testing:
```
Deployer.Release.Adapter       # behaviour
Deployer.ReleaseMock           # Mox mock used in tests
```
Adapters are configured via `config/test.exs` for the test environment.

### GenServer Pattern
Stateful processes (monitors, catalog trackers, hot-upgrade servers) use GenServer.
Follow existing naming: `Module.Server` for the GenServer, `Module.Supervisor` for its supervisor.

### Phoenix LiveView
All real-time UI pages use LiveView.
Component files live under `apps/deployex_web/lib/deployex_web/live/`.

### Config Provider Pattern
`Foundation.ConfigProvider.Env.Config` and `Foundation.ConfigProvider.Secrets.Manager` are used in releases to inject configuration at runtime.

### PubSub
`Phoenix.PubSub` is used for broadcasting state changes.
Broadcast from business logic apps; subscribe from LiveView components.

---

## Code Preferences

Prescriptive rules for writing new code in this project.

- **Prefer supervised GenServers over ad-hoc concurrency.**
  Any long-lived, stateful, or background process must be a `GenServer` under a supervision tree.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thiagoesteves/deployex](https://github.com/thiagoesteves/deployex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
