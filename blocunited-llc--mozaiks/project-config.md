---
trigger: always_on
description: Repository-level guidance for coding agents working in this repo.
---

# AGENTS.md

Repository-level guidance for coding agents working in this repo.

Read [ARCHITECTURE.md](ARCHITECTURE.md) and [CLAUDE.md](CLAUDE.md) first.

This repo uses layered FastAPI hosts as the canonical server composition:

- `mozaiksai.hosts.runtime`
- `mozaiksai.hosts.platform`
- `mozaiksai.hosts.studio`
- `mozaiksai.hosts.mozaiks`

`mozaiksai.hosts.runtime` is the execution substrate. `mozaiksai.hosts.platform`
is the headless app host. `mozaiksai.hosts.studio` is the Studio management
interface host — the shared management layer for both local and hosted
deployments. `mozaiksai.hosts.mozaiks` is the hosted Mozaiks product host —
extends Studio, does not replace it.

Start via the CLI:

```
mozaiks serve ./my-app                  # platform host (no factory dependency)
mozaiks serve ./my-app --host studio    # Studio management host (requires factory_app)
```

Or directly via uvicorn:

```
uvicorn mozaiksai.hosts.studio:app --reload
```

CLI and Studio are **parallel interfaces** over shared system capabilities, not a
superset chain. Studio is not the CLI's UI. CLI owns developer tooling (filesystem,
scaffolding, process management). Studio owns the management interface (workspace
status, build lifecycle, artifacts, run history, config).

The current repo layout is transitional. The canonical target is documented in
[docs/architecture/foundations/distribution-and-workspace-model.md](docs/architecture/foundations/distribution-and-workspace-model.md).
Do not reintroduce a hybrid root that mixes the starter app bundle with shared
factory workflows.

## Repo Status

This codebase is **not in production**.

That means optimization goals are different from a legacy enterprise codebase:

- Prefer the cleanest canonical implementation.
- Prefer replacement over preservation.
- Remove stale logic when a better contract or architecture is introduced.
- Do not keep compatibility shims, aliases, wrappers, fallback branches, or duplicate schemas unless explicitly requested.

## Replacement Policy

When adjusting behavior:

- Replace outdated logic instead of layering new logic on top of it.
- Delete obsolete prompt guidance, docs, tests, config fields, and dead code paths that no longer match the current contract.
- Do not leave "temporary" legacy branches behind.
- Do not preserve old shapes "just in case" unless the user explicitly asks for backward compatibility.

If a new contract is introduced, update all affected layers together:

- runtime behavior
- generator prompts/hooks
- declarative schemas
- validation
- docs
- tests

## Clean Code Standard

Avoid "AI slop":

- no speculative abstractions
- no duplicate helpers with overlapping purpose
- no verbose compatibility code for non-production paths
- no stale comments describing removed behavior
- no split source of truth when one canonical source will do

Prefer:

- tight contracts
- explicit validation
- small, named abstractions with clear ownership
- removing drift at the source

## Canonical Repo Boundary

Canonical ownership:

| Layer | Owns |
|-------|------|
| `mozaiksai.hosts.runtime` / `mozaiksai` | AI execution substrate, sessions, transport, persistence, workflow execution |
| `mozaiksai.hosts.platform` | Headless app host: modules, pages, shell config, admin, actions, routing |
| `mozaiksai.hosts.studio` | Studio management interface host — shared management layer (local and hosted) |
| `mozaiksai.hosts.mozaiks` | Hosted Mozaiks product host — extends Studio with hosted-only capabilities |
| `mozaiksai.hosts.bootstrap` | Repo-local path defaults (CWD-relative; no-ops when not in repo checkout) |
| `mozaiks_cli/` | CLI / developer interface — parallel to Studio, not a subset of it |
| `factory_app/app/` | First-party Studio app bundle — shared control-plane routes, default brand, and factory app contract |
| `factory_app/app/ui/pages/custom/studio/` | Studio UI components — management interface layer |
| `factory_app/app/modules/factory_control_plane/` | First-party Studio control-plane module |
| `chat-ui/src/admin/` | Platform-management surfaces — registered by Studio, inherited by Mozaiks App |
| `platform/` | Repo-local infrastructure assets only — not an app workspace |
| `generated/` | Generator output awaiting validation/promotion |

Canonical target:

- generated/customer apps become standalone workspaces/repositories
- shared generation core lives outside any individual app workspace
- app workspaces are self-contained and keep `config/`, `ui/pages/`, `workflows/`,
  `modules/`, `ui/`, and `brand/` together under the active app root
- hosted product workspaces should consume that same contract from their own repos

## Module Contract Rule

When working in or generating modules:

- Every capability pack that needs deterministic app behavior must emit the
  canonical module contract files: `module.yaml`, `events.yaml`,
  `subscriptions.yaml`, `notifications.yaml`, `settings.yaml`, `admin.yaml`,
  and `backend/handler.py`.
- YAML declares contracts, capabilities, events, settings, notification rules,
  subscriptions, and admin panels.
- Python stubs implement behavior and hooks: `backend/handler.py` is required
  (thin dispatch — one method per declared action, no business logic, no ctx.db,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlocUnited-LLC/mozaiks](https://github.com/BlocUnited-LLC/mozaiks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
