---
trigger: always_on
description: Lootlog is an open-source Margonem companion that connects an in-game client,
---

# Lootlog

Lootlog is an open-source Margonem companion that connects an in-game client,
a shared web workspace, and Discord. It turns supported gameplay events into
current, durable information for players and organized groups without requiring
them to copy timers, loot records, or coordination state by hand.

The primary daily user is an active member of an organized Margonem group. A
leader, deputy, or tactician usually makes the adoption decision, but every
member must receive useful in-game feedback rather than acting only as a data
source.

## What Lootlog must protect

### 1. Trust the signal

Accepted durable records must not disappear silently, and retries must not
create unintended duplicates. Show degraded or stale state instead of
presenting it as current truth.

### 2. Keep Margonem fast

Normal play must not feel slower with Lootlog enabled. Keep game-client work
bounded; a client performance regression is a release blocker.

### 3. Isolate Organizations

The Organization is the top-level security boundary. Apply its access policy to
base records, derived views, delivery paths, and metadata.

### 4. Complete the connected workflow

A change is incomplete when it works on one relevant surface but fails on
another. Account for the Game client, Web app, Discord bot, public surfaces,
installation methods, deployed contracts, and generated clients as applicable.

Resolve architectural trade-offs in this order:

1. Do not interfere with Margonem or lose accepted durable data.
2. Preserve Game client performance.
3. Preserve security and Organization isolation.
4. Preserve real-time reliability.
5. Control infrastructure cost.
6. Preserve development speed and maintainability.
7. Preserve abstract future flexibility.

## Project principles

Prefer ambitious outcomes and simple systems. Understand the real constraint,
then implement the smallest complete model that makes correct behavior
unsurprising. Remove accidental complexity instead of preserving it, and avoid
machinery justified only by a possible future need.

Measure before adding performance work. Fix a defect at the shared root cause
after tracing every caller and affected boundary. Treat current code and
verification as evidence; roadmap documents and target architecture are not
proof that a feature already exists.

## GPT-6 Astra operating defaults

These defaults tune GPT-6 Astra and define the same work contract for other
capable agents:

- Infer the user's intent and authorized scope from their request and the prior
  conversation. Treat requests such as “help me,” “can you,” and “I want to” as
  instructions to do the work, not invitations to describe how it could be done.
- Persist until the authorized outcome is complete. Do not stop at a plan,
  partial fix, or capability statement when the requested work can be completed.
- Inspect the repository before asking questions. Ask only when missing input
  would materially change the result, new authority is required, or the next
  action would be destructive or irreversible.
- Complete reversible preparation before requesting approval for a consequential
  final action. Present a concrete, reviewable result rather than a hypothetical
  proposal.
- Explicit user instructions override repository defaults and skill guidelines.
  System and platform policy, authorization boundaries, and the user's ownership
  of data remain controlling. If the user explicitly changes a product,
  security, or compatibility contract, include its migration, documentation,
  and verification consequences in the work.
- Delegate independent, bounded work when parallel execution will materially
  improve speed or quality. The primary agent owns integration, conflict
  resolution, and final verification.
- Lead with the outcome. Use plain, direct language and concise paragraphs. Use
  lists only for parallel, sequential, or comparative information.
- If a skill causes the work to pause, remain unfinished, or change direction,
  name the exact `SKILL.md`, identify the controlling instruction, and explain
  briefly how it applies.

## Working in this repository

This is the repository's only `AGENTS.md` and applies to every workspace. Apply
repository material in this order after the operating defaults above:

1. this file;
2. repository skills in `.agents/skills`;
3. generic or external skills;
4. lint, tests, and CI as mechanical enforcement.

Preserve unrelated user changes in the working tree. Assume the application is
already running; do not start it. Keep temporary plans, research, and agent
scratch files outside tracked repository paths.

<!-- CODEGRAPH_START -->

### CodeGraph

This repository is indexed by CodeGraph through `.codegraph/`. Use it before
grep, find, or broad source reads when locating code or tracing behavior:

- MCP: use `codegraph_explore` for symbols, source, and call paths, or
  `codegraph_node` for one symbol or file.
- Shell: use `codegraph explore "<question>"` or
  `codegraph node <symbol-or-file>`.

<!-- CODEGRAPH_END -->

## Vendored repositories

When work depends on a library vendored under `repos/`, inspect that repository
before using web search or guessing from documentation. Use its implementation,
tests, and module structure as reference material.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lootlog/monorepo](https://github.com/lootlog/monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
