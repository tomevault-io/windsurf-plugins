---
trigger: always_on
description: This repo has two halves:
---

# Claude for Securing Source Code

This repo has two halves:

- **Interactive skills** (`.claude/skills/`) — read and write files in the
  repo (no target-code execution), run in this session:
  `/quickstart` (front door / Q&A), `/threat-model` (bootstrap, interview, or
  bootstrap-then-interview → `THREAT_MODEL.md`), `/vuln-scan` (static review →
  `VULN-FINDINGS.json`), `/triage` (verify + dedupe + rank a findings pile),
  `/patch` (generate candidate fixes → `PATCHES/`), `/customize` (port the
  pipeline to another stack). Route the user to these for scoping, static
  review, Q&A, and post-run triage.
- **`vuln-pipeline`** (`harness/`) — the autonomous pipeline. Docker + ASAN,
  executes target code, needs a sandbox (see `docs/security.md`). Route here
  when the user wants to actually find and verify crashes.

Docs for each topic are in `docs/`; targets are in `targets/` (canary is the
fast smoke test). The rest of this file is the pipeline operator guide.

---

# vuln-pipeline

Execution-verified vulnerability discovery for C/C++ targets. A find-agent reads
source, crafts inputs, runs an ASAN-instrumented binary until it lands a 3/3
reproducing crash. A grade-agent in a fresh container verifies it. Output is a
crashing input file, not prose.

## When the user asks you to run it

**Recommended: streaming mode with CC as the dashboard.** Walk the user
through recon interactively (show the discovered focus areas, let them
adjust), then launch `--runs N --parallel --stream` in the background. Reports
land in `reports/bug_NN/` as each crash passes grade and the judge clears it —
first one typically in minutes, not hours. Tail the reports directory and
surface each new `report.json` to the user with a summary as it appears.
Status check when asked: "N finds alive, M graded, K reports written — here's
the newest." Stragglers don't block disk writes; a stuck find-agent slows
itself, not the batch.

For first-time use on a target, run a small wave first (`--runs 3 --parallel
--stream --max-turns 100`) so the user gets a feel for token burn and prompt
calibration before scaling up. Check transcripts together if something looks
off.

Launch long runs in the background (tmux with a stable socket, or
`run_in_background`) so you can tail logs, watch `found_bugs.jsonl`, and
answer the user's questions without blocking.

## Running it

Each pipeline phase is a standalone subcommand:

```bash
vuln-pipeline recon <target> --model <model>        # propose focus_areas (YAML → stdout)
vuln-pipeline run <target> --model <model>          # find + grade, one run
vuln-pipeline run <target> --runs N --parallel      # N concurrent finds, round-robin over focus_areas
vuln-pipeline run <target> --auto-focus             # recon first, use its partition
vuln-pipeline run <target> --stream                 # judge + report stream in as grades land (recommended)
vuln-pipeline run <target> --find-only              # skip grade (prompt iteration)
vuln-pipeline run <target> --accept-dos             # DoS-class crashes count as valid finds (benchmark mode)
vuln-pipeline run <target> --resume <results_dir>   # continue a partially-killed batch (skip terminal runs)
vuln-pipeline dedup results/<target>/<ts>/          # group crashes by signature (summary view)
vuln-pipeline report results/<target>/<ts>/         # batch-mode: exploitability analysis per unique crash
vuln-pipeline report results/<target>/<ts>/ --fresh # ignore existing bug_NN/report.json checkpoints
vuln-pipeline patch results/<target>/<ts>/          # generate + verify a fix per unique crash
vuln-pipeline patch results/<target>/<ts>/ --bug N --no-reattack  # one bug, faster (skip re-attack tier)
```

Results → `results/<target>/<timestamp>/`. For `--runs N`: subdirs `run_000/`,
`run_001/`, etc., plus a shared `found_bugs.jsonl` at the batch root.
Reports → `results/<target>/<timestamp>/reports/bug_NN/` (manifest at
`reports/manifest.jsonl`).

## Watching a run

Two output layers, both on by default:

- **Heartbeat** (stdout, every ~25 turns): `[agent] N tool calls (M msgs)` —
  proves the process is alive during long silent stretches.
- **Progress** (stderr, per-action): `[find:N] → Bash: /work/entry ...` — shows
  what the agent is doing. One line per tool call and text block.

Transcripts stream to `{find,grade,recon,report}_transcript.jsonl` as messages
arrive (fsync'd). With `--stream`, judge transcripts go to
`reports/judge_runNNN.jsonl` and the report-grader's to
`report_transcript_..._grader.jsonl`. A mid-run kill leaves readable
transcripts on disk — nothing is buffered in memory waiting for the run to
finish.

**Checking a background run:** tail the log, `cat found_bugs.jsonl` for crashes
landed so far, count `run_*/result.json` files for graded results. Each
result.json writes the moment its run's grade finishes — stragglers don't
block disk writes. With `--stream`, also `ls reports/bug_*/report.json` for
reports landed, `cat reports/manifest.jsonl` for bug-id assignments, and
`cat reports/judge_log.jsonl` for the per-crash NEW/DUP_BETTER/DUP_SKIP
verdicts with reasoning. A stuck straggler can be killed (`docker rm -f
find_<target>_<N>`).

**Rate limits / API errors don't kill runs.** Each agent is one long-lived
`claude -p` session; a 429 or 5xx mid-run is retried inside the CLI first,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthropics/defending-code-reference-harness](https://github.com/anthropics/defending-code-reference-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
