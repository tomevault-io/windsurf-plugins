---
trigger: always_on
description: An **AI-native, skill-driven** back-office substrate (a "中台" template). The repo
---

# CLAUDE.md

## What this is — read first

An **AI-native, skill-driven** back-office substrate (a "中台" template). The repo
is **not a product**. It is the **source of truth for the skill catalogue**
(`.claude/skills/`): one **`add-component`** skill — a catalogue + retriever over
**35+ copy-ready admin UI shapes** (CRUD, detail, master-detail, kanban, calendar,
wizard, billing, RBAC, i18n, …), each a reference doc + a generated template — plus
a handful of **operation skills** (`scaffold-dashboard`, `add-backend`,
`rebrand`). A **Skills
Gallery** renders every shape's own demo. The demos, the
resources, and the two business cases all exist for one reason: **to back a shape
and be the live proof it produces working UI.** A shape's distributed `templates/`
are *generated from* this repo's working source and kept byte-for-byte in sync, so
the catalogue never ships code the repo hasn't typechecked, built, and tested.

**The stance is frontend-opinionated, backend-agnostic — preserve it.** The frontend
is one fixed, curated stack you *compose* from (TanStack Start + the shape catalogue +
the UI / form / table / chart system on `@base-ui`) — be opinionated here. The backend
(business **data** and **auth**) is *swappable* behind two seams — `Repository` and
`AuthProvider` — never the frontend's concern: pick one of the `add-backend` presets
(Postgres + Drizzle + better-auth by default; or Hono / FastAPI / Supabase, with
Prisma / Auth.js / custom-JWT). The enforcing rule is mechanical: backend specifics
(`@/db`, `@/lib/auth`, `drizzle-orm`, `pg`, an SDK client) live **only** in server-only
modules (a resource's `server.ts`, `src/db/*`, `src/lib/auth*.ts`, the `infra/data`
adapters) and reach the client solely through `createServerFn` + the two seams. Don't
let a backend leak upward, and don't fork the frontend stack per project.

**Two modes of work — know which you're in:**

- **Authoring the substrate — the default here.** You're adding or fixing a
  *skill*, the platform layer, a gallery demo, or a business case *inside this
  repo*. Whatever you produce must (a) be real, working repo code — so it's
  verified and visible in the gallery — and (b) stay in lockstep with its
  distributed skill via `sync-skills`. **If you are editing files in this repo,
  you are almost certainly in this mode.** → see *Authoring a skill* below.
- **Porting out.** A *different* agent stands up a real product by scaffolding the
  clean base into a **new** project and *composing* it (scaffold → rebrand → pick a
  backend → add resources → add shapes). The base is demo-free + gallery-free, so
  there's nothing to strip. That workflow lives in `PORTING.md` and the
  `scaffold-dashboard` / `rebrand` / `add-backend`
  skills — not here. Don't confuse "build a product" (port-out) with "maintain the substrate"
  (the work in this repo).

## The skill model — the most important convention

- **The repo source is the single source of truth.** A skill's bundled
  `templates/*` is a **generated copy** of a repo file, produced by
  `scripts/sync-skills.ts`. The UI shapes all live in **one** skill —
  `add-component` — whose templates are the flattened union of
  `COMPONENT_SOURCES` (a `Record<componentName, repoSourcePath[]>` in
  `sync-skills.ts`); each shape also has a hand-authored
  `add-component/references/<name>.md` (the "Add it / Foundation / Invariants /
  Verify" prose — **not** generated). Templates are a flat folder, basename only
  (a basename-collision guard fails the sync if two sources clash). Whole-project
  templates (the standalone backend presets) come from a second map in
  `sync-skills.ts`, `TREE_MANIFEST`, which copies a source *directory tree*
  (`backends/<preset>/`) **recursively** into `templates/<preset>/` — same drift
  guard, structure preserved, install/build junk skipped.
- **NEVER hand-edit anything under `templates/`.** Edit the repo source, then run
  `bun run sync-skills` to regenerate. `bun run sync-skills --check` is the
  **drift guard** — byte-for-byte compare, exits non-zero on any drift or missing
  source. Run it before finishing.
- **The gallery demo *is* the skill's test.** A skill's source is a real,
  self-contained, zero-config route/component the repo typechecks / builds /
  tests / renders in the Skills Gallery — so "does this skill produce working UI"
  is continuously proven. A skill whose demo doesn't render is not done.
- **`scripts/build-base.ts`** assembles the clean base bundle the
  `scaffold-dashboard` skill ships (the platform shell, with demo / scenario /
  gallery code stripped and the `clean/` overrides applied). Re-run
  `bun run build-base` after changing the platform layer or the `clean/` files.

## Authoring a shape (a component in `add-component`)

Adding a new UI shape means adding a **component** to the `add-component`
catalogue — not a new top-level skill. To add one:

1. **Build the demo in the repo**, self-contained and zero-config (local/static
   data, no Drizzle table): a route under `src/routes/_app/gallery/<demo>.tsx`,
   plus any small component under `src/components/…`, `src/infra/…`, or
   `src/lib/…`. Make it real, working code — that is both the proof and the test.
2. **Surface it in the Skills Gallery:** add a `SHAPES` entry in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahpxex/open-dashboard](https://github.com/ahpxex/open-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
