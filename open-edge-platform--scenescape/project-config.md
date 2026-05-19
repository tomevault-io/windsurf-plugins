---
trigger: always_on
description: This skill contains:
---

# SceneScape AI Agent Instructions

Intel® SceneScape is a microservice-based spatial awareness framework for multimodal sensor fusion. This guide enables AI agents to work effectively in this distributed system.

**Current Version**: Read from `version.txt` at repository root

## Licensing Requirements (Critical - All Files)

**CRITICAL - All files must include:**

- SPDX license header: `SPDX-License-Identifier: Apache-2.0`
- Copyright line: `(C) <YEAR> Intel Corporation` (use current year for new files)
- Example:
  ```python
  # SPDX-FileCopyrightText: (C) 2026 Intel Corporation
  # SPDX-License-Identifier: Apache-2.0
  ```
- **Enforcement**: REUSE compliance checking in CI
- Add to new files: `make add-licensing FILE=<filename>`

## Language-Specific Skills (Load On-Demand)

Consult these based on the code you're working with:

- **Python** (`.github/skills/python/SKILL.md`): Coding standards, imports, patterns
  - **CRITICAL**: 2 spaces for indentation (checked by `make indent-check`)
- **JavaScript** (`.github/skills/javascript/SKILL.md`): Frontend conventions
- **Shell** (`.github/skills/shell/SKILL.md`): Bash scripting guidelines
- **Makefile** (`.github/skills/makefile/SKILL.md`): Build system conventions
- **Testing** (`.github/skills/testing/SKILL.md`): Test creation frameworks

### Skills Caching Strategy

Skills are loaded on-demand based on task context to optimize token usage:

**Pre-Cached (Always Available)**:

- `copilot-instructions.md` (this file, always loaded)
- `python.md` (high frequency, pre-cached)
- `documentation-how.md` (high frequency, pre-cached)

**Loaded Automatically on Demand**:

- `testing.md` - Loaded when task involves tests or `test` keyword detected
- `javascript.md` - Loaded when `.js` files are being edited
- `shell.md` - Loaded when `.sh` files are being edited
- `makefile.md` - Loaded when Makefile or build system changes

Skills are detected and loaded based on file type, task keywords, and context signals. Explicitly request a skill if the auto-detection doesn't load it.

### Instruction Placement Policy (Critical)

- Prefer skill files under `.github/skills/` for detailed procedural rules.
- Keep this file focused on high-level routing and references to canonical skill documents.
- Avoid duplicating policy/checklist text across this file and skills.
- If overlap is found, retain one canonical source and replace duplicates with a short pointer.

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
- Do not assume trusted inputs, networks, or environments.
- Be explicit about assumptions and limitations.
- Fail safely and visibly.

## AI Output Trust Model

Treat AI-generated output as **untrusted draft code** until reviewed and tested.
Reject suggestions that bypass security controls for convenience or introduce unsafe defaults.

For detailed security review guidance, follow:
`.github/skills/security/SKILL.md`.

## Architecture Overview

**Core Components:**

- **Scene Controller** (`controller/`): Central state management for scenes, objects, cameras via gRPC/REST
- **Manager** (`manager/`): Django-based web UI, REST API, PostgreSQL schema management
- **Auto Camera Calibration** (`autocalibration/`): Computes camera intrinsics/extrinsics from sensor feeds (docker-compose still references as `camcalibration`)
- **DL Streamer Pipeline Server**: Video analytics pipeline integration (external service config in `dlstreamer-pipeline-server/`)
- **Mapping & Cluster Analytics** (`mapping/`, `cluster_analytics/`): Experimental modules (enable via `build-experimental`)
- **Model Installer** (`model_installer/`): Manages OpenVINO Zoo model installation

**Message Flow:**

```
Sensors → MQTT (broker) → Scene Controller → Manager/Web UI
           ↓                              ↓
       JSON validation           PostgreSQL (metadata only)
```

**Key Insight**: Scene Controller maintains runtime state (object tracking, camera positions); Manager provides UI/persistence layer. No video/object location data persists in DB—only static configuration.

## Build System Patterns

**Multi-component Docker builds** organized in `common.mk`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-edge-platform/scenescape](https://github.com/open-edge-platform/scenescape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
