---
trigger: always_on
description: Guidance for Claude Code when working in this repository. This is the
---

# CLAUDE.md

Guidance for Claude Code when working in this repository. This is the
monorepo-level file. Subtrees have their own `CLAUDE.md`, and the subtree file
wins for code inside that subtree.

## Repository layout

Three services, plus documentation and deployment config:

- **`app/`** the product frontend: a Next.js 16 / React 19 / TypeScript
  app (App Router, shadcn/ui, Tailwind v4, TanStack Query, Zustand, pnpm). It
  reaches every backend through same-origin `/api/*` proxy routes of its own.
  See `app/CLAUDE.md`.
- **`api/`** a FastAPI sidecar (uv, Python 3.11) owning what the node data
  model does not: KPIs, reports, the data catalog, favorites, tags, feeds and
  the AI provider. It stores no users; identity comes from node.
- **`node/`** the query backend, now a headless API (Flask + SQLAlchemy
  backend, Python via Poetry). Its React client and `viz-lib` visualization
  package were deleted in an earlier phase; the product UI is `app/`. It keeps
  its own conventions and tooling. See `node/CLAUDE.md`.
- **`docs/`** the Docusaurus documentation site, plus engineering notes.
- **`helm/`** generic Helm charts, each with an example values file beside
  it. The per-environment values and the provisioning scripts are deployment
  specific: they exist only in the deploy repository and are absent from the
  community one.
- **`ci/`** test and build jobs; `.gitlab-ci.yml` drives CI. The deploy jobs
  and the values they need are in the deploy repository only.

`node/` is checked in directly (not a git submodule) and is maintained here
like the rest of the tree. Follow its own `CLAUDE.md` and code style
(Black 119, ruff, Prettier) when working in it. The starter patterns below
apply to `app/`, not to `node/`.

## How the services connect

`app` never calls node, the sidecar, or ClickHouse from the browser. All
traffic goes through its own Next.js route handlers under
`app/src/app/api/*`, which read backend URLs and keys from server-side
env vars and forward the request. Auth uses the user's own node session
cookie so node enforces per-user permissions; a separate internal API key is
used only by the admin proxy routes after a session admin check.

## How a deployment is composed and released

A deployment can be the community tree plus one or more **packs**: separate
repositories, each holding a distribution that extends the product without
forking it. In the deploy repository, `docs/deploying.md` is the concrete
runbook (it names the project, namespace and hosts, so it does not travel to
the community repository). What follows is the part that governs how you make
changes, wherever you are working.

**There are two repositories, and the split is by repository, not by branch.**
The deploy repository holds the per-environment values, the deploy jobs and the
provisioning scripts. The community repository (`veodyn-ce`) is the published
tree, has none of those, and **is the one every build actually clones**.
`scripts/public_tree_forbidden_paths.py` lists what must never appear in the
community tree, and `scripts/check-public-tree.py` enforces it there.

The practical consequence, and the one that has already cost a redundant fix:
**a change made in only one of the two is stranded.** The community repository
is not generated from the deploy repository on any schedule, so the same edit
has to land in both. An earlier `oss` branch in the deploy repository used to
play the community role and no longer feeds anything; treat any instruction
that still points at it as stale.

**Releases are tag-gated and automatic.** Every job on a release tag runs
without a manual gate and ships to production. Never push a tag to try
something out. Stage runs the same pipeline with one variable changed, and its
jobs are manual buttons, so stage is where a change gets proven.

**Nothing builds from this checkout.** An assemble step clones the community
tree and each pack, runs the overlays, and hands the composed directory to every
build job as its Docker context. The community tree is cloned at a commit pinned
by each pack's manifest, so core changes reach a deployment when a pack is
re-cut, not when the core moves. Both packs carry the pin and both enforce it.

### The frontend composes at source; the Python services compose as layers

The bundler has to see pack code before `pnpm build`, so a frontend overlay is a
source-tree operation. The two Python services install pack distributions as
image layers and import them at runtime. When you change something, ask which
mechanism it rides: a layer can be added without rebuilding underneath it, a
frontend change cannot.

### Installing a layer does not register it

This has cost four separate releases and it is the first thing to check when a
feature is "deployed" but absent. **An installed layer is inert until a
deployment names it, and the deploy succeeds either way.** node imports
exactly the query runners and destinations its two variables list; the sidecar
imports exactly the modules its extra-modules variable lists; a worker runs
whatever command it is given; the enterprise schema advances only if its
migrations are switched on. Each is a values-file line, not a code change.

Never conclude a feature shipped from a green job, a correct values file, or a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veodyn/veodyn-ce](https://github.com/veodyn/veodyn-ce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
