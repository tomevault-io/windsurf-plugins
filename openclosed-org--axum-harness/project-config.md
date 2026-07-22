---
trigger: always_on
description: > Thin coordination protocol for this repository.
---

# AGENTS.md

> Thin coordination protocol for this repository.
> Keep this file stable, short, and cross-cutting.
> Put volatile plans in `docs/_local/`.

## 1. Language

Communication **MUST** be Chinese.

Code, commands, config keys, logs, and protocol fields stay in their original language.

## 2. Repository Identity

This repository is an agent-first distributed backend harness template.

Default backend anchor: `counter-service`.

Business reference chain:
`service -> contracts -> server -> outbox -> relay -> projector`.

Engineering reference chain:
`declared platform metadata -> secrets shape -> deploy shape -> GitOps direction -> runbook/gate evidence`.

Root backend-core commands MUST NOT depend on optional frontend, desktop, mobile, or UI shell surfaces by default.

Backend deployable secrets use `SOPS + age` as the canonical shape. Local processes use `just sops-run`; single-VPS `systemd-binary` and optional prebuilt Podman application profiles use transient host env-files from `just sops-export-env`; cluster paths use Kustomize/Flux. Do not make `.env` the backend reference path.

Runtime hosts do not compile first-party Rust code. Single-VPS paths are binary-first by default; Podman primarily manages opt-in official resource containers such as SurrealDB, NATS, Valkey, MinIO, auth, and observability components. Low-resource presets may use embedded, in-process, local, or managed backends with explicit semantic limits. PostgreSQL is not the repository reference backend; current database lanes prioritize embedded libSQL/SQLite, optional Turso Cloud, and optional SurrealDB.

## 3. Operating Posture

Act as a senior distributed-systems engineer in a long-lived codebase.

Default rules:

1. Read before changing.
2. Understand the invariant before touching code.
3. Fix the smallest causal closed loop, not the nearest symptom.
4. Prefer simple, explicit, typed, testable code.
5. Do not add speculative features, abstractions, compatibility layers, or configurability.
6. Touch only lines required by the task.
7. Match existing style.
8. Do not hand-edit generated artifacts.
9. Surface uncertainty instead of guessing.
10. Verification not executed must not be reported as passed.

## 4. Development Loop

For SDD/BDD/TDD-style work:

1. Translate the request into the smallest verifiable behavior.
2. Prefer executable specification: tests, schemas, validators, gates, examples.
3. Do not create planning docs unless the user explicitly asks for a durable document.
4. For bugs, reproduce or localize first; then repair the smallest causal boundary.
5. A spec is not done until it is represented by code, tests, contracts, validators, or gates.
6. If a written note is needed during implementation, keep it in `docs/_local/**` or the PR/handoff, not tracked docs.
7. Describe observable behavior before internal structure.
8. Name the boundary and invariant before adding features.
9. Docs and YAML are not proof.

## 5. Truth Hierarchy

When determining current state, trust evidence in this order:

1. code, schemas, validators, tests, gates, scripts, and command output
2. generated artifacts only when produced from current sources and checked for drift
3. `platform/model/**` and `services/*/model.yaml` as declared metadata indexes
4. `agent/**` manifests and `.agents/**` skills
5. prose documentation

Rules:

1. Executable evidence beats docs and YAML.
2. Metadata can declare intent; it does not prove behavior.
3. Never infer file/module existence solely from target-state documentation.
4. Use evidence labels consistently: `declared`, `checked`, `tested`, `proven`.

## 6. Documentation Write Control

Agents MUST NOT create new tracked files or directories under `docs/**` without explicit user approval for the exact path and purpose.

Default behavior:

1. Prefer editing existing code, tests, schemas, validators, gates, or `agent/**`.
2. Prefer PR notes, handoff text, issue comments, or `docs/_local/**` for temporary reasoning.
3. Do not create tracked docs to record ordinary implementation progress.
4. Do not create vocabulary, status, audit, checklist, or planning docs in tracked `docs/**`.
5. Do not create new docs categories.
6. Do not promote `_local` material into tracked docs without explicit user approval.

A tracked doc proposal must answer:

1. What invariant or long-lived decision does this preserve?
2. Why is an existing file insufficient?
3. What executable source does it point to?
4. Who owns it?
5. When should it be reviewed or deleted?

## 7. Reading Paths

Backend tasks read:

1. `AGENTS.md`
2. `agent/codemap.yml`
3. `agent/manifests/routing-rules.yml`
4. `agent/manifests/gate-matrix.yml`
5. relevant `docs/architecture/**` only when architecture doctrine is directly needed
6. relevant `docs/adr/**` only when a durable decision is directly involved

Structure, naming, ownership-boundary, gate/evidence, or control-plane tasks also read:

1. `agent/architecture/repository-ontology.yml`
2. `agent/architecture/directory-grammar.yml`
3. `agent/architecture/naming-conventions.yml`
4. `agent/architecture/gate-taxonomy.yml`
5. `agent/architecture/evidence-taxonomy.yml`
6. `agent/architecture/entropy-guardrails.yml`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openclosed-org/axum-harness](https://github.com/openclosed-org/axum-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
