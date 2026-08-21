---
trigger: always_on
description: > Repository-level agent instructions and skill registry.
---

# uncle-bob

> Repository-level agent instructions and skill registry.

## Version & Tooling Policy

Skills MUST NOT hardcode version numbers. Detect the project's actual versions from `package.json`, lockfiles, and `pyproject.toml`. Use web search to verify latest stable patterns. When the project's version differs from latest stable, note the gap and recommend an upgrade path.

## Shell Entry Points

Use these command-entry defaults when working in this repository:

1. use `uv run python ...` for repository Python scripts, tests, and helper
   tooling
2. for short ad hoc local inspection when `uv` is not needed, use the
   interpreter command that matches the local environment rather than assuming
   one universal fallback name
3. prefer interpreter-explicit commands and confirm the command exists in the
   current environment before relying on it
4. prefer `task` wrappers when the repository already exposes a check or helper
   through `Taskfile.yml`

## Policy Versus Defaults

This repository is intentionally opinionated, but not every opinion has the
same status.

Repository policy means the rule is explicitly part of the repository contract
and is often backed by validation, integrity checks, or mandatory skill
instructions.

Strong defaults mean the repository currently recommends the approach because it
fits best here, while downstream adopters may still need to adapt it
deliberately.

Freshness review for volatile framework and tool guidance is warning-only by
default. Do not turn freshness into a blocking gate unless the repository
explicitly promotes it beyond advisory status.

| Technology   | Version Policy      | Primary Tool     | Fallback |
| ------------ | ------------------- | ---------------- | -------- |
| Node.js      | Latest LTS          | bun              | npm/node |
| Python       | Latest stable       | uv               | pip      |
| TypeScript   | Latest stable       | (via bun or npm) | —        |
| Vue          | Latest stable       | —                | —        |
| Nuxt         | Latest stable       | —                | —        |
| Tailwind CSS | Latest stable       | —                | —        |
| Pydantic     | Latest stable (v2+) | —                | —        |

## Repository And Distribution Boundaries

This repository is the authoring and validation factory for the distributable
agent customizations. Do not assume every repository file ships to downstream
projects.

Distributable surfaces:

1. `.agents/skills/`
   The portable skill payload. Skills may depend on their own `SKILL.md`,
   `references/`, `assets/`, `scripts/`, and explicitly named sibling skills.
   Skills must not depend on repo-maintenance scripts, root `Taskfile.yml`, CI,
   plugin metadata, root documentation, `.ub-workflows/`, or optional agents.
2. `.github/agents/ub-teacher.agent.md`
   The optional teaching-agent payload. The agent may know about itself and the
   skills. Skills must not require this agent or mention it as a runtime
   dependency.
3. Repository-only surfaces
   `AGENTS.md`, `README.md`, `Taskfile.yml`, `pyproject.toml`, `plugin.json`,
   `.github/workflows/`, `scripts/repo-maintenance/`, `tests/`, `docs/`, and
   factory workflow artifacts are repository truth. These files may know about
   the skills, the optional agent, packaging, CI, and validation rules.

When editing:

1. Keep files under `.agents/skills/` host-agnostic, repo-agnostic, and
   portable unless the skill explicitly owns the referenced asset or helper.
2. Keep `ub-teacher` independent from factory-only paths except where it needs
   to locate the distributable skills in this repository.
3. Put factory-only planning, validation, release, and maintenance knowledge in
   repository-only surfaces, not inside distributable skill contracts.
4. If a downstream project needs repo behavior, bundle it through skill-owned
   `scripts/` or `assets/`, or describe how to detect and adapt the target
   project instead of pointing at this repository's root tooling.

## Documentation Synchronization Policy

This repository has three active truth surfaces:

1. `.agents/skills/` and `.github/agents/`
   These are runtime and distribution truth for the installed skills and custom
   agents.
2. `docs/` and `README.md`
   These are published explanation truth for humans evaluating and adopting
   the portable skills.
3. `AGENTS.md`, `Taskfile.yml`, `.github/workflows/`, `scripts/`, and
   repo-maintenance checks
   These are repository-control truth for local workflow, CI, validation, and
   synchronization rules.

Keep those surfaces synchronized. Documentation drift is a defect when any of
the following are true:

1. a real skill under `.agents/skills/` has no matching published docs page
2. docs mention a skill, custom agent, command, workflow, or path that no
   longer exists
3. skill or custom-agent behavior changes without matching documentation
   updates
4. public docs describe repository-maintenance internals instead of portable
   skill behavior
5. documentation introduces behavior, guarantees, or workflow steps that the
   skill contracts do not support
6. workflow, validation, deployment, or command docs no longer match
   `Taskfile.yml`, `package.json`, scripts, or GitHub Actions

Synchronization is bidirectional:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robert-hoffmann/uncle-bob](https://github.com/robert-hoffmann/uncle-bob) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
