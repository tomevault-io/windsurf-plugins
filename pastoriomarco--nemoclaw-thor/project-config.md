---
trigger: always_on
description: > **For LLMs (Claude / ChatGPT) helping the user with this repo:** read this
---

# AGENTS.md — NemoClaw-Thor scope and intended workflows

> **For LLMs (Claude / ChatGPT) helping the user with this repo:** read this
> file first. It defines what this repo does and does not own, points you
> to the operational docs, and lists the boundary rules that protect the
> user from repeating known mistakes. Read referenced docs *before*
> editing code or running scripts.

---

## Cross-repo authority map (start here)

The ManyForge stack spans three sibling repos. Each owns a different
question. Land in the right one before making a change:

| Question | Authoritative repo | AGENTS.md |
|---|---|---|
| **What is the contract / spec / ADR?** ("what should this do?") | `dev_ws/src/manyforge_specs/` | `manyforge_specs/AGENTS.md` |
| **What's in the implementation code / tests?** ("how is it written today?") | `dev_ws/src/manyforge/` | `manyforge/AGENTS.md` (one-page redirect to `manyforge_specs`) |
| **How does it run on Thor — serving, sandbox, integration?** ("how do we deploy and operate?") | this repo (`NemoClaw-Thor/`) | this file, plus the integration subtree's own [`manyforge/AGENTS.md`](manyforge/AGENTS.md) |

If your change spans repos (most do): start at
`manyforge_specs/AGENTS.md`, walk down to `dev_ws/src/manyforge/`, then
back here for the runtime artifacts. Don't write spec-level content
in this repo; it belongs upstream in `manyforge_specs`.

The Composer-assistant production default (lane + model) and the
runbook for bringing it up live in this repo at
[`manyforge/README.md`](manyforge/README.md) and
[`manyforge/docs/COMPOSER-ASSISTANT-RUNBOOK.md`](manyforge/docs/COMPOSER-ASSISTANT-RUNBOOK.md).

---

## Purpose

This repository provides a tested, reproducible setup for running
**NemoClaw** on **Jetson AGX Thor (SM110)** with local LLM inference, in
service of the **ManyForge assistant pipeline**.

It scopes to two concerns:

1. **NemoClaw onboarding on Thor** — a documented (ideally LLM-assisted)
   path from a clean Thor host to a working sandbox running OpenClaw or
   Hermes against a local vLLM endpoint.
2. **LLM serving for NemoClaw** — Thor-tuned vLLM and TRT-Edge-LLM
   container builds, model profiles, and launch scripts that expose an
   OpenAI-compatible `/v1` endpoint consumable by NemoClaw and (downstream)
   ManyForge.

The downstream consumer is the ManyForge composer running an assistant
agent backed by these models. This repo hosts the **deployment-side
integration runbook** that wires the OpenClaw runtime in a NemoClaw
sandbox to ManyForge's MCP surfaces (egress preset, skill bundle, MCP
server registration); see "ManyForge integration" below.

There are two assistant-provider bridges, both implementing ManyForge's
provider HTTP contract:

- **Direct vLLM lane** — `manyforge_assistant_bridge` on `:8100`, lives
  in the ManyForge repo. Talks straight to vLLM, runs the agent loop
  in-process. This repo does not own it.
- **OpenClaw lane** — `openclaw_assistant_bridge` on `:8200`, lives in
  this repo at [`manyforge/openclaw_assistant_bridge/`](manyforge/openclaw_assistant_bridge/).
  Adapter that dispatches into the NemoClaw `my-assistant` sandbox
  running OpenClaw, which runs the agent loop and calls vLLM through
  the OpenClaw gateway. **Production default since 2026-05-07.** This
  repo does own it.

---

## What this repo owns vs consumes

**Owns** (edit freely, version in this repo):

- `setup/` — sandbox/control-plane scope: prerequisite checks
  (`checks.sh`), sandbox runtime helpers (`sandbox-runtime.sh`), the
  local-provider configuration entrypoint (`configure-local-provider.sh`),
  the system-health entrypoint (`status.sh`), egress policies under
  `setup/policies/`, and the canonical onboarding workflow doc
  `setup/NEMOCLAW-OPENCLAW-WORKFLOW.md`.
- `serving/` — model-serving scope: vLLM model profile registry
  (`config.sh`), launch-arg matrix (`launch.sh`), entry points
  (`start-model.sh`, `start-duo.sh`), the Thor-specific Dockerfiles and
  runtime patches under `serving/docker/`, the agentic benchmark harness
  under `serving/agentic-bench/`, loose perf probes under
  `serving/benchmarks/`, and per-scope investigation/perf docs under
  `serving/docs/`.
- `manyforge/` — ManyForge integration scope: the deployment-side
  provisioner (`setup-manyforge-assistant.sh`), the egress preset
  (`policies/manyforge-composer.preset.yaml`), the OpenClaw-lane
  assistant-provider adapter (`openclaw_assistant_bridge/`, port
  `:8200`, production default), the bridge audit-log mount point
  (`bridge/`), and integration docs under `manyforge/docs/`.
- Top-level docs: `README.md`, this file (`AGENTS.md`), `VERSIONS.md`
  (single source of truth for verified versions across all three
  scopes), and `USER_QUICKSTART_MANUAL.md`.

This repo does **not** own the **direct-vLLM** assistant-provider
bridge service (`manyforge_assistant_bridge`). That bridge implements
ManyForge's HTTP contract and lives in the ManyForge repo
(`manyforge/manyforge_assistant_bridge/`); it consumes the vLLM
endpoint this repo's launch scripts expose. The bridge's architectural
design lives alongside the contract in
`manyforge_specs/docs/spec/485-assistant-bridge-architecture.md`.

This repo **does** own the **OpenClaw-lane** adapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pastoriomarco/NemoClaw-Thor](https://github.com/pastoriomarco/NemoClaw-Thor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
