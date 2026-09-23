---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Starting a task

Before starting any task, fetch `origin` and ensure the worktree is based on the latest
`origin/main`. Start changes from that revision on a focused feature branch. If local changes,
unpushed commits, or worktree state prevent a safe update, do not overwrite, reset, or discard
them. Stop and tell the user what must be reconciled.

Before editing or committing, verify both the working directory and current branch. This matters
when an agent can access multiple repositories or worktrees. Never assume that a command ran in
the intended checkout.

## Project overview

This repository is a collection of OpenTelemetry Collector recipes. Each recipe demonstrates a
specific configuration or integration and must be understandable and reproducible on its own.

There is no unit-test suite. Runtime smoke tests and Collector configuration validation are the
tests for this repository.

## Repository structure

Recipes are organized as a menu, by depth of effort:

- **`starters/`**: quick, local, single-concept recipes
- **`mains/`**: substantial, often Kubernetes-based, real-world recipes
- **`desserts/`**: advanced showcases and optional refinements
- **`sides/`**: shared building blocks reused by recipes, such as the LGTM stack and sample apps
- **Recipe structure**: each recipe folder is kebab-case and contains a `README.md`, its
  configuration (`otelcol.yaml` for local recipes, `otelcol-cr.yaml` for Kubernetes), and any
  supporting files. Configuration files always use `.yaml`, never `.yml`.

The root `README.md` contains a hand-maintained recipe index. Add, rename, or remove its row when
a recipe changes.

## Source-of-truth policy

Collector components evolve quickly. Before adding or changing component configuration, check
the component's upstream README in `opentelemetry-collector` or
`opentelemetry-collector-contrib`. Use current, non-deprecated component type names and verify
stability per signal.

Before a repository-wide version bump, confirm the latest official releases for:

- `open-telemetry/opentelemetry-collector-releases`
- `open-telemetry/opentelemetry-operator`

Update every affected recipe and validate each one. Do not infer an Operator-managed Collector
image version when a manifest does not pin one; verify it with a real deployment or describe the
limitation explicitly.

## Common commands

### Run a local Collector

```bash
otelcol-contrib --config otelcol.yaml
otelcol-contrib --config starters/<recipe-name>/otelcol.yaml
```

### Generate test data

```bash
# Send traces over OTLP/HTTP and attach a resource attribute
telemetrygen traces --otlp-http --otlp-insecure --otlp-attributes='recipe="<recipe-name>"'

# Send logs over OTLP/gRPC
telemetrygen logs --otlp-insecure --body "<log message>"

# Send metrics over OTLP/gRPC
telemetrygen metrics --otlp-insecure
```

`--otlp-attributes` sets resource attributes. Use `--telemetry-attributes` when a recipe needs
span, log-record, or metric data-point attributes that a processor or connector reads from the
telemetry item itself. This distinction must be verified when documenting a `telemetrygen`
command.

### Kubernetes setup

```bash
k3d registry create dosedetelemetria
k3d cluster create --registry-use k3d-dosedetelemetria:<port> dosedetelemetria

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.21.1/cert-manager.yaml
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/download/v0.158.0/opentelemetry-operator.yaml

kubectl create ns <recipe-name>
kubens <recipe-name>
```

### LGTM stack

```bash
docker run -p 3000:3000 -p 4318:4318 --rm -d grafana/otel-lgtm:0.30.2

kubectl create ns lgtm
kubectl apply -f sides/lgtm/lgtm.yaml
```

### TLS certificate generation

```bash
cfssl genkey -initca ca-csr.json | cfssljson -bare ca
cfssl gencert -ca ca.pem -ca-key ca-key.pem client-csr.json | cfssljson -bare client
cfssl gencert -ca ca.pem -ca-key ca-key.pem server-csr.json | cfssljson -bare server
```

Never commit generated private keys.

## Recipe README contract

Each recipe README follows this order:

1. **Title**: `# 🍜 Recipe: <Name>`
2. **Description**: one or two sentences describing what the recipe demonstrates
3. **Metadata table**: `**Signals**`, `**Runs on**`, and `**Key components**`
4. **🧄 Ingredients**: required tools and files
5. **🥣 Preparation**: numbered, reproducible steps using native `otelcol-contrib`,
   `telemetrygen`, Docker, or Kubernetes commands
6. **🎯 Key details**: optional configuration explanations; this is the only optional section
7. **😋 Tested with**: pinned versions as a plain bullet list, with no preamble

Keep recipes narrow and self-contained. Use realistic values rather than `foo` or `bar`.
Commands and examples must be directly runnable.

## Configuration conventions

### Component naming

In Collector YAML, always use the current, non-deprecated component type. Deprecated aliases may
still work, but new and updated recipes must not use them. The `Key components` metadata cells use
upstream Go package names such as `filelogreceiver` and `tailsamplingprocessor`; do not confuse

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpkrohling/otelcol-cookbook](https://github.com/jpkrohling/otelcol-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
