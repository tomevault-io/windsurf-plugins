---
trigger: always_on
description: cicd-sensor is an eBPF-powered CI/CD runtime security sensor.
---

# Repo Instructions

cicd-sensor is an eBPF-powered CI/CD runtime security sensor.
The source of truth for design is `docs/`.

## AI agent workflow

This repo is developed through collaborative engineering, not autonomous "vibe coding".

AI agents should first investigate the code, docs, runtime behavior, and tradeoffs; then help write down the design or plan; then implement the agreed change. Do not skip directly from a rough idea to a finished PR.

Before changing external state such as creating, closing, reopening, or merging GitHub Issues / PRs, pushing branches, or publishing releases, show the exact target and proposed content, then wait for explicit approval.

Use ignored `work_docs/` for local temporary design drafts, progress notes, investigation logs, and result reports. Do not place local-only working notes at the repository root.

## What to read first

- `docs/index.md` — project goal and supported platforms
- `docs/user-guide/overview.md` — runner environments and usage models
- `docs/developer-guide/overview.md` — repository layout and subsystem reading order
- `docs/developer-guide/agent.md` — Job / Scope / JobRegistry / KernelTracker model
- `docs/developer-guide/agent-ownership-boundaries.md` — Agent / JobRegistry / Job / JobScopeState ownership rules and per-Job evaluation rationale
- `docs/developer-guide/ebpf-runtime.md` — cgroup v2 tracking, BPF map boundary, eBPF code style and contribution contract
- `docs/developer-guide/manager.md` — config and log delivery boundary
- `docs/developer-guide/rule-engine.md` — RuleSet / RuleModifier / CEL flow

## Detailed rules

The files below add detail that only applies when touching specific paths. **They apply to any AI coding agent working in this repo — Claude Code, Codex, Gemini, or otherwise. The `.claude/` directory is only there because Claude Code auto-loads from that path; the contents are not Claude-specific.** Read the relevant one before changing code in that area.

| File | Apply when |
| --- | --- |
| `.claude/rules/10-code.md` | Touching `**/*.go`, `go.mod`, `go.sum`. Go baseline, tooling, style, comments. |
| `.claude/rules/20-testing.md` | Writing or reviewing tests. Required test-case table and coverage-perspective table. |
| `.claude/rules/30-cel-rules.md` | Touching `rules/**`, `internal/rule/**`. RuleSet / RuleModifier schema, CEL surface, event-type sources. |
| `.claude/rules/40-supply-chain.md` | Touching `.github/**`, `.gitlab-ci.yml`, Dependabot, or Renovate config. SHA pinning and cooldown. |
| `.claude/rules/50-design-docs.md` | Writing or updating local Design Docs for large feature additions or substantial behavior changes. |

## Build and test

- `make build` — build agent + manager binaries (Linux).
- `make test` — run unit tests.
- `go test -race ./...` — race detector (required for concurrency changes).
- `make check` — `generate` + `test` + `rules-validate` + `rules-bundle-validate` + `diff-check` (the gate this repo uses before commit).
- `make integration` / `make bpf-integration` — integration suites (need privileges; may require Linux).
- `make rules-validate` — validate baseline rule YAML.
- `make generate` — regenerate protobuf and bpf2go output (run after touching `proto/` or BPF C sources). BPF compilation runs through Docker, so macOS / Windows hosts work as long as Docker is available.

## Repository layout

| Path | Role |
| --- | --- |
| `cmd/cicd-sensor` | Agent CLI |
| `cmd/cicd-sensor-manager` | Manager server |
| `cmd/cicd-sensorctl` | Report / attestation / rule validation CLI |
| `internal/agent` | Agent runtime (Listener, JobRegistry, Job, Scope, KernelTracker) |
| `internal/rule` | RuleSet / RuleModifier schema, resolution, CEL compile and evaluate |
| `internal/manager` | Config service, collector ingest, output routing |
| `internal/ctl` | Report and attestation generation |
| `proto/` | Connect / protobuf wire schema |
| `rules/` | Baseline rule YAML |
| `docs/` | Design source of truth (mdbook published from `cicd-sensor.github.io`) |

## Agent components

The Agent is built from several components, each owning a different boundary. Before writing code, identify which component owns the state, lifecycle, or interface you are touching. Do not let responsibilities leak across components.

| Component | Owns |
| --- | --- |
| `Agent` | Top-level process orchestrator. Owns provider/runner selection, socket lifecycles, manager clients, host config cache startup, and shutdown. |
| `Listener` | Unix-socket HTTP entrypoint. Owns provider routes, peer credentials, request trust checks, and dispatch into JobRegistry. |
| `NRI observer` | Separate `cicd-sensor nri` process. Observes containerd NRI events and sends Kubernetes staging requests to the Agent. |
| `ManagerClient / host config cache` | Manager config fetch boundary and cached host config for Kubernetes host paths. Does not own Jobs or KernelTracker state. |
| `JobRegistry` | Active jobs catalog and KernelTracker binding. Creates scope state, composes tracking primitives, and finalizes Jobs. |
| `Job` | One CI/CD job's lifecycle, identity, and event worker. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cicd-sensor/cicd-sensor](https://github.com/cicd-sensor/cicd-sensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
