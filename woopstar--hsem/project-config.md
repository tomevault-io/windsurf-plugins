---
trigger: always_on
description: **Always read `.github/memories.md` before starting any work.**
---

# Custom GitHub Copilot Instructions

## Repository Memory (Read First)

**Always read `.github/memories.md` before starting any work.**

It contains:
- Module responsibility map for all planner and utils files
- Canonical patterns you must use (never re-invent)
- MILP variable vector layout (6*n)
- File size limits and oversized files
- Cycle cost formula with the mandatory 2x denominator
- All open refactor and bug issues (#439–#447)
- Huawei entity wiring protocol
- Logging and testing rules

---

## Standard Issue-Solving Workflow

When asked to solve a GitHub issue, always follow these steps in order:

0. **Checkout main and pull latest**
   ```bash
   git checkout main
   git pull
   ```
1. **Read the GitHub issue** — Understand the problem fully before touching any code.
2. **Read `.github/memories.md`** — Check if the issue touches a known pattern or canonical helper.
3. **Create a branch** using the issue prefix and a short slug.
   - Format: `<type>/<issue-number>-<slug>` — e.g., `fix/444-milp-cycle-cost`
4. **Understand the relevant code** — Search and read the affected files before making changes.
5. **Implement the smallest safe fix** — No unrelated changes, no broad refactors.
6. **Update documentation** — Update every docs/ file that describes the changed behaviour
   (planner guide, spec, config flow reference, memories.md, README, etc.).
7. **Add or update regression tests** — Cover the bug or new behavior.
8. **Run the relevant tests** — `pytest tests/` or the targeted test file.
9. **Run lint/type + quality checks** — all four must pass before opening a PR:
   - `tox -e lint` — isort + black + ruff format + ruff check
   - `tox -e typing` — mypy type checking
   - `tox -e quality` — pyright + vulture
   - `tox -e py313` — pytest with coverage
10. **Report a summary** including:
   - Issue title
   - Branch name
   - Files changed
   - What changed and why
   - Tests added or updated
   - Test and lint results
11. **Create a pull request** linked to the issue using `Fixes #<ISSUE_NUMBER>` in the description.
12. **Keep the PR up to date** — after every follow-up commit on a branch that already has an open
    PR, update both the PR title and description to reflect the current state of all changes made.
    Tick off any completed acceptance criteria in the PR checklist.
    - Use `gh pr edit <PR_NUMBER> --title "..." --body-file <file>` — write the PR body
      to a temp file first, pass it with `--body-file`, then delete the file.
    - **Never** pass a multiline body inline via `--body "..."`: PowerShell corrupts the
      content (newlines become `∙` characters; backticks become `\x5c` escapes).

## Planner Specification Rule (Mandatory)
- **Always read `docs/hsem-planner-spec.md` before touching any planner code** — engine, cost
  function, SoC simulation, candidate generation, slot population, or safety gates.
- **Every planner change must satisfy all spec invariants**: energy balance per slot, SoC bounds,
  cost identity (`winner.cost == final_output.cost`), terminal-SoC accounting, and safety gates.
- **Update `docs/hsem-planner-spec.md`** when a change intentionally alters planner semantics.
  Spec and implementation must never diverge silently.
- **Add or update tests** covering the affected invariants for every planner change.
- A planner PR is not done until: spec is consistent, invariant tests pass, and lint is clean.
- See `AGENTS.md` → **Planner Specification** for the full compliance checklist.

## Documentation Update Rule (Mandatory)
- **All documentation that describes the changed behaviour must be updated in the same PR.**
  This includes, but is not limited to:
  - `docs/hsem-planner-guide.md` — planner inputs, outputs, cost function, scenarios
  - `docs/hsem-planner-spec.md` — specification invariants and formulas
  - `docs/hsem-config-flow-reference.md` — config/options flow step tables
  - `docs/ev-charge-plan-setup.md` — EV planned load setup guide
  - `.github/memories.md` — canonical patterns, module map, open issues
  - `README.md` — user-facing feature descriptions and links
- **Check every docs/ file before closing a PR** — if a file describes something you changed,
  update it. Stale documentation causes confusion and bugs.
- **A PR is not done until all affected docs are consistent with the implementation.**

## Huawei Solar Sensor Rule (Mandatory)
- **Always use entities exposed by `wlcrs/huawei_solar`** for every inverter/battery value.
- Never hard-code numeric battery constants — always source from the live HA entity.
- If a value is needed but not yet wired into HSEM, add it through the full stack:
  `const.py` → `flows/huawei_solar.py` → **`translations/en.json`** (both `config` and
  `options` `huawei_solar` steps) → `models/sensor_config.py` →
  `custom_sensors/config_reader.py` → `custom_sensors/state_collector.py` →
  `models/live_state.py` → `coordinator.py`
- **Always check `docs/huawei_entities.md` first** for the verified list of available HA entities
  before searching the upstream `wlcrs/huawei_solar` repo or guessing an entity ID.
- See `AGENTS.md` → **Huawei Solar Sensor Usage Rule** for the full wiring protocol.

## Canonical Helpers (Mandatory)

These helpers exist — never re-implement them inline:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woopstar/hsem](https://github.com/woopstar/hsem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
