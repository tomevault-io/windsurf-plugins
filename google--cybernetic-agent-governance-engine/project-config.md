---
trigger: always_on
description: > **Reference architecture — deployable and live-testable.** CAGE demonstrates
---

# AGENTS.md — Contributor & AI-Agent Standards

> **Reference architecture — deployable and live-testable.** CAGE demonstrates
> governance patterns for AI systems. The codebase is fully deployable to a
> real GKE cluster (dev or prod), and live GKE testing is supported and
> expected — this is not a paper design. The "reference architecture" framing
> means the governance, compliance, and region-guard patterns below are
> illustrative models for adopters to adapt to their own environments, not
> that the system is restricted to local or simulated execution. Deployment,
> change-management, and region-guard rules below describe how to operate a
> live instance of CAGE; they are illustrative patterns for adopters, not
> mandatory production obligations imposed on this repository's maintainers.

This file defines standards for anyone (human or AI coding agent) contributing
to this repository. It is written in the tool-agnostic `AGENTS.md` convention
supported natively by most AI coding assistants (including Antigravity, Roo Code,
Cursor, Cline, GitHub Copilot, and Windsurf) — see
[Tool-Specific Configuration](#tool-specific-configuration) at the bottom.

## Table of Contents

1. [Commit Message Standard](#commit-message-standard)
2. [Branch Naming & Merge Strategy](#branch-naming--merge-strategy)
3. [Code Standards](#code-standards)
4. [Deployment Rules](#deployment-rules)
5. [Debugging Standards](#debugging-standards)
6. [Compliance Artifact Obligations](#compliance-artifact-obligations)
7. [Architecture & Design Standards](#architecture--design-standards)
8. [Documentation Standards](#documentation-standards)
9. [Answering Questions About This Repository](#answering-questions-about-this-repository)
10. [Tool-Specific Configuration](#tool-specific-configuration)
11. [Test Execution](#test-execution)

---

## Commit Message Standard

This project follows [Conventional Commits v1.0.0](https://www.conventionalcommits.org/).
Full detail (examples, self-validation checklist) lives in
[`CONTRIBUTING.md`](CONTRIBUTING.md#commit-message-standard). Summary:

**Format:** `<type>(<scope>): <short summary>` — subject line ≤ 72 characters.

**Types (exactly these 10):** `feat` | `fix` | `docs` | `style` | `refactor` |
`perf` | `test` | `chore` | `ci` | `revert`

**Scopes (use at most one):** `gateway` | `compliance` | `infra` | `governance` |
`tests` | `docs` | `ci` | `agentsight` | `advisor` | `nemo` | `opa`

**Rules:**
- Imperative mood ("add", not "added"/"adds")
- No trailing period
- Breaking changes: `!` after type/scope, plus a `BREAKING CHANGE:` footer
  (both must be present together, never just one)
- PR titles become squash-merge commit messages and must also follow this
  format

Before finalizing any commit message or PR title, self-check: type is valid,
scope (if present) is valid, subject ≤ 72 chars, imperative mood, no trailing
period, and breaking-change marker/footer are coupled correctly.

---

## Branch Naming & Merge Strategy

Full detail lives in [`CONTRIBUTING.md`](CONTRIBUTING.md#branch-naming-conventions).
Summary:

| Purpose | Pattern | Example |
|---|---|---|
| New feature | `feat/<short-description>` | `feat/redis-rate-limiter` |
| Bug fix | `fix/<short-description>` | `fix/oscal-uuid-collision` |
| Documentation | `docs/<short-description>` | `docs/stpa-control-diagram` |
| Refactor | `refactor/<short-description>` | `refactor/gateway-middleware` |
| CI / tooling | `ci/<short-description>` | `ci/pin-actions-sha` |
| Hotfix on release | `hotfix/<version>-<description>` | `hotfix/2.0.1-redis-timeout` |
| Release candidate | `rc-v<semver>` | `rc-v2.1.0` |
| Experiment / spike | `spike/<short-description>` | `spike/cbf-formal-proof` |

**Rules:** lowercase kebab-case only; description ≤ 30 characters after the
prefix; delete branches after merge; never work directly on `main` or `rc-v*`.

**Merge strategy: squash merge only, for every PR into `main` — no exceptions,
including release integration branches.** A `squash-merge-guard` CI job
(`.github/workflows/ci.yml`) fails the build on any two-parent merge commit
reaching `main`. Never suggest `git merge <branch>` into `main`, `git merge
--no-ff`, or GitHub's "Create a merge commit" / "Rebase and merge" options.
Always say: *"Use 'Squash and merge' on GitHub; confirm the pre-filled commit
message matches the PR title and follows Conventional Commits format."*

When asked to commit or push directly to `main` or `rc-v*`, refuse and instead
suggest a feature branch + PR.

---

## Code Standards

### Before creating any file in `src/`
- Prepend the Apache 2.0 license header for `.py`, `.ts`, `.tsx`, `.js` files
  (template below).
- Verify no secrets, credentials, or PII are embedded anywhere in the file.

### License Header — Python Template

```python
# Copyright 2026 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/cybernetic-agent-governance-engine](https://github.com/google/cybernetic-agent-governance-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
