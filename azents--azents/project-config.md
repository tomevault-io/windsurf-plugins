---
trigger: always_on
description: Azents is an AI agent platform. This repository contains the backend, runtime providers, web frontends, test substrate, documentation, and Helm chart.
---

# Project Rules for Azents

Azents is an AI agent platform. This repository contains the backend, runtime providers, web frontends, test substrate, documentation, and Helm chart.

## Language Rule

- Write all git-tracked artifacts in English.
- Intentional non-English text is allowed only when the artifact itself is localized text or a locale sample.
- Record project decisions, design notes, comments, documentation, commit-facing text, and code-facing text in English even when chat discussion happened in another language.

## Repository Structure

| Path | Purpose |
| --- | --- |
| `python/apps/azents/` | Backend API server, worker, scheduler, CLI, DB schemas |
| `python/apps/azents-runtime-runner/` | Runtime runner image and entrypoint |
| `python/apps/azents-runtime-provider-docker/` | Docker runtime provider |
| `python/apps/azents-runtime-provider-kubernetes/` | Kubernetes runtime provider |
| `python/libs/az-common/` | Shared Python utilities |
| `python/libs/azents-runtime-control/` | Runtime control protocol/client |
| `python/libs/azents-public-client/` | Generated Python public API client |
| `python/libs/azents-admin-client/` | Generated Python admin API client |
| `typescript/apps/azents-web/` | Main web app |
| `typescript/apps/azents-admin-web/` | Admin web app |
| `typescript/packages/azents-public-client/` | Generated TypeScript public API client |
| `typescript/packages/azents-admin-client/` | Generated TypeScript admin API client |
| `testenv/azents/` | Fixture/prerequisite support and E2E tests |
| `docs/azents/` | ADR, design, issues, notes, plans, and living specs |
| `infra/charts/azents/` | Helm chart |
| `proto/azents/` | Runtime control protobuf definitions |

## Conventions System

Detailed coding rules live under `.claude/conventions/` and are indexed by `.claude/rules/`.

- Always start with `.claude/rules/conventions.md`.
- For Python work, also read `.claude/rules/python-conventions.md` when relevant.
- For TypeScript work, also read `.claude/rules/typescript-conventions.md` when relevant.
- For test substrate work, read `.claude/rules/testenv-conventions.md`.
- For infrastructure/Helm work, read `.claude/rules/infra-conventions.md`.
- Do not bulk-read every convention body. Read only bodies whose title applies to the change.

## Editing Strategy

### Replace Incorrect Units Instead of Patching Them

When an existing unit's responsibilities, control flow, or state model no longer match
the intended behavior, replace that function, class, module, or similarly narrow
surface instead of preserving the wrong structure with flags, branches, adapters, or
incremental special cases.

- Prefer a complete replacement when the expected behavior and acceptance checks are
  clear enough to verify.
- Preserve surrounding public contracts, call sites, migrations, and compatibility
  boundaries unless the task explicitly changes them.
- Do not minimize a diff by modifying a misunderstood implementation whose shape is
  itself the defect.

## Development Environment

```console
$ docker compose -f docker-compose.azents.yaml up -d
```

Backend:

```console
$ cd python/apps/azents
$ uv run python -m azents
```

Web:

```console
$ cd typescript
$ pnpm install
$ pnpm run dev --filter=@azents/web
```

## Python Commands

Run commands from the relevant Python subproject directory.

```console
$ uv run ruff check --fix .
$ uv run ruff format .
$ uv run pytest
```

Run the configured type checker for the subproject:

```console
$ uv run ty check --error-on-warning
```

Azents-specific backend commands:

```console
$ cd python/apps/azents
$ uv run python src/cli/dump_openapi.py
$ uv run python -m azents
```

## TypeScript Commands

Run from `typescript/`.

```console
$ pnpm install
$ pnpm run format
$ pnpm run lint
$ pnpm run typecheck
$ pnpm run build
$ pnpm run dev --filter=@azents/web
$ pnpm run dev --filter=@azents/admin-web
```

## Azents Documentation System

The azents project uses the Living Spec system:

- **Requirements** (`docs/azents/requirements/`) — confirmed user or system outcomes and acceptance criteria for one development snapshot. Implemented Requirements are immutable.
- **SPEC.md** (`docs/azents/spec/`) — current system behavior. Update when code changes.
- **ADR** (`docs/azents/adr/`) — append-only decision history. Implemented/adopted ADRs are immutable.
- **Design** (`docs/azents/design/`) — development-time design documents. Do not rewrite implemented design documents as living specs; current behavior belongs in spec docs.

The [branding-260823 historical document migration](docs/azents/adr/branding-260823-historical-document-migration.md)
authorizes one completed brand-only rewrite of historical snapshots and canonical
document identifiers. This exception does not make implemented snapshots generally
mutable; the normal immutability rules apply after that migration.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [azents/azents](https://github.com/azents/azents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
