---
trigger: always_on
description: This file applies to the entire repository. A more specific `AGENTS.md` or
---

# AGENTS.md

This file applies to the entire repository. A more specific `AGENTS.md` or
`AGENTS.override.md` in a subdirectory may refine these rules for that subtree.

## Project

SBP, Simple Bridge Panel, is a small self-hosted Go application for preparing
and managing a fresh VPN server. It provides a web panel, a privileged local
agent, SQLite state, component lifecycle management, access groups, credentials,
traffic counters, and update recovery.

Supported production target:

- fresh Ubuntu 24.04 LTS;
- Linux amd64;
- systemd and apt;
- a directly reachable public IPv4 address;
- root or sudo access for installation.

Do not broaden support claims without implementation and tests.
SBP 1.x starts from a fresh server. Do not add pre-1.0 migrations, adoption,
upgrade payloads, or cleanup paths.

## Repository map

- `cmd/sbp-panel`: executable entrypoint and command modes.
- `internal/panel`: unprivileged HTTPS UI/API, authentication, group access
  reconciliation, and traffic persistence.
- `internal/agent`: privileged allowlisted operations for systemd, apt, sysctl,
  Docker, credentials, metrics, and updates.
- `internal/store`: SQLite schema, queries, and transactions.
- `internal/panel/web`: embedded HTML, CSS, JavaScript, and SVG assets.
- `internal/buildinfo`: product name, repository, and release version.
- `deploy`: bootstrap, uninstall, and shell lifecycle assertions.
- `install.sh`: public one-line installer.
- `.github/workflows/release.yml`: CI and release bundle pipeline.
- `README.md`: concise user-facing overview and commands.
- `PLANS.md`: format and live record for complex execution plans.

## Sources of truth

Use this order when facts disagree:

1. current code and tests;
2. checked-in configuration and release workflow;
3. `README.md`;
4. comments, old plans, and historical discussion.

Never invent a protocol, version, port, feature, compatibility claim, or
recovery guarantee. Verify it in the current tree.

## Working process

1. Read this file, `PLANS.md`, the relevant code, and `git status` before
   editing.
2. Preserve unrelated and pre-existing user changes.
3. For a small, local, low-risk change, implement directly.
4. For multi-step, architectural, destructive, security-sensitive, or
   long-running work, create or update an ExecPlan in `PLANS.md` first.
5. Make the smallest coherent change that fully solves the problem.
6. Validate in proportion to risk, then inspect the final diff.
7. Update the plan's progress, decisions, discoveries, and outcome while the
   work proceeds. Do not leave a completed plan claiming unfinished work.
8. Do not commit, tag, publish a release, or mutate a live server unless the
   user explicitly asks.

An ExecPlan must state the desired outcome, constraints, observable acceptance
criteria, exact implementation context, validation commands, and recovery path.
It must be usable by a contributor who has only the repository and the plan.

## Safety invariants

- Treat the privileged agent as a root-equivalent surface.
- Never commit real passwords, cookies, private keys, UUID credentials, session
  tokens, server IP addresses, or production database contents.
- Never adopt or remove external Docker, tuning, containers, images, or files
  merely because their names resemble SBP resources.
- Destructive operations must use exact targets, ownership evidence, strict
  error handling, and post-action verification.
- Panel-only uninstall must not remove `/opt/vpn-panel-managed`, Docker,
  managed service containers or images, or network tuning.
- Component uninstall must respect dependency order and refuse unsafe removal.
- Keep install, update, retry, rollback, and uninstall operations idempotent.
- Persist desired state before or atomically with external mutations. If an
  operation spans SQLite and system state, define compensation or reconciliation.
- Preserve temporary rollback artifacts until replacement services pass health
  checks, then remove only proven SBP-owned staging and rollback paths.
- Every persistent file, log, cache, backup, meter, and runtime artifact must
  have an explicit owner, size or retention bound, cleanup path, and recovery
  purpose. Do not add unbounded logs, archives, caches, or polling state.
- Keep generated binaries, databases, credentials, logs, caches, coverage,
  editor state, temporary files, and build output out of the tracked tree.
- Do not weaken authentication, CSRF, rate limits, secret permissions, service
  isolation, checksum checks, or update rollback without an explicit security
  rationale and tests.

## Runtime invariants

- The panel process is unprivileged. Privileged actions go through the local
  agent socket and an allowlisted API.
- Group expiration is desired access state and must be reconciled with runtime
  credentials and routing rooms.
- Xray credential changes should use its runtime API. Do not reintroduce shared
  container restarts for ordinary add, toggle, or remove operations.
- AmneziaWG peer changes use `awg syncconf` and must not restart the shared
  container for ordinary add, toggle, or remove operations.
- Distinct protocol variants must use distinct component IDs, container names,
  managed directories, public ports, runtime APIs, and traffic namespaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silenceremember/sbp-panel](https://github.com/silenceremember/sbp-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
