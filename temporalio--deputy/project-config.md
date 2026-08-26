---
trigger: always_on
description: Deputy is designed to run in a variety of environments, including local developer machines, CI systems, and isolated agent containers. This document provides an overview of the agent architecture, workflows, and key considerations for contributing to or modifying agent-related functionality.
---

# AGENTS.md

Deputy is designed to run in a variety of environments, including local developer machines, CI systems, and isolated agent containers. This document provides an overview of the agent architecture, workflows, and key considerations for contributing to or modifying agent-related functionality.

## Start here (docs map)

- Project overview + doc index: [`docs/README.md`](docs/README.md)
- Command reference: [`docs/commands/README.md`](docs/commands/README.md)
- Concepts (targets, policies, remediation): [`docs/concepts/README.md`](docs/concepts/README.md)
- Guides (CI, agents, plugins): [`docs/guides/README.md`](docs/guides/README.md)
- Reference (config, env vars, logging): [`docs/reference/README.md`](docs/reference/README.md)
- Development guides: [`docs/development/README.md`](docs/development/README.md)

## Repo map (high-level)

- Entry points: [`main.go`](main.go) -> [`internal/cli/cli.go`](internal/cli/cli.go) -> [`internal/cli/cmd/root.go`](internal/cli/cmd/root.go)
- CLI commands: [`internal/cli/cmd/`](internal/cli/cmd/)
- Core packages: [`internal/inventory`](internal/inventory), [`internal/analysis`](internal/analysis), [`internal/remediation`](internal/remediation), [`internal/policy`](internal/policy), [`internal/sbom`](internal/sbom), [`internal/proxy`](internal/proxy), [`internal/gitutil`](internal/gitutil)
- Services layer: [`internal/services`](internal/services) (clients/transport), server handlers in [`internal/server`](internal/server)
- Targets: [`internal/targets/detect.go`](internal/targets/detect.go)
- Policies + examples: [`policy/examples`](policy/examples)

## Foundations and extensions

- Inventory/SCA is built on [OSV-SCALIBR](https://github.com/google/osv-scalibr); Deputy extends it with additional extractors, plugins, and policy-aware workflows. Secrets scanning is implemented in Deputy’s own engine under [`internal/secrets`](internal/secrets).
- SBOM generation uses [Protobom](https://github.com/protobom/protobom) with CycloneDX/SPDX output in [`internal/sbom`](internal/sbom).

## Security-critical invariants

- Remote server mode must not access local filesystem or execute code. Use `ValidateRemoteTarget()` for target validation and guard any code execution with `localMode` checks. See [`internal/targets/detect.go`](internal/targets/detect.go) and [`internal/services/services.go`](internal/services/services.go).
- When adding new handlers:
  - Read-only network operations are OK remotely (generally safe, but SSRF risks should be considered).
  - Any filesystem access or code execution must be blocked unless `localMode=true`.
- Target detection is layered: the CLI does richer filesystem-aware detection; shared detection in `internal/targets` is pure pattern matching for RPC routing.

## Deployment + scale

- Deputy is designed to run anywhere: local CLI/in-process, local daemon, or remote shared service/SaaS with authn/authz and policy enforcement. See [`docs/reference/README.md`](docs/reference/README.md) and [`docs/commands/server.md`](docs/commands/server.md).
- Cloud-native + CI/CD use cases (including GitHub Actions) are first-class: [`docs/guides/github-actions.md`](docs/guides/github-actions.md).
- Extensibility is core: extractor plugins and sandbox plugins enable new ecosystems and runtimes without modifying core. See [`docs/guides/plugins.md`](docs/guides/plugins.md).

## Policy/CEL notes

- Policy inputs are proto messages; fields use `snake_case`. Keep variable names aligned with proto definitions. See [`docs/reference/policy-inputs.md`](docs/reference/policy-inputs.md) and [`api/deputy`](api/deputy).
- Use typed entrypoint constants (not string literals): [`internal/policy/entrypoints.go`](internal/policy/entrypoints.go).

## Proto / RPC / Observability

- Protos are the API boundary; generate code with Buf and keep proto changes paired with regenerated Go code. See [`api/deputy`](api/deputy), [`api/buf.gen.yaml`](api/buf.gen.yaml), and [`gen/`](gen/).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temporalio/deputy](https://github.com/temporalio/deputy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
