---
trigger: always_on
description: This file is the canonical maintainer guide for coders (including AI coding agents) working on the OpenMapStack repository. Keep repository-wide development guidance here and keep it vendor-neutral.
---

# AGENTS.md

This file is the canonical maintainer guide for coders (including AI coding agents) working on the OpenMapStack repository. Keep repository-wide development guidance here and keep it vendor-neutral.

## What this repository is

OpenMapStack is a managed agent-skill codebase, not an OpenMapStack GIS analysis project.

The repository develops and quality-assures a distributable geospatial agent skill:

- `SKILL.md` and `references/` are **product artifacts** consumed by agents in other projects.
- `templates/` and `examples/` define and demonstrate the artifacts the skill should cause those agents to produce.
- `openmapstack/` is executable support code for validating, running, inspecting, and verifying those artifacts.
- `evals/` measures whether the skill and its supporting contract produce correct, reproducible behavior.
- `tests/` tests the repository's Python implementation and eval infrastructure.

Do **not** treat `SKILL.md` as maintainer instructions for this repository. Read or edit it when the task concerns the shipped skill behavior, but do not preload or follow its GIS workflow merely because you are modifying this codebase.

## Sources of truth

Read only what is relevant to the current issue.

| Area | Source of truth |
|---|---|
| Shipped agent behavior | `SKILL.md` |
| Detailed shipped guidance | `references/*.md` |
| Reproducible project contract | `references/project-spec.md` |
| Project templates | `templates/` |
| Canonical worked integration example | `examples/tartu-development/` |
| CLI, schema handling, reusable checks | `openmapstack/` |
| Eval architecture, cases, scoring, live adapters | `evals/README.md`, `evals/` |
| Maintainer architecture/rationale/debugging knowledge | `docs/maintainers/` |
| Implementation tests | `tests/` |
| User-facing install and usage docs | `README.md` |

Prefer the code/schema/test that owns a behavior over a prose summary elsewhere. Avoid duplicating detailed rules between files; link to the authoritative source instead.

## Agent and provider neutrality

OpenMapStack must remain useful across coding agents and model providers.

- Shared skill instructions, project contracts, schemas, checks, and scoring semantics must not assume Claude, Codex, OpenAI, Anthropic, or any other vendor.
- Provider-specific behavior belongs behind an explicit compatibility surface such as `evals/adapters/`, `.claude-plugin/`, `agents/`, or a provider-specific CI leg.
- Treat supported agents as peer implementations of the same behavioral contract. Do not make one adapter's quirks the generic architecture.
- Keep normalized eval evidence and scoring vendor-neutral; retain raw provider events only as auditable adapter evidence.
- A provider-specific integration may be added without forcing its terminology, model IDs, environment variables, or protocol into shared interfaces.
- Repository-wide maintainer instructions belong in this file. Tool-specific bootstrap files should only import or point to `AGENTS.md` plus genuinely tool-local compatibility notes.

Named Claude/Codex/OpenAI integrations in tests, adapters, workflows, and install docs are intentional compatibility surfaces and do not violate this rule.

## Cold-start workflow

For a new issue or increment:

1. Read this file, the issue/task, and `git status`/the current diff.
2. Locate the owning module and its tests; inspect only the relevant files first.
3. Read `docs/maintainers/` only when architecture, rationale, recurring traps, or prior decisions could materially affect the task.
4. Read `SKILL.md` or detailed references only if the change affects shipped skill behavior.
5. Make the smallest coherent change that resolves the issue; avoid unrelated refactors.
6. Add or update the appropriate test/eval evidence.
7. Run the narrowest relevant checks, then broader checks when the change warrants them.
8. Update README/reference docs only when their public contract or usage has changed.

Do not recursively read the whole repository to "understand the project" before working.

## Durable project knowledge

Tool-specific memory (Claude/Gemini auto-memory, Pi/Codex session history, IDE agent state, etc.) is a convenience cache, not a source of truth.

When work reveals a non-obvious fact that will matter across future sessions or tools, promote it to the repository **only when rediscovery would be materially expensive or error-prone**:

- repository-wide workflow/routing rule -> `AGENTS.md`;
- cross-cutting architecture or invariant -> `docs/maintainers/architecture.md`;
- recurring hard-to-rediscover diagnostic trap -> `docs/maintainers/debugging.md`;
- consequential architectural choice and rationale -> `docs/maintainers/decisions/`;
- behavioral guarantee -> owning code plus tests/evals;
- product/skill behavior -> `SKILL.md` or the owning `references/` document.

Do not persist facts that are obvious from the code, cheap to rediscover, specific to one transient session, or already owned by another contract. Do not let a tool-specific private memory become the sole record of a project invariant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaakla/openmapstack](https://github.com/jaakla/openmapstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
