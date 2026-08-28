---
trigger: always_on
description: This file is the machine-friendly entrypoint for agents and humans editing this
---

# Praxist Contributor Contract

This file is the machine-friendly entrypoint for agents and humans editing this
repository. Read it before making architecture, runtime, plugin, task, test, or
documentation changes.

## 1. Repository Identity

Praxist is an autonomous research platform.

The product nickname is Praxist.

The canonical Python package is `praxist`.

The canonical operator entrypoint is:

```bash
praxist start --task-path /path/to/task-project --daemonize --json
```

From a source checkout, run the same lifecycle through `uv`:

```bash
uv run praxist start --task-path /path/to/task-project --daemonize --json
```

The external task path model is mandatory. Real task projects are explicit
inputs, not bundled system source.

The active architecture documents are:

```text
docs/concepts/architecture.md
docs/concepts/runtime-model.md
```

The source documentation entry is:

```text
docs/index.md
```

When Codex or Claude Code is asked to install Praxist for a first-time user,
follow the dedicated OOBE contract before ordinary onboarding or takeover:

```text
docs/agents/oobe-install.md
```

The agent-managed OOBE lane and the local terminal wizard share setup profiles,
configuration, diagnostics, and skill registration. Installation stops after
readiness checks; project selection and takeover require a separate request
after the operator reads the first-task manual. The two lanes do not share an
interaction controller or a second persistent OOBE state store.
After `python3 -m pip install --index-url https://pypi.org/simple "praxist[agents,codex]"`, run
`praxist setup --agent-managed` and
follow its `next_required_action`; provider defaults, saved credentials, and a
passing doctor report do not count as an operator profile selection.

## 2. Current Source Layout

The important root directories are:

```text
praxist/        system package
praxist/core/   protocol, registry, replay, credentials, ledgers, stores
praxist/plugins/ generic reusable plugin catalog
praxist/infrastructure/ internal infra adapters and shims
praxist/product_usage/ opt-in pseudonymous usage protocol and client
praxist/testing/ offline fake workflow fixture support
services/product_usage/ separately deployed usage collector and retention service
examples/           complete runnable reference projects
templates/          tracked task templates and smoke fixtures
tests/              long-lived unit, conformance, workflow, hardening tests
docs/               source docs
scripts/            operator and build wrappers
tasks/              ignored local task workspace
```

The canonical package directory is `praxist/`. Do not introduce parallel
system-package directories. The following unrelated obsolete directory name is
also forbidden:

```text
auto_research/
```

Do not use them in active code, examples, commands, or new API reference pages.

Product usage is an observation boundary, not research-loop semantics. The
built-in client and closed event protocol live in `praxist/product_usage/`; the
neutral lifecycle projection lives in `praxist/infrastructure/`; collector and
deployment assets live in `services/product_usage/`. Observation receives only
validated aggregate lifecycle summaries after stable boundaries. It must not
read task content, result payloads, prompts, logs, paths, credentials, provider
details, or hardware details, and no capture/upload failure may alter a run
result, schedule, artifact, exit code, or control-flow decision.

## 3. Core Boundary

Core code belongs in `praxist/core`.

Core owns contracts and control-plane services:

- protocol dataclasses and event shapes;
- plugin discovery and resolution;
- credential references and redaction;
- model/provider profile contracts;
- budget requests, grants, ledgers, and usage records;
- prompt layout manifests and cache provenance;
- trajectory, artifact, finding, replay, and source snapshots;
- task project resolution;
- workflow stage interfaces;
- tool server interfaces;
- runtime interfaces;
- role skill loading;
- stable storage views.

Core must not contain task facts.

Core must not contain benchmark-specific prompts, roles, evaluations, audits, or
harness code.

Core must not shell out to a specific agent runtime directly. Runtime execution
goes through an `AgentRuntime` plugin.

Core must not make provider-specific API calls directly. Provider selection and
credential binding go through `ModelProvider` and `CredentialResolver` layers.

Core must not read a real task unless startup received an explicit task path.

## 4. Plugin Boundary

Generic reusable plugins belong under `praxist/plugins`.

Current plugin kinds include:

- `agent_runtime`;
- `model_provider`;
- `workflow_stage`;
- `tool_server`;
- `graph_maintainer`;
- `budget_policy`;
- `panel_topology`;
- generic `audit_rule`;
- generic `evaluation`.

Plugins are reusable system components. They are not user projects.

Plugin manifests use `plugin.yaml`.

Executable plugins declare an importable entrypoint in the manifest.

The code implementing a plugin must physically live inside the selected plugin
directory unless the plugin is explicitly manifest-only.

Manifest-only plugins are allowed only when the manifest is the complete

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sapientinc/PRAXIST](https://github.com/sapientinc/PRAXIST) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
