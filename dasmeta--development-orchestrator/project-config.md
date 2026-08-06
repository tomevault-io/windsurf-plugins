---
trigger: always_on
description: This repository is a reusable development orchestration control plane. It is
---

# Development Orchestration Template

## Purpose
This repository is a reusable development orchestration control plane. It is
intended to be copied or forked for a specific product workspace, then wired to
that product's child repositories through `config/workspace.json`.

The root repository owns delivery wiring. It does not own product behavior,
shared interfaces, runtime implementation, or executable validation details.

## Fork Overlay
If `WORKSPACE.md` exists in the repository root, read it after this file.
`WORKSPACE.md` is fork-owned and may define product-specific child repositories,
routing rules, validation gates, and workspace conventions. Generic upstream
rules still apply unless the fork overlay narrows them for the product
workspace.

## Repository Model
Each product fork defines its child repositories in `config/workspace.json`.
The root instructions define:

- how feature work enters the orchestration flow
- how work is routed between configured child repository roles
- what prerequisites must exist before downstream stages start
- how generated feature state and mergeability evidence are interpreted

Repository-local artifact formats, approval flows, implementation details, and
domain rules belong in the configured child repositories.

## Configured Ownership
The template ships with these default roles:

- `intent`: product intent, behavior, acceptance criteria, edge cases, and
  logical test intent
- `contracts`: shared interfaces, request/response shapes, schemas, and
  compatibility rules
- `implementation`: runtime implementation for approved intent and contracts
- `validation`: executable validation grounded in approved intent and contracts

Product forks may rename or replace these repositories, but they must preserve
clear ownership boundaries in `config/workspace.json` and child repo
instructions.

## Root Execution Principle
Treat this workspace as an orchestrated multi-repo system, not a single mixed
codebase.

Rules:

- Read across repositories when needed for context.
- Write only in the explicitly targeted repository.
- Do not silently edit multiple repositories in one pass unless the operator
  explicitly requests multi-repo execution.
- Do not invent product behavior in this root repository.
- Do not invent shared interfaces in this root repository.
- Do not invent executable validation scenarios that are not grounded in the
  configured intent source, unless the operator explicitly accepts exploratory
  validation.
- Do not treat implementation code as the source of truth for behavior or
  contracts.

## Request Routing
Route work by the kind of change being requested and the active
`config/workspace.json` mapping:

- Behavior definition, business rules, acceptance criteria, scenarios:
  configured intent repository
- Shared API/schema/interface changes: configured contracts repository
- Runtime implementation: configured implementation repository
- Executable validation: configured validation repository
- Orchestration scripts, feature state, templates, and routing rules: this root
  repository

If a request spans multiple repositories, determine the correct sequence first
and then work one repository at a time.

## Dependency Order
Default upstream-to-downstream flow:

1. Intent
2. Contracts when shared interfaces are affected
3. Implementation
4. Validation

The actual stage IDs and repositories come from `config/workspace.json`.

Gates:

- Do not implement behavior if the required product intent is missing or
  unresolved in the configured intent repository.
- Do not change shared request/response shapes, schemas, or interface contracts
  only inside implementation repositories.
- Do not build executable validation for behavior that lacks approved intent or
  accepted exploratory scope.
- Do not claim end-to-end completion without relevant validation evidence,
  unless the operator explicitly accepts that gap.

## Handoff Rules
When work enters a child repository:

- follow that repository's local instructions and workflow
- keep changes bounded to that repository
- use upstream repositories as inputs, not places to infer missing truth

If upstream truth is missing:

- stop downstream implementation work
- route the work to the appropriate upstream repository instead of guessing

## Product-Specific Overlays
This template must stay product-agnostic. Product forks may add overlays for:

- product-specific child repository names
- domain object routing
- runtime AI or tool-writing guardrails
- UI builder handoff conventions
- deployment, preview, or validation gates

Those overlays belong in the product fork or child repositories, not in the
central template.

## Template Fork Contract
The full upstream/fork ownership contract is documented in
`docs/runbooks/template-fork-contract.md`.

Upstream-owned paths are generic template surfaces: `AGENTS.md`, `AI-INDEX.md`,
`README.md`, `TODO.md`, `.github/`, `.gitlab-ci.yml`,
`bitbucket-pipelines.yml`, `.specify/`, `.agents/`, `scripts/`, `schemas/`,
`templates/`, `tests/`, `docs/`, and
`config/workspace.example.json`.

Fork-owned reserved paths are `WORKSPACE.md`, `config/workspace.json`, root
`00-*` through `05-*` child repository folders, `workspace/`, and `overlays/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dasmeta/development-orchestrator](https://github.com/dasmeta/development-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
