---
trigger: always_on
description: `agentcov` records which lines of a repository an AI coding agent actually read,
---

# AGENTS

## Repo Overview

`agentcov` records which lines of a repository an AI coding agent actually read,
then renders that as coverage. The package is `agentcov`; the repository is
`trailofbits/agentcov`.

There are two ways activity gets in, and they converge immediately:

- **live hooks**: Codex calls `agentcov hook ...` on each tool use
- **backfill**: a Codex, Claude Code, or Pi session transcript is replayed after
  the fact

Both produce the same hook-shaped payload dict, so everything downstream of
`events_from_payload` is shared. When you fix a parsing bug, fix it once in the
shared layer rather than per agent.

The pipeline is:

```
payload -> parser -> CoverageEvent -> .agentcov/events.jsonl
        -> build_coverage -> .agentcov/coverage.json -> lcov / gcov / html / summary
```

Modules under `src/agentcov`:

- `cli.py`: argparse wiring and command handlers; every subcommand lands here
- `hooks.py`: Codex hook install/uninstall, and `events_from_payload`, the
  single entry point that turns a payload into `CoverageEvent`s
- `parser.py`: shell command parsing (`sed`, `head`, `tail`, `cat`, `awk`,
  `sort`, pipelines, simple `for` loops) and `rg`/`grep` output parsing
- `models.py`: `LineRange`, `ParsedObservation`, `CoverageEvent`, and the
  identity tuple used for dedupe
- `storage.py`: locked append to `events.jsonl`, atomic `coverage.json` write
- `aggregate.py`: events to per-file/per-line coverage, attribution, and
  session rollups
- `report.py`: LCOV, gcov, self-contained HTML, `summary`, and `unread` output
- `config.py`: `.agentcov.toml` loading and exclude matching
- `git.py`: repo root discovery, tracked-file inventory, line counts
- `paths.py`: path normalization relative to the repo root
- `transcripts.py`: locating session JSONL files on disk per agent
- `claude_transcript.py`, `codex_transcript.py`, `pi_transcript.py`: per-agent
  transcript record to payload translation
- `backfill.py`: orchestrates transcript replay and agent auto-detection
- `importers.py`: `agent-coverage`-style JSON import

## Working Style

- Prefer `uv run ...` for repo-local commands.
- Run the targeted test module first, then the full suite if the change touches
  shared behavior. The whole suite takes a couple of seconds, so there is no
  reason to skip it before committing.
- Do not revert unrelated worktree changes.
- `README.md` ends with a `## Roadmap` section listing what is deliberately not
  shipped yet. When you ship something it lists, drop that bullet in the same
  change.

## The Guessing Rule

This is the invariant that matters most in this codebase.

A read event claims "the agent saw these exact lines." Downstream that becomes
coverage someone may rely on to decide what has been reviewed. So when the
parser cannot determine the line range a command actually read, it must record
an **unknown** event with a reason rather than assume the whole file was read.

Concretely:

- Unsupported command shapes go through `_unknown(command, reason)`.
- `rg`/`grep` results are `search_seen`, not reads, and match lines and context
  lines are weighted separately.
- Truncated or errored tool results cap or drop the range instead of widening
  it; see `_cap_claude_read_input` and `_cap_pi_read_input`.
- Unbounded reads whose result gives no bounds are dropped, not treated as
  full-file reads.

When you add support for a new command or tool shape, add the narrow case and
leave everything else falling through to unknown. Widening a fallback to cover
more input is almost always the wrong fix.

`READ_WEIGHTS` in `parser.py` grades how much each shape contributes to
`attention_score`: a focused range read counts for more than `head`/`tail`,
which counts for more than a whole-file `cat`, which counts for more than a
search hit or a search context line. Raw `read_count` and `search_seen_count`
stay separate from the weighted score so a report can always explain itself.

## Common Commands

```bash
make dev            # uv sync --group dev
make lint           # ruff format --check, ruff check, ty check src
make format
make test           # pytest
make coverage       # pytest with agentcov's own line coverage
make check          # lint + test
make build          # uv build

uv run agentcov --help
uv run pytest -q tests/test_parser.py
make test TESTS=backfill
```

## Codebase Notes

When adding or changing a subcommand:

- wire the parser and handler in `src/agentcov/cli.py`
- put the actual work in the module that owns it, not in the handler
- update `README.md` if the user-facing surface changed
- add coverage in the matching `tests/test_*.py`

When adding support for a new shell command shape:

- add the case in `src/agentcov/parser.py` and return `ParsedObservation`s
- add both a positive test and a test that a near-miss variant still comes back
  as unknown, so the new case cannot silently widen later

When adding support for a new agent:

- add a `<agent>_transcript.py` that translates records into hook-shaped
  payloads, plus a `looks_like_<agent>_transcript` detector
- add session lookup in `transcripts.py`
- wire detection and orchestration in `backfill.py`
- do not add agent-specific behavior below `events_from_payload`; if you think

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trailofbits/agentcov](https://github.com/trailofbits/agentcov) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
