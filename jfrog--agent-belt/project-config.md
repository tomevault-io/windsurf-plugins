---
trigger: always_on
description: This file is the single entrypoint. Read it top-to-bottom and you have
---

# agent-belt - Guide for AI Coding Agents

This file is the single entrypoint. Read it top-to-bottom and you have
everything you need to make a non-trivial change to this repo.

## 1. What this is

A universal evaluation harness for headless CLI agents. It runs multi-turn
scenarios against any agent that has a CLI, scores the results with a
combination of rule-based checks and LLM judges, and aggregates them into
reports.

**The only public surface is the `belt` console script.** Everything
under `src/belt/{commands,runner,scorer,aggregator}/` is internal -
callers that aren't the CLI must go through documented agent/scorer
extension points.

## 2. Setup & first verification

```bash
uv sync                        # creates .venv, installs locked dev deps
uv run belt doctor             # checks Python, agents, providers, env
make check                     # lint + test - same as CI
```

`belt doctor` is the fastest way to sanity-check the install: it
verifies entry points resolve, agents are reachable, judge providers are
configured, and the active clone is the one your `belt` command
points at (a real foot-gun if you have multiple checkouts).

## 3. Where to start (by task)

| Task | Open this first |
|---|---|
| Add a new agent | `docs/glossary/PLUGGABILITY.md` (Authoring an agent), then `src/belt/agent/base.py` |
| Modify an existing agent's behaviour | `src/belt/agent/<agent>.py` (subclass `BaseAgentAdapter`) |
| Add a new scorer plugin | `docs/glossary/PLUGGABILITY.md` (Authoring a scorer), then `src/belt/scorer/base.py` |
| Add a new rule-based check | `src/belt/scorer/rules/scorer.py` (read neighbours first) |
| Change LLM judging or prompts | `src/belt/scorer/llm/scorer.py` |
| Add an LLM provider | `src/belt/scorer/llm/backend.py` (subclass `BaseJudgeBackend`) |
| Change scenario JSON shape | `src/belt/entities.py` + `docs/glossary/SCENARIOS.md` (reference appendix) |
| Add a CLI flag | `src/belt/commands/<cmd>.py` (argparse only) |
| Add a CLI subcommand | New `commands/<name>.py` (thin) + new module under the right phase library |
| Change run-phase pipeline | `src/belt/runner/phases/` |
| Change threshold/aggregation logic | `src/belt/aggregator/` |
| Add a new exporter (CSV / JUnit / vendor plugin) | `docs/glossary/PLUGGABILITY.md` (Authoring an exporter), then `src/belt/exporter/base.py` |
| Tune the export phase or chain on `eval` / `aggregate` | `src/belt/commands/export.py` (registry + dispatch) |
| Author a scenario | `docs/glossary/SCENARIOS.md` + `examples/scenarios/` |

For deeper navigation see [`docs/glossary/ARCHITECTURE.md`](docs/glossary/ARCHITECTURE.md)
("Where Things Live" table). Don't memorise the file tree - `tree src/belt -L 2`
is the canonical reference.

## 4. Design Principles

Skim [ARCHITECTURE.md → Design Principles](docs/glossary/ARCHITECTURE.md#design-principles)
before your first PR - this list is the working summary. Violating any
is a review block.

1. **Don't make phases import each other.** `runner/`, `scorer/`,
   `aggregator/`, and `exporter/` communicate through files on disk only. The
   contract is `entities.py`, not Python imports.
2. **Don't modify base-class signatures.** `BaseAgentAdapter`, `BaseScorer`,
   `BaseJudgeBackend` - extend by subclassing, never by widening the parent.
3. **Don't put behaviour into agent constructors.** Agents are thin
   plumbing; behaviour lives behind framework flags. If you find yourself
   wanting `agent_kwargs={"strict": True}`, add a flag.
4. **Don't `isinstance`-check entity subclasses.** Optional fields with
   safe defaults - both ends degrade gracefully.
5. **Don't raise raw exceptions from user-facing code paths.** Use a typed
   error from `errors.py` with actionable context.
6. **Don't ship a feature without updating the relevant doc** in
   `docs/glossary/`. The doc and the code change in the same PR.
7. **Don't bypass entry-point discovery** for agents or scorers. The
   `--allow-arbitrary-*` flags exist for exceptional cases only.
8. **Don't interpolate agent / judge / scenario text directly into Rich
   panels or Markdown.** Wrap in `rich_safe` or `md_safe` from
   `belt._safe`. Direct interpolation is an injection bug.
9. **Don't add a safety toggle that defaults to permit.** New behaviour
   gates are default-deny + an explicit `--allow-*` / `BELT_ALLOW_*`
   opt-in.
10. **Don't write `"BELT_*"` env-var names as string literals.**
    Import the constant from `belt.envvars` (or
    `_internal_envvars` for the private set).

## 5. Conventions

- **Read 2-3 neighbouring files before writing new code** - match local
  patterns rather than introducing new ones.
- **Namespace package layout.** Source lives in `src/belt/`; imports
  use the `belt.` prefix (`from belt.entities import ...`).
- **Pydantic for any data that crosses a phase or process boundary.**
  Plain dataclasses are fine for in-phase helpers.
- **Typed errors only.** Define new ones as subclasses of `BeltError`
  in `errors.py` with the `(message, hint)` shape.
- **Output integrity.** Anything written to terminal or markdown that
  contains user/agent input must go through `_safe.py` (`rich_safe`,
  `md_safe`). Never f-string user content directly into a Rich/Markdown
  template.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jfrog/agent-belt](https://github.com/jfrog/agent-belt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
