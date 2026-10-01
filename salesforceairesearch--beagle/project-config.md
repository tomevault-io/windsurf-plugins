---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This file is **stable** — invariants, architecture, principles, slow-changing project shape. For everything that moves with the code (current phase, what shipped when, what's next), read the live-state pointers below.

## Live state — read these (not this file)

If a fact in CLAUDE.md ever conflicts with one of these, **the live source wins**. CLAUDE.md describes the project's slow-changing shape, not point-in-time state.

- **What shipped recently**: `git log --oneline -20` and `git diff main...HEAD` on the current branch.
- **Internal sub-package layout**: walk the filesystem (`tree -L 2 xrlenv xrlenv_plugins`); only the top level is in this file.
- **Test suite**: `.venv/bin/python -m pytest -q` (the `pyproject.toml` `testpaths` includes `xrlenv_plugins`); `.venv/bin/python -m mypy` for strict type-check; `.venv/bin/python -m ruff check` for lint.
- **Sphinx docs site**: `uv pip install -e '.[docs]' && .venv/bin/sphinx-build -W -b html docs docs/_build/html`. Audience: end users + external developers.

## What XRLEnv is

Infrastructure for **agentic RL training**. Two cores:

1. **Sandboxing** — Docker (phase 0, universal) and CubeSandbox microVM (phase 2, Linux+KVM only) backends, plus a Function-Call execution mode (phase 1).
2. **Orchestration** — control plane + per-node agents managing thousands of concurrent long-horizon rollouts across cloud VMs (GCP + AWS, manual provisioning) and a local laptop.

It is **not** a trainer, not a model server, not a generic code interpreter. The policy lives trainer-side; the SDK calls whatever `policy.act(obs)` callable the user supplies.

## Architecture (three-plane split)

```
Trainer plane   (GPU host, runs policy, consumes trajectories)
       │   gRPC bidi-stream (rollout RPC)
Control plane   (single Python process in phase 0: scheduler, registry,
       ▲          capacity estimator, template catalog, state store, admin UI)
       │   outbound bidi gRPC stream from each node (spec 21)
       │   no inbound listener on the node side (invariant 7)
Data plane      (xrlenv-node daemons running BackendAdapters)
       │   in-sandbox stub protocol (uds / vsock)
Sandboxes       (Docker containers, CubeSandbox microVMs)
```

Trainer plane knows RL but nothing about sandboxes. Data plane knows sandboxes but nothing about RL. Control plane is the only thing that knows both, only in the narrow shape "schedule this template, return a rollout session." Don't violate this split.

## Top-level layout

Only the slow-changing top level is here. Sub-package detail (per-module roles inside `xrlenv/control/`, `xrlenv/node/`, etc.) belongs in the filesystem — read it directly when you need it.

```
xrlenv/              core platform (control plane, node agent, SDK, backends, observability, admin)
xrlenv_plugins/      benchmark + EnvAdapter plug-ins (PEP-420 namespace package; never inside xrlenv/)
specs/               00–21 design specs (the design source of truth)
deploy/              bootstrap/refresh/bring-up + systemd units; registry/ (3 registry servers + ops scripts), node/ (provisioning scripts)
nodes.yaml           operator inventory (loader at xrlenv/control/nodes_yaml.py)
docs/                Sphinx site (user + external-developer docs)
notes/               internal phase-gate docs + audit/rebuttal cycle (audit.md/rebuttal.md gitignored)
tests/unit/          unit tests (plus xrlenv_plugins/**/tests/ for plug-ins)
examples/            build-plans/, deployment_run_book/, nodes.yaml.example
README.md            developer entry point (dev setup + workflow; points users to the Sphinx docs)
```


## Critical design rules to never violate

These are the load-bearing design invariants. Most past mistakes in this design came from forgetting one of them:

1. **Sandbox identity ≠ rollout identity.** Separate columns, separate lifecycles. Phase 0 destroys the sandbox at rollout finish, but no code may assume `sandbox_id == rollout_id` — spec 18 sessions break that equation.
2. **Capacity is released only on node-confirmed destroy.** "Destroy enqueued" is not "slot free."
3. **Trajectories are immutable after seal.** Late reward updates write a new record set, never mutate.
4. **Template manifests are immutable for the duration of a training run** — pinned by `(name, version, digest)` at run start.
5. **Outbound-only node transport.** Never add an inbound listener on the node-agent without an explicit phase note + spec 04/07/09 update.
6. **State store holds metadata; blobs live on disk or object store.** Trajectory bodies, snapshot artifacts, image layers never go into SQLite/Redis.
7. **`task_key` is fairness; `instance_id` is identity.** Don't conflate them.

Plus these cross-cutting principles:

- **Mechanism not policy.** XRLEnv core ships primitives (`task_key`, `group_id`, `cancel_rollout`, `cancel_group`, anti-affinity, `max_runs_per_task`). Engine-specific over-request / filter / cancel loops live in the trainer adapters (`xrlenv/adapters/{slime,verl}.py`), never in core. Slime and verl have fundamentally different patterns; forcing one shape warps the others.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceAIResearch/Beagle](https://github.com/SalesforceAIResearch/Beagle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
