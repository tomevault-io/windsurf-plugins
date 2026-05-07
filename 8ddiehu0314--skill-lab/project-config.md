---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Python CLI tool (v0.7.0) that evaluates agent skills (SKILL.md files) via static analysis, LLM-as-judge quality review, trigger testing, and LLM-based test generation. Produces a 0-100 static score across 37 checks (structure:13, naming:3, description:3, content:13, security:5) / 5 dimensions, plus a 0-100 LLM judge score across 9 criteria / 2 axes (Activation Quality + Instruction Quality).

## Attribution

NEVER include Co-Authored-By lines, "Generated with Claude Code", or any AI co-authorship attribution in commit messages, PR descriptions, PR reviews, or any other output.

## Naming

| Name | Usage |
|------|-------|
| **Skill-Lab** | GitHub repo / project name |
| **skill-lab** | PyPI package (`pip install skill-lab`) |
| **sklab** | CLI command (`sklab evaluate ./my-skill`) |

## Docs

| Document | Contents |
|----------|----------|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Tech stack, data flow, CLI commands, check systems, design patterns |
| [docs/IMPLEMENTATION_PLAN.md](docs/IMPLEMENTATION_PLAN.md) | Vision, roadmap, design decisions |
| [docs/SECURITY.md](docs/SECURITY.md) | 5-layer security scan details |
| [docs/PRIVACY.md](docs/PRIVACY.md) | Telemetry & privacy policy |
| [docs/DEV_STATS.md](docs/DEV_STATS.md) | Telemetry flow, SQLite schema, event types, CI detection |
| [docs/versions/](docs/versions/) | Per-version specs (v0.1.0–v1.0.0) |

After code changes: update `ARCHITECTURE.md` (modules/CLI) and the relevant `docs/versions/vX.X.X.md`.

ALWAYS READ THE DOCS BEFORE ACTIONING

## Commands

```bash
pip install -e ".[dev]"       # install with dev deps
sklab                         # auto-scan repo + getting started guide (no subcommand)
sklab evaluate ./my-skill     # static analysis + LLM quality review (--optimize to chain into optimizer)
sklab evaluate --all          # evaluate every skill in CWD (also: --repo for git root)
sklab check                   # quick pass/fail (exit 0/1, good for CI)
sklab info ./my-skill         # metadata + token estimates
sklab prompt ./skill-a        # export skill as XML prompt
sklab trigger                 # run trigger tests (requires Claude CLI, --provider local|docker)
sklab generate                # generate trigger tests via LLM (multi-provider)
sklab optimize ./my-skill     # LLM-powered SKILL.md optimization (multi-provider)
sklab stats                   # usage statistics
sklab setup                   # configure hooks for Claude Code/Cursor
sklab scan ./my-skill         # security scan (BLOCK/SUS/ALLOW)
sklab list-checks             # browse all checks (--spec-only, --suggestions-only)
pytest tests/ -v                            # run all tests
pytest tests/test_checks.py -v              # run single test file
pytest tests/test_checks.py -k "keyword" -v # filter by keyword
mypy src/                     # type check
ruff check src/ && ruff format src/
/verify                       # runs all of the above (pytest, mypy, ruff check, ruff format)
```

Batch flags available on `evaluate`, `check`, and `scan`: `--all` (CWD) and `--repo` (git root).

## Critical Architecture Notes

See `docs/ARCHITECTURE.md` for full directory structure and data flow diagrams.

### Check System

- **Two check systems**: behavioral (`@register_check` classes in `structure.py`, `naming.py`, `content.py`, `security.py`) and schema-based (`FieldRule` in `schema.py` — append to add a check, no class needed). Per-file counts: structure:9, schema:9, content:13, security:5, naming:1. Dimension counts differ (checks can map to any dimension).
- **Adding a schema check**: append a `FieldRule` to `FRONTMATTER_SCHEMA` list in `schema.py` — no class needed. The `_make_schema_check()` factory auto-generates a registered class per rule.
- **Side-effect registration**: `StaticEvaluator.__init__()` imports check modules (`content`, `naming`, `schema`, `security`, `structure`) to trigger `@register_check` decorators. All checks must be registered before `registry.get_all()` is called.
- **Sync requirement**: `SPEC_FRONTMATTER_FIELDS` in `structure.py` must stay in sync with `FRONTMATTER_SCHEMA` in `schema.py`.
- **Security checks**: `security.py` has 5 separate `@register_check` classes (injection, evaluator, unicode, yaml, size) plus an unregistered `SecurityScanCheck` composite used by `sklab scan`.
- **Check count**: When adding/removing checks, update the "37 checks" count in this file's opening line and run `/update-counts` to sync docs and tests.

### Trace Checks & Runtimes

- **Trace checks**: `tracechecks/` is a parallel registration system using `@register_trace_handler` decorator and `TraceCheckRegistry`. 5 handlers: `command_presence`, `file_creation`, `event_sequence`, `loop_detection`, `efficiency`. Scored under the Execution dimension.
- **Runtimes**: `runtimes/` provides adapters for running trigger tests against live LLMs — `claude_runtime.py` (Claude Code CLI) and `codex_runtime.py` (OpenAI Codex). Both extend `base.py` ABC.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8ddieHu0314/Skill-Lab](https://github.com/8ddieHu0314/Skill-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
