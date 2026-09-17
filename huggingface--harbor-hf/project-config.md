---
trigger: always_on
description: - This repository is public. Treat all operator-specific information as private.
---

# Repository instructions

## Public repository privacy

- This repository is public. Treat all operator-specific information as private.
- Do not publish personal names, usernames, account namespaces, email addresses,
  home paths, machine names, private repository names, private Space or Bucket
  names, Job IDs, token display names, credential aliases, or private topology.
- This rule covers source, documentation, examples, tests, fixtures, generated
  files, logs, commits, branches, issues, pull requests, comments, and releases.
  Platform-assigned authorship is the only automatic exception.
- Use placeholders such as `<namespace>`, `<control-space>`, `<artifact-bucket>`,
  and `<service-token>`.
- Public availability elsewhere does not grant permission to repeat a private or
  operator-specific identifier in this repository.
- Publishing an exact external identifier or destination requires explicit
  approval for that exact value and exact public destination.
- If private data is published, remove it from the current public surface,
  disclose what was exposed and where, and recommend rotation when a credential
  could be affected. Ask before rewriting public history.
- Before each public commit, push, issue, pull request, comment, or release, run
  `uv run python scripts/check_public_privacy.py .`. Inspect the complete diff
  and public metadata. Remove private values before publication.

## Harbor-first design

- You MUST NOT duplicate behavior, configuration, state, or data that Harbor
  already provides. This requirement is the first and controlling design rule
  for all work in this repository.
- You MUST read [the design principles](docs/DESIGN_PRINCIPLES.md) before you
  design or implement a behavior change.
- A feature request MUST NOT be treated as proof that Harbor lacks the feature.
- Before you add a field, record, loop, parser, adapter, or UI control, you MUST
  inspect the pinned Harbor source and relevant history. The pull request
  description MUST name the checked files and public APIs.
- When Harbor already has the behavior or field, you MUST use Harbor directly
  through its native API and configuration. You MUST treat Harbor output as
  authoritative. You MUST NOT add an alias, mirrored field, fallback reader,
  second state machine, or renamed wrapper for the same concept. For example,
  you MUST NOT add `environment_flavor` when Harbor already uses
  `environment.kwargs.flavor`.
- Harbor owns `JobConfig` and benchmark task resolution. Harbor owns trial
  execution, including concurrency and retry behavior. It controls resume and
  locking behavior. Harbor results are authoritative for rewards and reported
  costs. The same rule applies to trajectories and built-in agent output.
- Harbor-HF owns authenticated submission and reviewed restrictions. It also
  owns the control Space and Bucket as well as HF Job lifecycle and cost stops.
  The disposable SQLite projection and web console also belong in Harbor-HF.
  Harbor-HF owns the leaderboard.
- You MUST keep benchmark and model names as data. You MUST keep necessary
  harness-specific behavior in a Harbor agent plugin behind `import_path`.
- If you cannot prove that Harbor lacks required general behavior, you MUST stop
  local design work and report the evidence. You MUST get explicit user
  confirmation before you open a Harbor issue or pull request.
- A temporary local implementation MUST have separate approval for an exact
  upstream gap. It MUST name the Harbor revision that permits its removal.
- Before you finish a behavior change, you MUST compare each changed schema or
  persisted field with Harbor. You MUST make the same comparison for API and UI
  values. You MUST remove any duplicate or renamed Harbor concept.

## General benchmark contracts

- Do not add model-, provider-, agent-, or harness-specific admission rules, compatibility lists,
  settings, defaults, request rewrites, or protocol workarounds to Harbor-HF.
- Apply generic schema, authorization, budget, provenance, and lifecycle rules uniformly. A
  configuration that satisfies those rules can run and report its normal failure when its selected
  components are incompatible.
- Treat a failed combination as benchmark evidence. Do not convert past failures into a hard-coded
  allowlist, denylist, compatibility matrix, or special case.
- Put component-specific fixes and settings in the responsible provider, harness, agent, or upstream
  adapter. Keep reviewed presets declarative and do not use them to claim pair-specific
  compatibility.

## Storage and resources

- The steady-state inventory is one private control Space and one private
  Bucket. Do not add a repository, Space, Bucket, Dataset, endpoint, schedule,
  backup store, lease store, status store, or result service for a run.
- Store current data under `runs/<run-id>/`. The service writes `run.json` and
  `state.json`. Harbor alone writes below `job/`.
- SQLite is a disposable three-table projection. It must rebuild from Bucket
  data and HF Job observations.
- Use a hard cutover. Do not add compatibility readers, dual writes, old profile
  support, or a second API version.

## Credentials

- The control Space has two operator-managed secrets. `HF_TOKEN` is the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huggingface/harbor-hf](https://github.com/huggingface/harbor-hf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
