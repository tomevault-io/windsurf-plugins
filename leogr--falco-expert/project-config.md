---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

---

## ⚠️ MANDATORY: Read This First

> **Note:** [`CLAUDE.md`](CLAUDE.md) is a symlink to this file. Claude Code users get these guidelines auto-loaded into the system prompt. Other AI agents must read this file explicitly.

**CRITICAL REQUIREMENT**: You MUST have [`AGENTS.md`](AGENTS.md) and [`README.md`](README.md) loaded in your context **every time** you:
- Start a new conversation or session
- Begin any new operation or task
- Resume after context compaction
- Return to this repository after working elsewhere

**Why this matters**: This knowledge base has specific conventions, era-based versioning, and strict guidelines that govern all operations. Working without these guidelines loaded in your context will lead to errors, inconsistencies, and wasted effort.

**How to comply**:
1. Ensure [`AGENTS.md`](AGENTS.md) is in your context (this file) — contains all working guidelines
2. Read [`README.md`](README.md) — contains the Table of Contents and current era info
3. Only then proceed with your task

**Do not skip this step.** Even if you think you remember the guidelines, re-read them. The guidelines may have been updated, and context compaction may have removed critical details.

---

## Project Overview

**Falco Expert** is a knowledge base for AI Agents focused on [Falco](https://falco.org), the Cloud Native Runtime Security tool (part of CNCF).

### Purpose

This repository serves as a centralized source of Falco knowledge for AI agents, enabling:
- Creation of Falco-focused AI agent skills
- Building dedicated Falco expert AI agents
- Assisting with Falco evolutive maintenance
- Providing an index for AI agents to retrieve Falco-related information
- Supporting evolution, refactoring, and development of the Falco codebase

## Primary Data Source

**https://github.com/falcosecurity** is the Falco project's official GitHub organization, hosting all the codebase and serving as the **ultimate source of truth** for all information.

### Organization Structure

The [`evolution`](refs/falcosecurity/evolution/) repository is the canonical source for understanding the falcosecurity organization:
- [`repositories.yaml`](refs/falcosecurity/evolution/repositories.yaml) - Master index of all 34+ repositories with scope and status
- [`maintainers.yaml`](refs/falcosecurity/evolution/maintainers.yaml) - Registry of all maintainers
- [`GOVERNANCE.md`](refs/falcosecurity/evolution/GOVERNANCE.md) - Project governance model

**Repository Scopes**:
- **Core**: Essential for building, installing, running, documenting, or using Falco (e.g., `falco`, `libs`, `rules`, `falcoctl`)
- **Ecosystem**: Optional extensions and integrations (e.g., `falcosidekick`, `driverkit`, `falco-talon`)
- **Infra**: Infrastructure support and testing (e.g., `test-infra`, `kernel-crawler`)
- **Special**: Unique purposes (e.g., `evolution`, `community`, `.github`)

**Repository Statuses**: Stable, Incubating, Sandbox, Deprecated

See [`digests/falcosecurity/evolution.md`](digests/falcosecurity/evolution.md) for the complete repository map.

## Era/Versioning System

Information in this repo is organized by Falco version "eras". The Falco version indicates the era of the collected information.

### Current Era: 0.44

- **Released**: May 26, 2026
- **Development cycle**: January 28, 2026 → May 26, 2026
- Patch versions (e.g., 0.44.x) belong to the same era

### Release Schedule

Falco typically releases 3 times per year:
- Last Monday of January
- Last Monday of May
- Last Monday of September

(Plus patch/hotfix releases in between. Actual dates may vary due to contingencies.)

### Component Version Mapping

- **Direct mapping**: Components shown in `falco --version` have 1-1 relationship with Falco version
- **Indirect mapping**: Ecosystem components without direct version mapping use development cycle time window
- Some component versions may span multiple Falco eras
- Some components may be specific to patch versions (still belong to corresponding era)

### Version Verification by Repository

**Universal method**: Run `git submodule status` from the repo root to see the pinned commit and tag for every submodule.

How to verify the era/version for each repository in [`refs/`](refs/):

| Repository | Verification method |
|------------|---------------------|
| `falco` | Git tag (e.g., `0.44.0`). Also: [`cmake/modules/falco-version.cmake`](refs/falcosecurity/falco/cmake/modules/falco-version.cmake) |
| `libs` | Git tag (e.g., `0.25.2`). Also: [`cmake/modules/versions.cmake`](refs/falcosecurity/libs/cmake/modules/versions.cmake) |
| `rules` | Git tag (e.g., `falco-incubating-rules-6.0.1`). Also: [`registry.yaml`](refs/falcosecurity/rules/registry.yaml) |
| `charts` | Git tag (e.g., `falco-9.0.0`). Also: [`charts/falco/Chart.yaml`](refs/falcosecurity/charts/charts/falco/Chart.yaml) → `version` and `appVersion` |
| `falcoctl` | Git tag (e.g., `v0.13.0`) |
| `plugins` | Git tag per plugin (e.g., `plugins/container/v0.7.1`). Also: individual plugin `CMakeLists.txt` or `go.mod` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leogr/falco-expert](https://github.com/leogr/falco-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
