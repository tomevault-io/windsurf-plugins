---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) in this repository.

## ⚠️ CRITICAL — LIFE OR DEATH: zero code comments

**This is the most important rule in this file. Violating it is treated as a failed task.**
Never write a comment that narrates, restates, or explains what the code already says — no
"fetch the user", no "call the service", no explaining your change to the reviewer. Make the
code self-explanatory instead: better names, smaller functions.

```ts
// BAD — restates the code. Never do this:
// fetch the user's repositories
const repos = await this.repositoryService.list(userId);

// GOOD — no comment; the code already says it:
const repos = await this.repositoryService.list(userId);
```

The only permitted comment captures something the code *cannot* express — a why, a constraint,
a gotcha a reader would otherwise miss. These are rare. When in doubt, do not comment.
**Before finishing any edit, re-read your diff and delete every comment that fails this bar.**

This rule is about code. Infra config (`tf/`, `ansible/`, `kubernetes/`) has no equivalent
remedy: the keys belong to Ceph, Terraform and Ansible, so there is no better name to pick and
no smaller function to extract. The bar is the same and restating a value is still banned, but
a comment recording why a value deviates from its default, or what breaks when it changes, is
usually the only place that knowledge can live.

## What this is

Yucca is a multi-tenant **backup service**: OIDC-authenticated users get S3-backed
[restic](https://restic.net/) repositories. Two things at once:

- **Application** (`packages/`) — NestJS/Go/Svelte services.
- **Infrastructure** (`tf/`, `ansible/`, `kubernetes/`, `charts/`) — Ceph object storage,
  Talos K8s, Flux GitOps, networking. See `README.md` and per-directory `README.md` files.

## Tooling

- Everything runs through **[mise](https://mise.jdx.dev/)** tasks (`.mise/tasks/` scripts +
  `.mise/config.toml` aggregates). mise pins every binary (node, pnpm, go, kubectl, helm, tilt,
  opentofu, ansible…) — do not assume a tool is on PATH outside of mise.
- **pnpm workspaces** with a **catalog** (`pnpm-workspace.yaml`): add/bump shared deps in the
  catalog, referenced as `"catalog:"` in each `package.json` — not in individual packages.
- Secrets come from **1Password** via `op run`; `.env` files contain `op://` refs, never
  literal secrets. `OP_ACCOUNT` is set in `.env` (copy `.env.example`).

## Common commands

```bash
mise dev                  # compose-based dev: deps, docker infra (postgres/minio/mock-oidc/victoria-*), all *:dev
mise <pkg>:dev            # one service, e.g. mise web:dev, mise yucca-api:dev

mise check                # lint + format check + svelte-check + unit tests (= the `checks` CI job)
mise fix                  # autofix lint/format + lingui extract
mise build                # build all packages

mise test                 # all unit tests (jest per NestJS pkg, vitest for web)
mise test:integration     # integration tests (--jobs 1; needs infra up)
mise test:integration:k3d # CI split: the database-backed suites; :s3 = the Ceph-backed ones
mise test:e2e             # e2e (needs the stack running); mise test:e2e:web = Playwright
mise <pkg>:test           # one package; args after -- go to jest: mise yucca-api:test -- -t "name"

mise yucca-api:migrations <args>   # DB migrations (@immich/sql-tools; yucca-api owns the schema)
```

### k3d + Tilt (prod-shaped dev)

`mise k3d:up` → `mise tilt:up` (build images, render charts from `kubernetes/apps/dev/local`,
port-forward, live_update); `tilt:down` / `k3d:down` to tear down. Tilt's source of truth is the
Flux tree under `kubernetes/` — see the extensively commented `Tiltfile`.

CI deploys resource subsets rather than the whole stack (`mise tilt:ci` still does everything):
`tilt:ci-infra` (integration — postgres, mock-oidc, victoria; **no Ceph**), then for e2e
`tilt:ci-ceph` (Rook only; builds no images, so it converges while the workspace installs)
followed by `tilt:ci-e2e` (the apps the e2e suites touch). Ceph converging outweighed every test
in the integration job, so that job dropped it and its S3-backed suites moved to the e2e one.

Ports: `5173` web · `3020` yucca-api · `3030` yucca-admin-api · `3010` michael ·
`8092` mock-oidc · `8025` mailpit · `8093` mock-postmark · `9000` ceph rgw · `8428` victoria-metrics · `9428` victoria-logs.

### Infrastructure commands

```bash
mise tf:plan / tf:init / tf:fmt    # terragrunt (TF_STACK_DIR=tf/deployment/<partition>/<region>/<stack>)
mise k8s:validate                  # helm template + kubeconform + flux-local of the k8s tree (no cluster)
mise mgmt:render-inventory / mgmt:ansible
```

**CI owns terraform applies** (`.github/workflows/infra.yml` on merge to main). Locally you may
`tf:plan` but never `tf:apply`. Gotcha: unset the shell's stray `AWS_CA_BUNDLE` before planning —
it breaks the OVH S3 state backend. We own the `futo-org/netbird` provider
(`../terraform-provider-netbird`, pin ≥ 1.0.2); renaming a NetBird setup key forces replacement
(the API can't rename keys), regenerating its value.

## Application architecture

Backend services are **NestJS 11 + TypeScript**: controllers → services → repositories,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [immich-app/yucca](https://github.com/immich-app/yucca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
