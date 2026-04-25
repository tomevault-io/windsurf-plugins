---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Two things in one repo:

1. **Claude Code plugin marketplace** — `.claude-plugin/marketplace.json` lists 17 plugins under `plugins/`. Each plugin contains skills as `plugins/<plugin>/skills/<skill>/SKILL.md` with YAML frontmatter. Owner: `BillSchumacher`.
2. **Evaluation framework** — Python 3.12 (Pipenv, stdlib only). Runs `claude -p` twice per test case (baseline vs with-skill), captures the full message stream, then scores via Opus 4.6 rubric judge, automated check scripts, and before/after diffs. Results are TSVs in `results/`. Currently 68 eval cases.

## Plugins

| Plugin | Purpose |
|---|---|
| `dev-workflow` | TDD with Gherkin, code quality, git discipline |
| `python-style` | Type annotations, docstrings, PEP 8, streaming |
| `secure-coding` | OWASP Top 10:2025, ASVS 5.0, NIST SSDF/CSF 2.0 |
| `api-design` | REST contracts: OpenAPI 3.1, RFC 9457, pagination, idempotency |
| `observability` | OpenTelemetry, structured logging, SLI/SLO |
| `acceptance-criteria` | Requirements specification: ISO 29148, BDD |
| `skill-orchestration` | Meta-skill for invoking ALL relevant skills, not just one |
| `efficient-code` | Language-neutral algorithmic efficiency (core) |
| `efficient-code-{c,cpp,csharp,go,javascript,php,python,rust,typescript}` | Language-specific efficiency: stdlib helpers + syntax/compiler/runtime gotchas |

## Commands

```bash
pipenv install --dev
pipenv run python -m src.cli list                         # list test cases (68)
pipenv run python -m src.cli run                          # run all (~2-3 hours)
pipenv run python -m src.cli run --cases "security_*"     # glob filter
pipenv run python -m src.cli report --run-id <ID>         # view a past run
pipenv run python -m src.cli new-plugin my-plugin \       # scaffold a plugin
  --description "One-line description"
pipenv run pytest tests/ -v                               # unit tests (18)
```

Full suite is 68 cases at ~2 min each. Use `run_in_background: true` when running from Claude Code.

## Plugin marketplace layout

```
.claude-plugin/
  marketplace.json                 # Catalog (name, owner, plugins[])
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json                  # name, description, version, author
    skills/
      <skill-name>/
        SKILL.md                   # YAML frontmatter + content
```

### Key behaviors

- Skills are auto-invoked by Claude based on the frontmatter `description`.
- **The agent only invokes ONE skill by default**, even when multiple match. Load `skill-orchestration` alongside worker skills for multi-skill tasks.
- Skills can influence how the agent WRITES new code but **cannot reliably make the agent REWRITE existing code** it wasn't asked to change. The system prompt's "don't change what you weren't asked to change" rule is stronger than skill directives.
- When given existing inefficient code to EXTEND, the skill sometimes overrides the existing pattern (Go `string +=` → `Builder`, Python list → set) and sometimes doesn't (C++ `auto` → `const auto&`, Python `sorted[:k]` → `heapq`). Patterns that look like "idiomatic correct code" are hardest to override.

### Adding a new plugin

Use the scaffold command — detects author from git remote:

```bash
pipenv run python -m src.cli new-plugin my-plugin \
  --description "One-line description" \
  [--skill my-skill]  # defaults to the plugin name
  [--author Name]     # override the detected author
```

Author detection (`src/git_meta.py`): parses `git remote get-url origin` owner segment (GitHub/GitLab/Bitbucket/SSH). Falls back to `git config user.name`, then `"unknown"`.

## Eval framework architecture

`src/cli.py` orchestrates the pipeline per case:

1. **`runner.py`** — builds the `claude -p` command. Uses `--output-format stream-json --verbose --dangerously-skip-permissions`. Subprocess calls use `encoding="utf-8", errors="replace"` (required on Windows — default cp1252 crashes on emojis/em-dashes). Baseline uses `--disable-slash-commands`; with-skill uses repeated `--plugin-dir <abs path>`. Each variant runs in an isolated temp dir. `run_claude()` returns `(text, messages)`. There is a separate `run_claude_json()` for `--output-format json` calls (scorer/differ use this).
2. **`scorer.py`** — Opus 4.6 judge call via `run_claude_json()`. Uses `--append-system-prompt` to instruct JSON output; parsed by `parse_judge_response()`. **Do not use `--json-schema`** — it caused API errors. `enrich_with_written_files()` appends `Write` tool contents so the judge sees actual code, not just the agent's summary.
3. **`checker.py`** — runs linters (per code block) and check scripts. Sets `PYTHONIOENCODING=utf-8` and `PYTHONUTF8=1` in child env. Passes `$EVAL_MESSAGES_FILE` (JSON path) and `$EVAL_EXPECTED_SKILLS` (comma-separated) to check scripts.
4. **`differ.py`** — `difflib.unified_diff` + Opus 4.6 diff summary.
5. **`results.py`** — streams rows to TSV incrementally.

`config.py` holds `RunResult(case_id, variant, raw_output, model, timestamp, messages)` and path constants.

## Test case TOML schema

```toml
[case]
id = "unique_id"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BillSchumacher) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
