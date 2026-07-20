---
trigger: always_on
description: This file is the operational map for changing Fiction Engine safely. It is written for both human contributors and AI coding agents.
---

# Editing Guide for Coding Agents

This file is the operational map for changing Fiction Engine safely. It is written for both human contributors and AI coding agents.

## First-pass orientation

Before editing behavior:

1. Read `docs/PIPELINE.md` for execution order and ownership boundaries.
2. Search `docs/CODE_MAP.md` for the handler or function involved.
3. Read the relevant schema in `schemas.py` before changing any model output.
4. Read the corresponding commit function before adding fields that should persist.
5. Find the nearest regression test and run the narrow test first.

Regenerate the structural map after moving or adding functions:

```bash
python tools/generate_code_map.py
python tools/project_check.py
```

## Edit routing

| Change | Primary files | Usually inspect too |
|---|---|---|
| Player input interpretation | `agents/director.py` (`director_interpret`) | `schemas.py`, `prompts.py`, pipeline tests |
| Flow planning, resume, or streaming | `agents/runtime.py` (`build_plan`, `_run_pipeline`) | `agents/storage.py`, `checkpoints.py`, pipeline tests |
| Opening scene generation | `agents/director.py`, `agents/perception.py` | `scene.py`, `spatial.py`, `commit.py` |
| Perception or information leakage | `agents/perception.py`, `agents/common.py` delivery helpers | `spatial.py`, `scene.py`, `schemas.py`, perception tests |
| Character decisions or dialogue | `agents/character.py`, `agents/loops.py` | `memory.py`, `scene.py`, `prompts.py` |
| Background (unregistered) presence reactions | `agents/background.py`, `commit.py` (`pick_background_reactor`) | `agents/perception.py` (merge into dialogue_log), `prompts.py`, `schemas.py` |
| Objective action resolution | `agents/director.py` (`director_resolve`) | `schemas.py`, `spatial.py`, `commit.py` |
| Narration | `agents/narration.py` (`narrator`) | narrator prompt in `prompts.py`, output validation |
| Persistence or rollback | `commit.py`, `checkpoints.py` | `db.py`, `memory.py`, restore tests |
| Lore retrieval or hierarchy | `memory.py`, `agents/mapping.py` | `app.py`, lore tests |
| Character/persona format | `character_schema.py` | `importers.py`, editor UI, schema tests |
| Provider behavior | `providers.py` | `app.py` provider routes, `prompt_cache.py` |
| API behavior | `app.py` | matching file in `static/js/` |
| Browser UI | `static/index.html`, `static/js/`, CSS | matching API route in `app.py` |
| Database shape | `db.py` | migrations, snapshot/export/restore code, tests |

## Core invariants

These are architectural guarantees, not stylistic preferences.

### Information boundaries

- A character may use only its perception, memory, knowledge configuration, relationships, private history, and explicit inferences.
- Objective world state must not be copied into a character context merely with an instruction to ignore unavailable details.
- Perception of an action onset and perception of its resolved outcome are separate passes.
- The Narrator should render the player-facing view, not an omniscient reconstruction of every private agent result.

### Authority boundaries

- The player owns the declaration of player speech, thought, and attempted action.
- The Director interprets and resolves declarations; it must not silently replace the player’s declared content.
- Character agents declare behavior but do not author objective success.
- Model output is provisional until deterministic commit code validates and persists it.

### Persistence boundaries

- `steps` and `variants` preserve inspectable intermediate outputs; exactly one active variant should exist per materialized step.
- A checkpoint is established before a pipeline run mutates durable state.
- Stable event identifiers should prevent duplicate memories and duplicate persistence on reruns.
- Primary turn effects are atomic: a commit-domain failure must roll back every durable effect from that turn. Slow provider work belongs in preparation before the outer write transaction.
- New persistent fields require an explicit owner, snapshot/export behavior, restore behavior, and a regression test.

## Source-of-truth order

When several representations disagree, resolve the conflict deliberately rather than updating all copies blindly.

1. **SQLite rows and `world` keys** are the durable runtime state.
2. **Active step variants** are the inspectable result of the current turn.
3. **`PipelineContext`** is the in-memory working state for one execution.
4. **Pydantic schemas** define accepted structured model output.
5. **Prompts** describe desired behavior but do not override deterministic validation.
6. **`Design.md`** describes intended architecture; verify implementation before assuming it is current.

The current architecture still has overlapping physical representations (`world.scene` and normalized world tables). Read the relevant commit and restore paths before changing either.

## Safe change workflow

1. Reproduce the problem with a focused test or saved payload.
2. Identify the earliest stage where the data first becomes wrong.
3. Fix that stage rather than compensating in the Narrator or UI.
4. Validate the structured output in `schemas.py` when possible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [N0819/Fiction_Engine](https://github.com/N0819/Fiction_Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
