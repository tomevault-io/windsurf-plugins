---
trigger: always_on
description: This file is repo-local guidance for Codex, Claude Code, and other coding
---

# Agent Instructions

This file is repo-local guidance for Codex, Claude Code, and other coding
agents working on Eval Engineer. Keep it short and update it whenever the repo
workflow, source-of-truth files, or recurring operating rules change.

## Project Intent

Eval Engineer is a general Galileo evidence workflow for improving AI agents,
RAG apps, and future AI systems. Do not shape the product around the first
support-agent fixture. The support-agent cases are validation fixtures, not the
skill's scope.

The near-term product goal is to reduce time-to-RCA for agent builders and
owners while increasing Galileo discoverability for non-developer personas.
Prefer RCA workflows that query log streams, inspect traces/sessions/spans,
identify failure patterns, compare behavior over time, and return grounded
answers with links or stable IDs back to Galileo data.

The north-star loop is:

1. run the AI app
2. log traces and metrics to Galileo
3. fetch compact evidence
4. diagnose failure
5. propose a bounded fix
6. verify with local and Galileo evidence
7. keep only changes that improve measured behavior

## Read First

- `docs/plan.md`: product direction and architecture.
- `docs/tasks.md`: current task checklist and Linear issue mapping.
- `docs/progress.md`: latest work completed and next move.
- `.galileo/learnings.md`: repo-specific durable learnings.
- `blogs/`: product thinking; useful for design intent, but do not treat as
  runtime instructions.

## Skill Rules

- Canonical skill source: `skills/eval-engineer/`.
- Codex install link: `.agents/skills/eval-engineer`.
- Claude install link: `.claude/skills/eval-engineer`.
- Public installer CLI: `eval-engineer` from `pyproject.toml`. Keep it runnable
  through `uvx --from git+https://github.com/Galileo-Agent-Labs/eval-engineer.git`.
- Keep skill distribution skill-first. Future Codex and Claude plugins should
  package the canonical skill source instead of maintaining separate copies.
- Keep `SKILL.md` general across agents, RAG, workflows, metrics, and providers.
- Keep RCA outputs grounded in trace, span, session, metric, dataset, and
  experiment evidence.
- For reference fixtures, give each case a risk profile, quality dimensions,
  and case-specific Galileo metric profile. Do not rely on one global metric
  list to prove safety, quality, performance, and cost across all cases.
- Use `skills/eval-engineer/references/metric-profile-checklist.md`
  and `skills/eval-engineer/assets/metric-profile-template.md` before
  optimizing cost or adding broad fixture coverage.
- When logging test-suite cases to Galileo, include the full expected-output
  contract in `dataset_output`: expected decision, required/forbidden citations,
  required tools, answer constraints, abstention/permission requirements, risk
  profile, quality dimensions, and intended metrics.
- Use `skills/eval-engineer/references/rca-recipe.md` for generalized
  diagnose-fix-verify work and update it when a reusable Galileo RCA pattern is
  discovered.
- When improving command skills, keep frontmatter descriptions trigger-focused,
  keep `eval-engineer` as a narrow router, load references conditionally, and
  promote recurring Galileo mistakes into focused skill gotchas and validation
  loops.
- Keep detailed Galileo mechanics in `skills/eval-engineer/references/`.
- Keep deterministic helpers in `skills/eval-engineer/scripts/`.
- Do not hardcode `TC-1`, the Nexus support agent, Brazil, one model, or one
  metric into the general skill.

### Skill Package Hygiene

- Treat skill frontmatter as routing surface, not documentation. Descriptions
  should name the user job and evidence objects that should trigger the skill:
  Galileo URLs, traces, sessions, log streams, metrics, datasets, experiments,
  tokenomics, RCA, audit, setup, and measurement. Compact wording is good only
  when these trigger nouns survive.
- Keep the router and command skills non-overlapping. `eval-engineer` routes and
  reports workspace status; `eval-fetch` retrieves evidence; `eval-diagnose`
  performs RCA; `eval-measure` defines metric contracts; `eval-dataset` turns
  failures into cases; `eval-cost` handles tokenomics; `eval-audit` reviews risk
  and launch readiness; and so on.
- Budget `SKILL.md` for the workflow an agent must follow after the skill
  triggers. Move Galileo API mechanics, schemas, long examples, and edge-case
  notes to `references/`; move repeatable parsing, summarizing, and comparison
  work to `scripts/`.
- Before merging, renaming, deleting, or splitting skills, verify the kept copy
  is present in the canonical source, the Codex and Claude install links, and
  the installer bundle. Check current usage evidence such as skill mentions,
  `SKILL.md` reads, local install roots, and test coverage before deciding a
  skill is unused.
- Do not delete ignored or untracked skill directories unless the replacement
  path is named or the user confirms they are disposable.

## Working Set

- `.galileo/config.yml`: agent type, metrics, editable files, verification
  commands.
- `.galileo/current/`: current evidence and working artifacts.
- `.galileo/sessions/`: historical evidence.
- `.galileo/eval-dataset/`: candidate, accepted, and rejected eval cases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Galileo-Agent-Labs/eval-engineer](https://github.com/Galileo-Agent-Labs/eval-engineer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
