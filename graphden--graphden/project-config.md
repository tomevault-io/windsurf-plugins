---
trigger: always_on
description: Packages & Dependencies (install / publish / registry) model, placement, lifecycle, and safety invariants for the Graphden editor. Apply when touching registry/, app/editor package/workspace/surface UI, or the package-version/package-install schema.
---


# Packages & Dependencies — spec

This rule captures the ONE conceptual model that package install/publish/registry
must obey, so the feature lands coherently (not as uncoordinated fragments).
It is grounded in cross-cutting editor principles that ALL editor work shares.

## 0. Governing principles (cross-cutting — never violate)

- **P1 Placement follows intent.** Three surfaces, three intents:
  - **Build** = author *your* project (Explorer tree, graph canvas, and the
    context-bar chips: workspace scope, branch, packages). Anything you do
    *while building* lives here.
  - **Organization** = governance / administration of the org.
  - **Platform** = cross-org administration.
  An affordance lives where its intent lives. Corollary for packages:
  *install* is a build act (add a building block) → Build; *publish* is an
  authoring act on what you built → an action on the thing; *govern* (catalog,
  who-may-publish, audit) → Organization.
- **P2 Personal overlay vs shared graph.** Per-user *view* state is per-browser
  `localStorage` (workspace scope + hidden set, lens, branch selection). Shared
  truth is the graph/DB. A personal choice must never mutate the shared graph or
  another user's view. Tenant data must never leak across orgs.
- **P3 Graph-native, minimal entities.** Reuse what exists: inheritance
  (`parent-ids` + binding overrides) as "change a bit"; immutable
  content-addressed `:package-version`; per-branch fn-versioning. Per-function
  PROPERTIES follow `.cursor/rules/function-metadata-and-identity.mdc`:
  graph by default (inherit a marker / a binding / a type), a DB column only for
  identity-dedup / org-RLS / VCS plumbing, NEVER a name or prefix. Add a field
  or entity ONLY when that rule says the graph can't serve it, and justify it in
  the diff. (`:package-version.org_id` in §5 is such a justified column: RLS.)
- **P4 No hardcoding of names or name-parts in code.** Dispatch, identity,
  classification, and cache keys use ids, or a real field (e.g. `role`), or
  graph structure — never a literal fn/package name, and never a name PREFIX
  (`_`, `_anon-`, etc.). This is worse than schema bloat. (Cosmetic display of a
  naming *convention* is tolerated only where NO behavior/identity depends on
  it.)
- **P5 Safe by default.** Privileged acts are capability-gated; tenant data is
  org-scoped + RLS-isolated; no request-outliving cache is un-org/principal
  keyed; the user sees what they're getting (effects, contents) before install.

## 1. The two version axes (must stay distinct)

- **Package version** = an immutable, content-addressed RELEASE TAG in the
  registry (`:package-version`, `name@version`, content hash; republishing the
  same `(name,version)` is rejected).
- **Internal versioning** = our per-branch fn-version rows + merge. This is the
  live graph's VCS.
- **Relationship:** *publish* snapshots the current (branch-resolved) state of a
  namespace subtree into an immutable bundle. *install* MATERIALIZES that bundle
  into the graph as ordinary fns (which then live under branch-versioning and
  propagate on merge) + writes a **pin** `(branch, package-name) → version`.
  The package version is provenance metadata layered over branch-versioning;
  the two never conflict. Install is branch-scoped (stage on dev → merge to prod).

## 2. Install — a Build act

- Entry point: a **Build-surface context-bar chip "packages"** (sibling of the
  workspace/branch chips; hidden off Build; hidden entirely when the optional
  `registry` package is absent — probe `window.API`, never a name). It opens a
  **browser**, NOT an Organization panel.
- The browser provides: **search**; a **per-package detail** view showing its
  **versions**, its **public interface** (the package root namespace's fns —
  see §6), and the **effects it requests**; a **version selector** (install ANY
  version, including older — rollback is the same symmetric operation);
  and an **"update available"** affordance when a newer version exists.
- Installs are **pinned** — never auto-updated. Updating/rolling back is an
  explicit act that repoints the pin, re-materializes, and rewrites the
  project's own refs old→new (package-internal refs never mix across versions).
- Install writes into the current branch (staging); it propagates on merge.

## 3. Publish — an authoring act on your work

- Entry point: a **"Publish" action on a namespace** (the project/subtree you
  built) — NOT a form buried in a shared panel, and NOT on the Organization
  page. It reuses the namespace = project unit (same unit Workspaces scopes).
- **Gated by a `publish-packages` capability** (tenancy grant vocabulary). An
  un-capable member cannot publish. (Install needs no capability beyond auth,
  unless a deployment chooses to restrict it.)
- Publish **freezes the transitive dependency versions** into the published
  bundle (a baked lockfile) → installs are reproducible.
- Publish targets the org's **private registry by default**; publishing publicly
  is an explicit, separate opt-in.

## 4. Governance — on the Organization surface


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Graphden/graphden](https://github.com/Graphden/graphden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
