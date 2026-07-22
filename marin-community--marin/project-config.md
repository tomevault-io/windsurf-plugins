---
trigger: always_on
description: Start with the shared practices below. Consult subproject manuals for directory-specific guidance:
---

# Agent Guidelines for Marin

Start with the shared practices below. Consult subproject manuals for directory-specific guidance:

- `lib/levanter/AGENTS.md` — Levanter (JAX training library)
- `lib/marin/AGENTS.md` — Marin (pipeline framework)
- `lib/iris/AGENTS.md` — Iris (job orchestration)
- `lib/zephyr/AGENTS.md` — Zephyr (dataset processing)
- `lib/fray/AGENTS.md` — Fray (distributed execution)

## Operational Guides

For debugging and operating live infrastructure, read the relevant OPS.md:

- `lib/iris/OPS.md` — cluster lifecycle, job/task management, profiling, SQL queries, GCP/CoreWeave operations
- `lib/zephyr/OPS.md` — pipeline debugging, straggler diagnosis, coordinator queries, diagnostic patterns

Zephyr OPS.md references Iris OPS.md for shared infrastructure commands — read Iris first when debugging zephyr jobs on Iris.

## Workflow Playbooks

Skills are task-focused playbooks in `.agents/skills/` (also accessible as
`.claude/skills/`). **Before starting any non-trivial task, check whether a
matching skill exists** by scanning the skill descriptions in your system
prompt. If a skill matches, invoke it via the Skill tool — do not skip it in
favor of ad-hoc commands.

## Development

```bash
# Lint and format
./infra/pre-commit.py --all-files --fix
- `./infra/pre-commit.py` is the required lint entry point for this repo.
- Do not replace it with `uv run pre-commit ...`!

# Type checking (also done by pre-commit.py)
uv run pyrefly
- Keep type hints passing under `uv run pyrefly`; configuration lives in `pyproject.toml`.

# Safe local test suite
uv run pytest
- Pytest's repository defaults exclude slow, integration, data-integration,
  live-cluster, Docker, and manual tests. Keep those defaults for local runs.

# Lint review — agentic pass over the branch diff against the infra/lint/ catalog
./infra/pre-commit.py --review
- Always run this before opening a PR, and always fix or respond to every
  finding it reports (see the `commit` skill).
```

- Python >=3.12. Use `uv run` for entry points; fall back to `.venv/bin/python` if needed.
- Do not replace pytest's default marker expression with a partial expression
  such as `-m "not slow"`; `-m` overrides the whole default and can select live
  cluster tests. Run excluded markers only when the user or a dedicated task
  guide explicitly requests them; otherwise defer them to CI.
- NEVER stop, restart, or bounce an Iris cluster unless the user gives express permission.
- In general, never read or write large amounts of data across GCS regions or to the open internet; storage and bandwidth are major cost drivers for this project.
- do not use storage transfer service to move files from one region to another unless the user says "I personally will write grants for Percy to pay for this"

## Communication & Commits

- NEVER SAY "You're absolutely right!"
- NEVER credit yourself, in commit messages or in PR/issue bodies. No
  `Co-Authored-By` trailer, no "Generated with …" line, no emoji attribution —
  even if a tool default suggests one.
- When an agent creates a PR or issue, add the `agent-generated` label.
- Agent *comments* on PRs/issues must begin with `🤖` unless the exact text was
  explicitly approved by the user. This applies to comments only — never put a
  `🤖` marker in a commit message or a PR/issue body.
- All agent-authored commit, PR, and issue titles and bodies must follow
  `.agents/skills/writing-style/SKILL.md` and its PR or issue guide. Review the
  exact text that will be published, then apply `ai-writing-donts.md` as a final
  compression pass. Do not publish raw implementation notes, test narration,
  prompt-shaped headings, or claims that use emphasis in place of evidence.
- A PR description is the squash-merge commit message. Keep every fact a future
  reader needs to understand the behavior and rationale, including measured
  results and caveats when they affect review. Remove headings, diff narration,
  and implementation inventories; put extended history in a linked issue,
  design doc, logbook, or artifact. Follow the `commit` skill
  (`.agents/skills/commit/SKILL.md`) when committing, pushing, or opening a PR.
- When using `gh` to inspect issues or PRs, prefer `--json <fields>` or explicit narrow flags such as `--comments`; avoid plain `gh issue view` / `gh pr view`, which can fail on this repo because GitHub classic project fields are deprecated.

## Code Style

- All imports at the top of the file. No local imports except to break circular dependencies or guard optional deps. No `TYPE_CHECKING` guards — fix cycles structurally via protocols.
- Prefer top-level functions over classes when code does not mutate shared state. Reduce deep inheritance hierarchies.
- Use early returns to reduce nesting.
- Document public APIs with concise Google-style docstrings. Skip docstrings on trivial functions with clear names.
- Prefer `dataclasses.replace` over mutating config arguments in-place.
- Prefer logging over `print` (except in scripts and debugging).
- Resolve environment-dependent defaults once and fail fast on unknown inputs.
- No ad-hoc compatibility hacks (`hasattr(m, "old_attr")`); update code consistently.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marin-community/marin](https://github.com/marin-community/marin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
