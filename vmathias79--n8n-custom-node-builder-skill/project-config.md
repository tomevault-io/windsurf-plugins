---
trigger: always_on
description: Plan, build, test, verify, and deploy n8n custom nodes and credentials for community packages or private installs. Use when creating new nodes, extending existing nodes, choosing declarative vs programmatic style, implementing credential auth, designing node UI parameters, enforcing n8n verification constraints, and preparing release or private deployment.
---


# n8n Custom Node Builder Skill

## Goal

Implement or extend n8n nodes end-to-end using a reference-first workflow that is stable offline.

## Reference-First Rule

Before each lifecycle step, load the matching local file from `references/`.

- Start every task with `references/README.md` and `references/01-overview.md`.
- Do not browse external docs unless local references are stale or missing a required detail.
- If any reference is stale (older than 45 days), refresh only the changed files and update timestamps.

## Build Lifecycle

```
[Plan Scope] ──→ [Choose Style] ──────────────────┐
                       │                          │
               [Declarative]              [Programmatic]
               routing-based              execute() method
               (references/04)            (references/05)
                       │                          │
                       └──────────┬───────────────┘
                                  ↓
                    [UI Parameters] → [Credentials] → [Codex]
                    (ref/07)          (ref/08)         (ref/09)
                                  ↓
                            [Test + Lint]
                            (ref/10)
                                  ↓
                  ┌───────────────┴──────────────────┐
                  ↓                                  ↓
       [Community Deploy]                   [Private Install]
       (ref/11)                             (ref/13)
                  ↓
       [Verification?]
         ↓         ↓
       [Yes]      [No]
     (ref/12)  (npm publish)
```

## Quick Reference

| Need to… | Load |
|----------|------|
| Overview and prerequisites | [`references/01-overview.md`](./references/01-overview.md) |
| Plan node scope, auth, and error handling | [`references/02-planning.md`](./references/02-planning.md) |
| Choose declarative vs programmatic style | [`references/03-approach-selection.md`](./references/03-approach-selection.md) |
| Build a declarative node | [`references/04-declarative.md`](./references/04-declarative.md) |
| Build a programmatic or trigger node | [`references/05-programmatic.md`](./references/05-programmatic.md) |
| Validate package file structure | [`references/06-node-file-structure.md`](./references/06-node-file-structure.md) |
| Design UI parameters and field types | [`references/07-ui-elements.md`](./references/07-ui-elements.md) |
| Implement credentials and auth injection | [`references/08-credentials.md`](./references/08-credentials.md) |
| Configure codex metadata | [`references/09-codex.md`](./references/09-codex.md) |
| Test and lint locally | [`references/10-testing.md`](./references/10-testing.md) |
| Publish to npm community | [`references/11-community-deploy.md`](./references/11-community-deploy.md) |
| Prepare for n8n verification | [`references/12-verification-guidelines.md`](./references/12-verification-guidelines.md) |
| Install privately (Docker/global) | [`references/13-private-install.md`](./references/13-private-install.md) |

## Start Here

**Building a new node from scratch?**
→ Start at step 0 (Kickoff), follow the full workflow 0 → 9.

**Adding or fixing credentials only?**
→ Jump to step 6. Load `references/08-credentials.md`.

**Extending an existing node?**
→ Start at step 2 (Choose Style), then step 3 (Build).

**Preparing for n8n community verification?**
→ Jump to step 9 (Community path). Load `references/12-verification-guidelines.md` and `references/11-community-deploy.md`.

**Installing a built node privately (Docker)?**
→ Jump to step 9 (Private path). Load `references/13-private-install.md`.

**Node not appearing in n8n / debugging?**
→ Jump to step 8 (Test). Load `references/10-testing.md`.

## Workflow Router

### 0) Kickoff and context

Load:

- `references/README.md`
- `references/01-overview.md`

Do:

- Confirm task scope (new node, extension, verification prep, or private install).
- Confirm intended deployment target (community vs private).

### 1) Plan node scope

Load:

- `references/02-planning.md`

Do:

- Define node type and target API resources/operations.
- Define auth model and required permissions/scopes.
- Define error handling and retry/timeouts.
- Define expected outputs and data mapping behavior.

### 2) Choose implementation style

Load:

- `references/03-approach-selection.md`

Do:

- Default to declarative style.
- Use programmatic style when trigger behavior, non-REST behavior, custom transforms, or full versioning requires it.
- Record style choice in implementation notes before coding.

### 3) Build the node (style-specific)

Load one:

- `references/04-declarative.md`
- `references/05-programmatic.md`

Do:

- Implement resource/operation structure.
- Keep operation lists and parameter naming consistent.
- Keep optional inputs behind collections.

### 4) Validate file structure

Load:

- `references/06-node-file-structure.md`

Do:

- Ensure required package, node, and credential files exist.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vMathias79/n8n-custom-node-builder-skill](https://github.com/vMathias79/n8n-custom-node-builder-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
