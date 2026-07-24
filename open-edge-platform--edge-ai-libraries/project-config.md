---
trigger: always_on
description: **Edge AI Libraries** is a monorepo of optimized libraries, microservices, tools, and sample applications for building and deploying real-time AI solutions on edge devices. Components span computer vision, multimedia analytics, generative AI, time-series analytics, and model lifecycle management.
---

# Edge AI Libraries – AI Agent Instructions

**Edge AI Libraries** is a monorepo of optimized libraries, microservices, tools, and sample applications for building and deploying real-time AI solutions on edge devices. Components span computer vision, multimedia analytics, generative AI, time-series analytics, and model lifecycle management.

Components are **independently versioned and deployable**. Each component under `libraries/`, `microservices/`, `tools/`, `frameworks/`, and `sample-applications/` is self-contained with its own Dockerfile, Makefile, Helm chart, and tests.

## Licensing Requirements (Critical – All Files)

**All files must include:**

- SPDX license header: `SPDX-License-Identifier: Apache-2.0`
- Copyright line: `(C) <YEAR> Intel Corporation` (use current year for new files)
- Example:
  ```python
  # SPDX-FileCopyrightText: (C) 2026 Intel Corporation
  # SPDX-License-Identifier: Apache-2.0
  ```
- **Enforcement**: REUSE/license compliance checked in CI (see `codeql.yaml`, `zizmor-scan.yaml`)

## Language-Specific Skills (Load On-Demand)

Consult these based on the code you're working with. Skills reside under `.github/skills/`.

| Skill file | When to load |
|---|---|
| `.github/skills/security-review/SKILL.md` | Dockerfile, Compose, Helm, auth/authz, input parsing, file handling, secrets/logging, dependency upgrades, CI/CD workflow changes, privilege elevation |

> **Instruction Placement Policy**: Keep this file focused on high-level routing and architecture context. Detailed procedural checklists belong in skill files under `.github/skills/`. Avoid duplicating checklist content between this file and skills.

## Security Defaults (Always-On)

Apply secure-by-default behavior across all code generation, changes, and reviews, regardless of language or component.

- Prefer least privilege across code, services, identities, file permissions, APIs, containers, and workflows; avoid insecure defaults.
- Treat all external input as untrusted and validate format, type, range, and length at trust boundaries.
- Never hard-code or introduce secrets, credentials, keys, tokens, or passwords in source, tests, configs, or templates; use environment variables or approved secret-management mechanisms.
- Avoid exposing sensitive data in logs, traces, errors, metrics, or test artifacts.
- Prevent injection vulnerabilities by avoiding unsafe string construction and using safe, context-appropriate APIs.
- Prefer trusted, actively maintained dependencies and images; verify sources and pin versions where feasible.
- Avoid deprecated, unmaintained, or ambiguous packages.
- Do not suggest bypassing or weakening existing security checks or validations.
- Keep authorization checks server-side and close to protected resources.
- Avoid unsafe dynamic execution patterns (`eval`, `exec`, untrusted command construction).
- Prevent time-of-check/time-of-use (TOCTOU) race conditions in state-dependent checks (e.g., certificate validation.
- Do not assume trusted inputs, networks, or environments.
- Be explicit about assumptions and limitations.
- Fail safely and visibly.

## AI Output Trust Model

Treat AI-generated output as **untrusted draft code** until reviewed and tested.
Reject suggestions that bypass security controls for convenience or introduce unsafe defaults.

For detailed security review guidance, follow:
`.github/skills/security-review/SKILL.md`.

## Repository Structure

```
edge-ai-libraries/
├── libraries/          # Reusable AI/ML libraries (anomalib, datumaro, geti-sdk, model_api, …)
├── microservices/      # Standalone deployable services (dlstreamer-pipeline-server, model-registry,
│                       #   time-series-analytics, vlm-openvino-serving, audio-analyzer, …)
├── sample-applications/# End-to-end reference apps (chat-qna, document-summarization, …)
├── tools/              # Developer tooling (npu-monitor-tool, visual-pipeline-evaluation, …)
├── frameworks/         # Edge device enablement framework
└── .github/
    ├── workflows/      # Per-component CI (dlsps-*, timeseries-*, GENAI-*, modelregistry-*, …)
    └── skills/         # On-demand AI agent skill files
```

## Component Layout Convention

Each component follows a consistent layout:

```
<component>/
├── Dockerfile              # Container build definition
├── Makefile                # Standard targets: build, lint, test, coverage
├── README.md               # Quick start
├── helm/ or chart/         # Helm chart for Kubernetes deployment
├── docker/                 # Docker Compose files and supporting config
├── src/                    # Application source code
├── tests/                  # Unit and integration tests
├── docs/                   # Component documentation
├── requirements.txt         # Python runtime dependencies (or pyproject.toml / uv.lock)
└── document-versions.yaml  # Tracks documentation versioning
```

> Some newer components use `pyproject.toml` + `uv.lock` instead of `requirements.txt`. Prefer `uv` for lock-file-based installs in those components.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-edge-platform/edge-ai-libraries](https://github.com/open-edge-platform/edge-ai-libraries) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
