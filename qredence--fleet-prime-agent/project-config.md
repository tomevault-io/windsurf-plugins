---
trigger: always_on
description: `fleet-prime-agent` is the Fleet Prime product: a standalone web interface and launcher built around a checksum-pinned stock Prime Agent runtime.
---

# Fleet Prime — Agent Instructions

`fleet-prime-agent` is the Fleet Prime product: a standalone web interface and launcher built around a checksum-pinned stock Prime Agent runtime.

Fleet owns the product/interface layer.

Prime Agent owns the execution engine.

This file defines repository-wide execution rules.

## Working model

Before changing code:

1. Inspect the relevant implementation and its tests.
2. Search for existing implementations, types, helpers, and established boundaries before adding new abstractions.
3. Read `ARCHITECTURE.md` when the task affects ownership, lifecycle, runtime boundaries, protocol behavior, sessions, transports, or cross-package behavior.
4. Read a file under `docs/guides/` only when performing the specific operation that guide covers.

Treat current code, tests, workspace configuration, `PRIME_AGENT_RUNTIME.json`, package manifests, and executable validation as authoritative.

Documentation explains the system but does not override executable contracts.

## Execution

For clear, reversible work, inspect the relevant context and proceed.

Prefer the simplest implementation that fully satisfies the request.

Keep changes focused.

Reuse existing modules, types, packages, and abstractions when they fit instead of creating parallel mechanisms.

Remove obsolete, duplicated, or superseded code when its removal is safe, verified, and directly relevant to the task.

For multi-file or architectural work, maintain a concise task list and validate meaningful stages as you go.

Do not modify unrelated files merely to make a diff cleaner.

## Git and external effects

Preserve pre-existing staged, unstaged, and untracked changes.

Do not reset, clean, stash, overwrite, or revert changes you did not make.

Do not use destructive or broad Git operations such as:

```bash
git add .
git add -A
git reset --hard
git clean -fd
git checkout .
git stash
git commit --no-verify
```

When staging is explicitly requested, stage only files changed for the current task.

Do not commit, amend, push, open or merge pull requests, publish packages, create releases, deploy, or perform other externally visible actions unless explicitly requested.

Never force-push.

Never expose credentials, tokens, `.env` values, provider secrets, or private runtime data.

## Product boundary

Fleet Prime is an independent product layer over the stock upstream Prime Agent runtime.

Do not vendor, copy, or patch Prime Agent source into this repository.

Engine behavior such as:

* provider implementations;
* model implementations;
* daemon internals;
* upstream CLI behavior;
* core runtime behavior;
* upstream protocol implementation;

belongs upstream unless Fleet is adapting or presenting an already-supported upstream capability.

`PRIME_AGENT_RUNTIME.json` is the authoritative runtime pin.

When the runtime changes, keep every package reference to the stock runtime aligned with that manifest and follow `docs/guides/upstream-runtime.md`.

Do not encode the currently pinned runtime version into `AGENTS.md` or `ARCHITECTURE.md`.

## Web architecture boundaries

The primary dependency direction is:

```text
browser
  ↓ HTTP / NDJSON / SSE
web/app
  ↓ typed Fleet protocol
web/server
  ↓ supported Prime Agent runtime / daemon API
stock Prime Agent runtime
```

Additional responsibilities:

```text
web/protocol
  → browser-safe transport contracts

packages/fleet-web
  → Fleet launcher/distribution
```

### Browser boundary

`web/app` must not import `prime-agent` or other upstream execution-runtime packages.

Browser code communicates with execution through Fleet's typed HTTP/stream protocol.

Do not expose raw runtime objects to browser code merely because the in-process implementation makes them technically accessible.

### Server boundary

`web/server` is Fleet's adapter boundary to Prime Agent.

It owns:

* runtime/daemon connection;
* Fleet-to-Prime session adaptation;
* upstream event mapping;
* browser-safe presentation mapping;
* HTTP handlers;
* replay infrastructure;
* pending interactions/dialogs;
* Fleet-managed presentation state;
* runtime compatibility handling.

Do not move Prime Agent-specific execution knowledge into `web/app`.

### Protocol boundary

`web/protocol` owns browser-safe wire types.

Server and browser should depend on the shared protocol instead of independently recreating equivalent event shapes.

Protocol evolution should remain additive or explicitly versioned/capability-gated when compatibility requires it.

See `docs/reference/adapter-contract.md` for the durable transport/privacy contract.

## Prime Agent connection invariants

Use the supported Prime Agent connection abstraction rather than bypassing it to access mutable runtime internals.

Fleet/client code should operate through supported connection/session projection surfaces.

Do not reintroduce direct raw `AgentSession`, `SessionManager`, extension-runtime, or equivalent mutable runtime access across the Fleet boundary when a supported connection abstraction exists.

Session replacement behavior should use supported replacement hooks/options rather than leaking runtime ownership across the client boundary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qredence/fleet-prime-agent](https://github.com/Qredence/fleet-prime-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
