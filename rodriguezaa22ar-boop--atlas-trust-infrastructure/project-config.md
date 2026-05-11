---
trigger: always_on
description: This repository contains `atlas-lab-toolkit`, a local-first, shell-native toolkit for authorized security assessment workflows.
---

# AGENTS.md

## Project

This repository contains `atlas-lab-toolkit`, a local-first, shell-native toolkit for authorized security assessment workflows.

Atlas is the operator control plane. It coordinates scope, targets, recon, evidence, findings, validation, reports, retention, and release-trust artifacts without becoming a monolith or replacing the underlying domain tools.

Primary domain split:

- `atlas` is the operator-facing control plane.
- `wiremap` owns reconnaissance, capture, and evidence interpretation.
- `vector` owns ranked action lanes, bounded validation, sessions, and outcomes.
- `intelctl` owns direct shared-intel inspection.
- `labctl` owns build, release, target, and administration workflows.

Do not collapse these domains into one tool unless explicitly instructed.

---

## Current Maturity

Atlas is in an internal engineering/refinement phase.

When the project says a pillar is `ready`, interpret that as:

> ready for internal testing, refinement, and trust hardening.

It does not mean:

- production-ready
- externally audited
- enterprise-ready
- deployment-certified
- autonomous
- cryptographically immutable
- tamper-proof

Use precise language such as:

- ready-to-refine
- internal readiness
- release-trust candidate
- metadata-only
- audit-friendly
- retention-ready
- verifiable

Avoid overclaiming.

---

## Core Philosophy

Atlas favors verifiable integrity over assumed trust.

Every meaningful feature should support at least one of these:

- scope enforcement
- operator control
- evidence integrity
- append-only auditability
- validation discipline
- report clarity
- retention freshness
- release trust
- reproducibility
- known limitations

If a change does not support one of those goals, question whether it belongs.

---

## Safety Boundary

Atlas is for authorized assessment orchestration only.

Do not add features that provide or encourage:

- autonomous exploitation
- persistence
- destructive testing
- credential spraying
- denial-of-service workflows
- stealth/evasion behavior
- out-of-scope target expansion
- malware-like behavior
- unauthorized access

Target-touching workflows must preserve:

1. scope checks
2. capability classification
3. operator intent
4. approval gates when required
5. ledger events
6. evidence handling when applicable

---

## Capability Tiers

Use the existing capability tier model consistently:

- Tier 0: read-only
- Tier 1: passive recon
- Tier 2: active recon
- Tier 3: safe validation, explicit approval required
- Tier 4: intrusive validation, explicit ROE required
- Tier 5: destructive, blocked by default

When unsure, classify higher, not lower.

---

## Development Environment

Use the repository's Nix development environment.

Enter the dev shell with:

```bash
nix-shell
```

The dev shell is expected to provide tools such as:

- `shellcheck`
- `shfmt`
- `bats`
- `git`
- `gpg`
- `jq`
- `fd`
- `rg`
- `rsync`
- `tmux`

Do not assume host-global dependencies are available outside `nix-shell`.

---

## Standard QA Commands

Before declaring a change complete, run the strongest relevant gate.

Preferred full gate:

```bash
nix-shell --run './bin/dev-qa'
```

For targeted checks, use:

```bash
bash -n <changed-shell-file>
git diff --check
nix-shell --run './bin/dev-lint'
nix-shell --run './bin/dev-test tests/atlas.bats'
nix-shell --run './bin/dev-stress'
```

If a command was not run, say so clearly in the final response.

Do not claim tests passed unless they were actually run.

---

## Code Style

Atlas is shell-native.

Follow Bash discipline:

```bash
set -euo pipefail
```

Rules:

- Quote variables.
- Prefer `printf` over `echo` for controlled output.
- Avoid `eval`.
- Avoid parsing `ls`.
- Use `mktemp` for temporary files.
- Clean up temp files with `trap`.
- Check dependencies before using them.
- Keep read-only commands read-only.
- Preserve append-only semantics for audit/event state.
- Use existing helper functions before adding new ones.
- Keep functions small and domain-specific.

Do not introduce large rewrites unless explicitly requested.

---

## File and State Model

Atlas currently favors inspectable file-backed state:

- env records
- NDJSON records
- Markdown packets
- SHA-256 anchors
- metadata-only manifests

Do not migrate to SQLite, server state, hidden caches, or web-backed state unless explicitly requested.

Preferred state direction:

```text
file-backed contracts first
SQLite later only after event, graph, and packet contracts stabilize
```

Important state artifacts include:

- target env records
- operation env records
- scope snapshots
- `ledger.ndjson`
- evidence records
- findings records
- validation records
- reports
- handoff packets
- closeout manifests
- audit packets
- archive packets
- release trust packets

---

## Read-Only Command Rule

Commands documented as read-only must not mutate state.

Read-only examples include:

- `atlas v1 status`
- `atlas op audit`
- `atlas op archive`
- `atlas op verify`
- `atlas op audit-verify`
- `atlas op archive-verify`
- `atlas op trust-chain`
- `atlas release verify`
- `atlas target story`
- `atlas cycle`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodriguezaa22ar-boop/atlas-trust-infrastructure](https://github.com/rodriguezaa22ar-boop/atlas-trust-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
