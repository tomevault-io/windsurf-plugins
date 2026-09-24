---
trigger: always_on
description: Use when adding Auth0 authentication to an Express.js app — login/logout
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this repo is

An **Agent Skill** that teaches coding assistants how to implement Auth0
authentication that follows Auth0's documented SDK usage and passes this
repo's routing and behavioral evals. It ships as a single Claude Code / Cursor / Copilot
plugin (`auth0`) containing **one** consolidated skill at
`plugins/auth0/skills/auth0/`: a router `SKILL.md` over a pool of on-demand
reference files, some of which are grouped into hub+leaf directories. The
deliverable is the skill itself.

## Repository layout

```
plugins/auth0/skills/auth0/
├── SKILL.md            # Required: the router (only file allowed in skill root)
├── references/         # On-demand docs, each reachable from SKILL.md. Every
│                       #   reference is a directory — framework-<name>/,
│                       #   feature-<name>/, tooling-<name>/, pattern-<name>/
│                       #   (kebab-case) — with an index.md that is either the
│                       #   whole reference (index-only) or a hub + document-
│                       #   section leaves (leaf group).
├── scripts/            # Optional: executable helpers
└── assets/             # Optional: static resources (templates, data)

evals/                  # Repo-root eval harness (NOT inside the skill dir):
├── routing-cases.json  #   deterministic routing cases (scripts/check_routing_evals.py)
├── activation/         #   activation evals — does the SKILL.md `description` fire on the
│                       #   right prompts and stay quiet on the wrong ones? The only layer
│                       #   that tests frontmatter; run it after any description edit.
└── behavioral/         #   behavioral evals — run-evals.mjs drives a live agent via the
                        #   claude CLI + execa. Kept out of the skill dir on purpose so
                        #   this dev-only harness isn't in per-skill security-scan scope.
```

Key top-level docs:

- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — **authoritative** rules for adding or
  editing Auth0 guidance: reference structure, router wiring, naming, and the
  validation gate. Read this before changing the skill. Frontmatter requirements
  are below.
- [`PLUGIN.md`](./PLUGIN.md) — plugin/marketplace architecture.
- [`README.md`](./README.md) — user-facing install and skill catalog.
- [`docs/architecture.md`](./docs/architecture.md) — why one skill, and how the
  router resolves a request to reference files.
- [`docs/openai-plugin.md`](./docs/openai-plugin.md) — OpenAI/Codex plugin
  packaging, pre-submission testing, and public submission.

## Before you change a skill

1. Read [`CONTRIBUTING.md`](./CONTRIBUTING.md). The conventions there are
   enforced and not optional.
2. Match the patterns of the existing reference files rather than inventing new
   structure. The naming convention (`framework-<name>`, `feature-<name>`,
   `tooling-<name>`, `pattern-<name>`) and the router-in-`SKILL.md` layout
   exist on purpose — keep them consistent. Navigation is a **depth-3 tree**:
   every reference must be reachable from `SKILL.md`; an index-only `index.md`
   or a group leaf must not link to any other `.md`; the only allowed second hop
   is a leaf-group hub `index.md` dispatching to leaves in its own directory.
   Enforced by `scripts/check_router_reachability.py`. See CONTRIBUTING.md →
   "Adding a reference" before splitting a large one (>1000 lines) into a leaf
   group.
3. For the step-by-step authoring procedure (which router tables to edit, in
   what order, and how to validate), use the `author-auth0-skill` skill in
   Claude Code. It walks through adding a framework, feature, tooling, or
   pattern reference so the change passes CI the first time.

## Required SKILL.md frontmatter

These fields are **enforced by the linter** — a skill missing any of them fails
CI:

- `name`, `description`
- `license` (use `Apache-2.0` to match the repository `LICENSE` unless a
  specific package requires otherwise)
- `metadata.author` in `Name <email>` format
- `metadata.openclaw.emoji` and `metadata.openclaw.homepage`

The `requires`, `os`, and `install` fields under `metadata.openclaw` are
[ClawHub](https://clawhub.ai) metadata used when a skill is installed via
`npx clawhub install`. If a skill's workflow invokes `auth0` CLI commands,
declare `requires.bins: [auth0]` (and the matching `install` block) so ClawHub
can prompt the user to install the CLI. The frontmatter of
`plugins/auth0/skills/auth0/SKILL.md` is a working example of all three fields.

## Validating your changes

This repo uses [skillsaw](https://github.com/stbenjam/skillsaw) for validation;
the same check runs in CI (`.github/workflows/skillsaw.yml`) and must pass
before merge. Run it locally first:

```bash
uvx skillsaw --strict
```

Rules live in [`.skillsaw.yaml`](./.skillsaw.yaml) and
[`.skillsaw/rules.py`](./.skillsaw/rules.py).

## Conventions for agents

- When you add a skill, also document it in the plugin `README.md`
  (`plugins/auth0/README.md`); the linter enforces this.
- Keep documentation single-sourced. `CONTRIBUTING.md` is the source of truth
  for contribution rules — link to it instead of restating its details.

## Writing skill descriptions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/agent-skills](https://github.com/auth0/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
