---
trigger: always_on
description: Angee is a thin composition framework for Django + React applications. It binds
---

# AGENTS.md

Angee is a thin composition framework for Django + React applications. It binds
boring, proven libraries into one deterministic product surface. Before adding,
replacing, or hand-rolling a capability, check the opinionated stack in
`docs/stack.md`; it is the single source of truth for which library owns what.
The dependency manifests lock the install shape: `pyproject.toml` + `uv.lock`
for Python, and the root `package.json` + `pnpm-workspace.yaml` +
`pnpm-lock.yaml` for the one TypeScript workspace.

The framework owns the seams, not the concerns. Product logic belongs in addons.
The composer turns addon contracts and project settings into a runnable project.
A project declares the root apps it composes through Django `INSTALLED_APPS`.

## Repository Role

This repository holds the Angee framework core: the language and the loom — the
data contract, composer, model toolkit, serving seams, and jobs seam. The core is
not an addon. Everything that gives a product a capability, including its API
protocol, is an addon; framework addons live under `addons/`, and consumer
addons are a product team's own code. See `docs/glossary.md` for these terms.

The first question for any change is *what level does it belong to?*

- **Framework core** — the composition language and shared machinery inherited
  by every project downstream. A change here is copied into every consumer, so
  hold it to the highest bar.
- **Framework addon** — a reusable product capability that ships with Angee.
- **Consumer addon** — product logic for a specific project, built on the
  framework.

Put each change at the level that owns the concern. Never solve at the consumer
level what the framework should own, and never push product specifics down into
the framework. Keeping each fact at its owning level is what keeps the whole
stack DRY.

## Repository Layout

A map by role, not a file inventory — core module docstrings and addon contracts
own the current behavior, and this points to those owners. The framework core at
`angee/` is the one real Python package (`django-angee`); product capabilities
live beside it under `addons/`. The `angee.*` namespace spans the core and base
addons without changing any import.

```text
.                           # the consolidated framework source checkout
├── angee/                  # `django-angee` — framework core + composer (PEP 420 namespace, no __init__.py)
│   ├── base/               # framework core: the model toolkit (abstract models, mixins, fields, managers)
│   ├── data/               # framework core: the product data-surface description contract
│   ├── compose/            # the composer — emits the concrete runtime (`manage.py angee build`)
│   └── jobs/               # the Celery seam (broker wiring, beat, queue routing)
├── docs/                   # intent docs — glossary, stack, guidelines, and `docs/howto/`
├── addons/                 # standard `angee.*` folder addons + co-located web fragments
├── packages/               # `@angee/{app,ui,refine,metadata}` + Storybook/e2e tooling (guide: docs/frontend/e2e.md)
├── examples/               # showcase consumer addons + reference Playwright suite
├── templates/              # Copier project / stack / workspace / service templates
├── tests/                  # merged core, addon, and template contract tests
├── .agents/                # shared agent methodology — reviewer agents, commands, skills, workflows (`.agents/README.md`; public)
├── .work/                  # private agent work-state — plans, notes, handovers (gitignored symlink to a separate private repo; never mirrored)
├── README.md               # human entry point; `AGENTS.md` is the agent/contributor entry point
└── pyproject.toml, uv.lock, package.json, pnpm-lock.yaml
```

The opt-in personal-messaging bridges remain in the external
`angee-messaging-bridges` repository. The Go operator, `hatch-angee`, and
`strawberry-django-hasura` also remain independently published repositories.

You edit **source models** in addons; the composer emits the concrete apps and
the `runtime/` tree. Generated `runtime/` is output — change the source, not the
artifact (see `docs/glossary.md`).

## Area Rules

### Framework core (`angee/`)

The core owns composition language and shared machinery inherited by every
project. It must not absorb product capability or addon vocabulary. Run the
full Python suite before handoff because core changes can affect every composed
area.

### Framework addons (`addons/`)

Each standard addon is a source folder with an `addon.toml`, not a separately
distributed Python package. Keep its capability, manifest, resources,
permissions, and `web/` fragment together. Generated SDL and `@angee/gql`
documents belong to the composed host; never move them into source or core.

### Frontend packages (`packages/`)

Schema independence is an invariant for the published framework packages: they
must not import project-generated `@angee/gql/*` modules. Package layering is
`refine` and `metadata` → rented libraries, `ui` → `refine` + `metadata`, and
`app` → all three. `packages/tsconfig.base.json` and
`packages/vitest.shared.ts` own shared package configuration. Update

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ang-ee/angee-django](https://github.com/ang-ee/angee-django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
