---
trigger: always_on
description: The Datadog Agent collects metrics, traces, logs, and security events and forwards them to the Datadog platform. Written primarily in Go; this is the main repository for Agent versions 6 and 7.
---

# Datadog Agent - Project Overview for AI coding assistant

## Project Summary
The Datadog Agent collects metrics, traces, logs, and security events and forwards them to the Datadog platform. Written primarily in Go; this is the main repository for Agent versions 6 and 7.

## Project Structure

### Core Directories
- `/cmd/` - Entry points for various agent components
  - `agent/` - Main agent binary
  - `cluster-agent/` - Kubernetes cluster agent
  - `dogstatsd/` - StatsD metrics daemon
  - `trace-agent/` - APM trace collection agent
  - `system-probe/` - System-level monitoring (eBPF)
  - `security-agent/` - Security monitoring
  - `process-agent/` - Process monitoring
  - `privateactionrunner/` - Executing actions

- `/pkg/` - Core Go packages and libraries

- `/comp/` - Component-based architecture modules (Fx components)

- `/tasks/` - Python invoke tasks for development
  - Build, test, lint, and deployment automation

- `/rtloader/` - Runtime loader for Python checks

- `/packages/` - The declarations of what goes into each package we build for distribution.

- `/omnibus/` - The legacy build system. Still in use, but we are trying not to add to it.


## Development Workflow

### Critical: Always use `dda inv`, never raw `go` commands

This project uses extensive custom Go build tags. Most source files are ignored
by the standard Go toolchain unless the correct tags are passed. The `dda inv`
wrapper tasks (defined in `tasks/`) compute the right build tags automatically.

**Never run these commands directly:**

| Instead of | Use |
|---|---|
| `go build …` | `dda inv agent.build`, `dda inv cluster-agent.build`, etc. |
| `go test …` | `dda inv test --targets=./pkg/…` |
| `go mod tidy` | `dda inv tidy` |
| `go vet …` | `dda inv linter.go` |
| `golangci-lint run …` | `dda inv linter.go` |

This also applies to indirect usage — do not shell out to `go build` or
`go test` for compilation checks. If you need to verify that code compiles,
build the relevant component with `dda inv *.build`.

### Common Commands

```bash
dda inv install-tools                                 # one-time: install dev tooling
dda inv agent.build --build-exclude=systemd           # build the main agent
dda inv <component>.build                             # build a component (dogstatsd, trace-agent, system-probe, …)
dda inv linter.all                                    # run all linters
./bin/agent/agent run -c bin/agent/dist/datadog.yaml  # run the built agent
```

### Development Configuration
Place the dev config at `dev/dist/datadog.yaml` (e.g. `echo "api_key: 0000001" > dev/dist/datadog.yaml`); after building it is copied to `bin/agent/dist/datadog.yaml`.

## Key Components

### Check System
- Checks are Python or Go modules that collect metrics
- Located in `cmd/agent/dist/checks/`
- Can be autodiscovered via Kubernetes annotations/labels

### Configuration
- Main config: `datadog.yaml`
- Check configs: `conf.d/<check_name>.d/conf.yaml`
- Supports environment variable overrides with `DD_` prefix

### eBPF-based System Checks
- Checks using eBPF probes require system-probe module running
- Examples: tcp_queue_length, oom_kill, seccomp_tracer
- Module code (system-probe): `pkg/collector/corechecks/ebpf/probe/<check>/`
- Check code (agent): `pkg/collector/corechecks/ebpf/<check>/`
- System-probe modules: `cmd/system-probe/modules/`
- Configuration: Set `<check_name>.enabled: true` in system-probe config
- See `pkg/collector/corechecks/ebpf/AGENTS.md` for detailed structure
- Quick reference: `.cursor/rules/system_probe_modules.mdc` for common patterns and pitfalls

### eBPF Bazel Build
eBPF programs, runtime-compilation bundles, and cgo godefs are built with Bazel — see `bazel/AGENTS.md` (§ eBPF programs and code generation).

## Testing Strategy

### Unit Tests
Go tests run via `dda inv test --targets=<package>` (see the `dda inv` table above) or `bazel test //pkg/... //comp/...`; Python checks use pytest.

### End-to-End Tests
- E2E tests live in `test/new-e2e/tests/` and use the framework in `test/e2e-framework/`
- Tests provision real AWS, GCP or Azure infrastructure, deploy the agent, and assert payloads
  arrive in **fakeintake**. By default it forwards payloads to `dddev` org account.
- Key docs: `test/e2e-framework/AGENTS.md` (framework), `test/fakeintake/AGENTS.md`
  (intake mock), `docs/public/how-to/test/e2e.md` (setup & running)
- Use `/write-e2e` skill or read those docs directly to write new E2E tests
- Run locally: `dda inv new-e2e-tests.run --targets=./tests/<area>/...`

### Manual QA
- When the agent needs to be inspected in a given environment (e.g. EKS, ECS, a cloud VM) that is not easily reproducible locally, use the manual QA infrastructure.
- Full guide (scenarios, commands, stack lifecycle): `docs/public/how-to/test/manual-qa/index.md`

### Linting
- Python: various linters via `dda inv linter.python`
- YAML: yamllint
- Shell: shellcheck

## Build System

### Invoke Tasks
The project uses Python's Invoke framework for custom tasks. Run `dda inv -l` to list them.

### Build Tags
Go build tags control feature inclusion, some examples are:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDog/datadog-agent](https://github.com/DataDog/datadog-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
