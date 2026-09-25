---
trigger: always_on
description: Practical Claude Code guidance for this repo. Product direction, delivery
---

# CLAUDE.md

Practical Claude Code guidance for this repo. Product direction, delivery
grain, PR report style, evidence boundaries, and commit trailers are defined in
`AGENTS.md` and `docs/DIRECTION.md` — read those first; this file does not
repeat them. `CONTEXT.md` is the glossary for the OMH ↔ Hermes Agent boundary
(which product owns which surface, state root, and TUI); use its terms before
reasoning about anything that touches Hermes Agent.

## What This Repo Is

oh-my-hermes (OMH) is a Hermes-native wrapper orchestration layer: a
deterministic skill catalog, router, and prepared-handoff generator installed
next to Hermes Agent. Core `omh` code makes no LLM, API, or network calls and
never patches Hermes. Pure Python 3.11+, zero runtime dependencies. Two scoped
exceptions. `omh coding fanout dispatch` (explicit opt-in) spawns local agent
CLIs as subprocesses — those CLIs make their own network calls; omh itself
makes none there, and nothing executes without that explicit command. The
`omh_jev_ask` plugin tool is the one place OMH itself opens a connection: it
POSTs typed questions to Jev (TypeSafe, or OpenRouter only with the operator
setting in `<omh_home>/jev/settings.json`) with the user's own key, only when
the user named Jev in that turn and a key resolves. Stdlib only, HTTPS only, a
fixed two-host route table, redirects refused;
`src/plugin_bundle/omh/jev_ask_client.py` is the only network client in
`src/`, pinned by INVARIANT 2's `NETWORK_CLIENT_BRIDGES`. Third-party Jev
plugins are still never called.

## Build & Test

```sh
PYTHONPATH=tests uv run python -m unittest discover -s tests -v   # full suite
PYTHONPATH=tests uv run python -m unittest tests/test_cli.py -v   # one file
uv run python -m compileall -q src tests                          # syntax gate
uv run python -m omh.cli docs workflows --check                   # byte gate
uv run python -m omh.cli docs roles --check                       # byte gate
uv run python -m omh.cli docs claims --check --json               # selected claims
uv run python -m omh.cli docs chain-table --check                 # byte gate
uv run python -m omh.cli docs navigation --check                  # docs structure gate
uv run python -m omh.cli docs skill-sources --check               # watch-closure gate
uv run --group lint ruff check src tests                          # static-analysis gate
git diff --check
```

- Always set `PYTHONPATH=tests` for unittest; test helpers live at tests root.
- Run the smallest test that proves your claim, then broaden if the touched
  surface is shared. Full suite before claiming done.
- `uv run --group lint ruff check src tests` installs the pinned Ruff version
  from the `lint` dependency group (declared in `pyproject.toml`) into the
  project's `uv`-managed environment — no globally installed `ruff` needed.
  CI runs the identical command as its own step. The initial rule set is
  Pyflakes (`F`) only, scoped narrow to stay actionable on a ~135k LOC repo;
  see the `[tool.ruff]` block in `pyproject.toml` for the per-file re-export
  exclusions and the deliberately-not-yet-enforced broad-exception
  (`BLE001`) policy, owned by issue #652.
- That broad-exception policy is a gate, not a comment.
  `tests/test_broad_exception_policy.py` re-derives every broad `except` site
  in `src/` from source and fails when one is not classified as either
  intentional (the failure is classified and surfaced) or needing the #637
  treatment (the failure is relabeled as a normal result). Add the verdict in
  that file when you add a broad `except`; do not record hit counts or line
  numbers in prose, they drift.

## Generated Artifacts Map

Source of truth → generated file → regen command → drift gate:

| Source | Generated | Regenerate | Gate |
| --- | --- | --- | --- |
| `src/skills/catalog.py` + `src/skills/render.py` via `builtin_skill_templates()` / `builtin_skill_reference_templates()` | `skills/*/SKILL.md`, `skills/*/references/*.md` | write template `.content` back to `skills/` (short Python loop; no dedicated CLI writer) | tap-skills staleness inside `docs workflows --check` (missing/stale/extra); `tests/test_router_content.py` |
| Same catalog data | `docs/WORKFLOWS.md` | `uv run python -m omh.cli docs workflows --output docs/WORKFLOWS.md` | `uv run python -m omh.cli docs workflows --check` |
| Same catalog data | `docs/ROLES.md` | `uv run python -m omh.cli docs roles --output docs/ROLES.md` | `uv run python -m omh.cli docs roles --check` |
| Demo case engine | `examples/use-cases/g1-g10-demo-cards.json` | `uv run python -m omh.cli cases demo --all --json` output | parse-equality in `tests/test_application_cases.py` |
| `capability_family_projection()` in `src/capabilities/families.py` | `src/plugin_bundle/omh/tools/capability_families.json` | `uv run python -m omh.cli docs capability-families` | `uv run python -m omh.cli docs capability-families --check`; dict-parity in `tests/test_plugin_capabilities.py` |
| `ulw_inventory_payload()` in `src/skills/catalog.py` via `src/catalogs/ulw_surfaces.py` | marked ULW region of `README.md` | `uv run python -m omh.cli docs ulw-inventory` | `uv run python -m omh.cli docs ulw-inventory --check`; `tests/test_ulw_inventory.py` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rlaope/oh-my-hermes](https://github.com/rlaope/oh-my-hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
