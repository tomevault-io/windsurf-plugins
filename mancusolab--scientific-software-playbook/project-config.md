---
trigger: always_on
description: 1. This file is for agents and contributors developing this repository itself.
---

## Scientific Software Playbook (Codex Native)

Audience and intent:
1. This file is for agents and contributors developing this repository itself.
2. It defines implementation-facing contracts, internal workflow rules, and source-of-truth plugin paths.
3. For user-facing installation and usage guidance on GitHub, use `README.md`.

Implementation-language scope:
1. This repository is currently tuned for Python/JAX scientific software workflows.
2. Additional implementation languages are currently out of scope for workflow contracts, house-style guidance, and reviewer expectations unless explicitly added later.

This repository supports one operational mode:

1. Global install mode (install into `CODEX_HOME` and run workflows directly in downstream projects).

Scope note: this repository hosts four plugins:
1. `scientific-plan-execute`
2. `scientific-research`
3. `scientific-house-style`
4. `scientific-agent-tools`

Operational workflow remains centered on `scientific-plan-execute`.

Dependency contract:
1. `scientific-plan-execute` is required for orchestration flow.
2. `scientific-research` is required for external-fact validation gates and research workflows.
3. `scientific-house-style` is required for workflow execution and review gates.
4. `scripts/install-codex-home.sh` auto-adds `scientific-research` and `scientific-house-style` when `scientific-plan-execute` is selected.
5. Required house-style skills must be resolvable at runtime:
- `jax-equinox-numerics`
- `jax-project-engineering`
- `functional-core-imperative-shell`
- `python-module-design`
6. Mandatory scientific correctness constraints remain in `scientific-plan-execute`:
   - `simulation-for-inference-validation`
7. `scientific-house-style` should carry reusable implementation-style guidance, while `scientific-plan-execute` enforces orchestration and hard-stop gates.
8. `scientific-agent-tools` is optional maintainer tooling for authoring plugins, skills, agents, and repository context files. It is not required for downstream scientific workflow execution.

## Plugin Assets (Source Of Truth)

### Skills (`scientific-plan-execute`)
- `asking-clarifying-questions`: `plugins/scientific-plan-execute/skills/asking-clarifying-questions/SKILL.md`
- `brainstorming`: `plugins/scientific-plan-execute/skills/brainstorming/SKILL.md`
- `using-plan-and-execute`: `plugins/scientific-plan-execute/skills/using-plan-and-execute/SKILL.md`
- `scientific-kickoff`: `plugins/scientific-plan-execute/skills/scientific-kickoff/SKILL.md`
- `starting-a-design-plan`: `plugins/scientific-plan-execute/skills/starting-a-design-plan/SKILL.md`
- `new-design-plan`: `plugins/scientific-plan-execute/skills/new-design-plan/SKILL.md`
- `validate-design-plan`: `plugins/scientific-plan-execute/skills/validate-design-plan/SKILL.md`
- `set-design-plan-status`: `plugins/scientific-plan-execute/skills/set-design-plan-status/SKILL.md`
- `starting-an-implementation-plan`: `plugins/scientific-plan-execute/skills/starting-an-implementation-plan/SKILL.md`
- `executing-an-implementation-plan`: `plugins/scientific-plan-execute/skills/executing-an-implementation-plan/SKILL.md`
- `writing-design-plans`: `plugins/scientific-plan-execute/skills/writing-design-plans/SKILL.md`
- `writing-implementation-plans`: `plugins/scientific-plan-execute/skills/writing-implementation-plans/SKILL.md`
- `simulation-for-inference-validation`: `plugins/scientific-plan-execute/skills/simulation-for-inference-validation/SKILL.md`
- `requesting-code-review`: `plugins/scientific-plan-execute/skills/requesting-code-review/SKILL.md`
- `verification-before-completion`: `plugins/scientific-plan-execute/skills/verification-before-completion/SKILL.md`
- `systematic-debugging`: `plugins/scientific-plan-execute/skills/systematic-debugging/SKILL.md`
- `test-driven-development`: `plugins/scientific-plan-execute/skills/test-driven-development/SKILL.md`
- `using-git-worktrees`: `plugins/scientific-plan-execute/skills/using-git-worktrees/SKILL.md`
- `finishing-a-development-branch`: `plugins/scientific-plan-execute/skills/finishing-a-development-branch/SKILL.md`

### Skills (`scientific-research`)
- `scientific-internet-research-pass`: `plugins/scientific-research/skills/scientific-internet-research-pass/SKILL.md`
- `scientific-codebase-investigation-pass`: `plugins/scientific-research/skills/scientific-codebase-investigation-pass/SKILL.md`

### Skills (`scientific-house-style`)
- `jax-equinox-numerics`: `plugins/scientific-house-style/skills/jax-equinox-numerics/SKILL.md`
- `jax-project-engineering`: `plugins/scientific-house-style/skills/jax-project-engineering/SKILL.md`
- `polars-data-engineering`: `plugins/scientific-house-style/skills/polars-data-engineering/SKILL.md`
- `functional-core-imperative-shell`: `plugins/scientific-house-style/skills/functional-core-imperative-shell/SKILL.md`
- `property-based-testing`: `plugins/scientific-house-style/skills/property-based-testing/SKILL.md`
- `python-module-design`: `plugins/scientific-house-style/skills/python-module-design/SKILL.md`
- `writing-for-a-technical-audience`: `plugins/scientific-house-style/skills/writing-for-a-technical-audience/SKILL.md`
- `writing-good-tests`: `plugins/scientific-house-style/skills/writing-good-tests/SKILL.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mancusolab/scientific-software-playbook](https://github.com/mancusolab/scientific-software-playbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
