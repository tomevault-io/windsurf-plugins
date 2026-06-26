---
trigger: always_on
description: This file helps AI Agents (and human developers) quickly understand the project structure and find relevant code.
---

# HiClaw Codebase Navigation Guide

This file helps AI Agents (and human developers) quickly understand the project structure and find relevant code.

## What is HiClaw

HiClaw is an open-source Agent Teams system that uses IM (Matrix protocol) for multi-Agent collaboration with human-in-the-loop oversight. It consists of a Manager Agent (coordinator) and Worker Agents (task executors), connected via an AI Gateway (Higress), Matrix Homeserver (Tuwunel), and HTTP file storage (MinIO or cloud OSS). Production-style deployments use the Kubernetes operator and Helm chart; local installs use Docker Compose scripts under `install/`.

## Project Structure

```
hiclaw/
├── hiclaw-controller/   # Kubernetes operator (Go): reconciles Worker, Manager, Team, Human CRDs
├── helm/                # Helm chart (K8s): Higress, Tuwunel, MinIO, controller, Manager CR, defaults
├── manager/             # Manager images: OpenClaw-based (Dockerfile) and CoPaw-based (Dockerfile.copaw)
├── worker/              # OpenClaw Worker image (shared base pattern; runtime also selected at deploy time)
├── copaw/               # CoPaw Python package source (published as e.g. copaw-worker on PyPI)
├── hermes/              # Hermes Python package source (Hermes Matrix worker runtime)
├── openhuman/           # OpenHuman Worker image: Rust core + native Matrix (channel-matrix feature)
├── openclaw-base/       # Base image: Ubuntu + Node.js + bundled agent assets + mcporter
├── shared/lib/          # Shared shell libs copied into images (hiclaw-env.sh, render-skills.sh, …)
├── install/             # Local install scripts (Docker Compose / embedded “all-in-one” stack)
├── scripts/             # Project-level utilities (e.g. replay-task.sh)
├── tests/               # Automated integration tests
├── docs/                # User-facing documentation
├── design/              # Internal design notes and API specs
├── changelog/           # Release notes fragments (current.md rolled into releases)
├── hack/                # Maintenance helpers (e.g. image mirror scripts)
├── migrate/             # Optional migration helpers
├── blog/                # Announcement / blog source
└── .github/workflows/   # CI: build images, tests, release automation
```

Logs and local artifacts (for example replay logs) stay out of git via `.gitignore`.

## Runtime model

**Worker runtimes** (per Worker CR `spec.runtime`, registry, or install defaults):

| Runtime   | Stack | Role |
|-----------|--------|------|
| `openclaw` | Node.js / OpenClaw (default) | Primary worker agent runtime |
| `copaw`    | Python / AgentScope via CoPaw | Alternative worker runtime |
| `hermes`   | Python / `hermes-worker` package | Alternative worker runtime (Matrix bridge + policies under `hermes/src/`) |

**Manager runtimes** (container env `HICLAW_MANAGER_RUNTIME`, CoPaw Manager CR / Helm `manager.runtime` where applicable):

| Runtime   | Behavior |
|-----------|-----------|
| `openclaw` (default) | OpenClaw gateway; primary Matrix channel uses the **message** tool pattern (see upstream OpenClaw / HiClaw manager config). |
| `copaw` | Python CoPaw workspace; Matrix traffic uses the **`copaw channels send`** CLI (see `start-copaw-manager.sh`). |

Hermes and OpenHuman are **Worker-only** runtimes in the API and Helm worker defaults; the Manager entrypoint in `start-manager-agent.sh` today starts **openclaw** or **copaw** only.

**Deployment runtime** (`HICLAW_RUNTIME`): local embedded stack vs `aliyun` vs `k8s` changes which bootstrap steps run inside the Manager container (for example Matrix registration and Higress setup are skipped or reduced in `k8s` because the controller owns them).

## `manager/agent/` layout (built into Manager images)

Agent-facing Markdown and skills under `manager/agent/` are copied to `/opt/hiclaw/agent/` in the image and synced into the Manager workspace by `upgrade-builtins.sh`. This tree is the single source of truth for builtin prompts and skills.

```
manager/agent/
├── AGENTS.md                    # OpenClaw Manager — primary bootstrap instructions
├── HEARTBEAT.md                 # OpenClaw Manager — periodic duties
├── SOUL.md                      # OpenClaw Manager — personality (often filled by onboarding)
├── TOOLS.md                     # Optional; referenced where applicable
├── skills/                      # Manager skills (shared by OpenClaw and CoPaw Managers)
│   └── <name>/                  # e.g. task-management, worker-management, … (each with SKILL.md)
├── skills-alpha/                # Experimental / optional skill packages
├── copaw-manager-agent/         # CoPaw Manager overrides
│   ├── AGENTS.md                # Replaces workspace AGENTS.md for CoPaw Manager
│   └── HEARTBEAT.md             # Replaces workspace HEARTBEAT.md for CoPaw Manager
├── worker-agent/                # Builtin OpenClaw Worker workspace template
├── copaw-worker-agent/          # Builtin CoPaw Worker workspace template
├── hermes-worker-agent/         # Builtin Hermes Worker workspace template
├── team-leader-agent/           # Team Leader agent template (Teams feature)
└── worker-skills/               # Extra worker skill templates (e.g. GitHub) pushed on demand
```

**Which files apply at startup**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentscope-ai/AgentTeams](https://github.com/agentscope-ai/AgentTeams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
