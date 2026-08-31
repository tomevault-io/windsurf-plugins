---
trigger: always_on
description: Development contract for this repository. Every local Pi bootstrap run must
---

# AGENTS.md

Development contract for this repository. Every local Pi bootstrap run must
follow it before changing code.

Four audiences, one file:

- **Issue authors** (humans or the monitor): granularity below.
- **Implementer Pi**: read first, TDD, tests, and one PR.
- **Reviewer Pi**: a new `pi --print` after the PR, `prompt_review.md`, and a
  new JSONL.
- **Runner**: labels, observability, review/fix, and merge.

## Issue granularity

Write GitHub Issues so one Pilot implement session can finish them. One Issue
is **one runtime outcome** (when X, should Y, actually Z).

- Size: one observable behavior, a handful of related files, tests included,
  hundreds of lines. Title should work as a test name.
- Open the Issue once the root cause is pinned.

## Minimal implementation (KISS/LEAN)

The positive contract for every implement and review session (Issue #118):
implement the smallest complete change that satisfies the Issue's
acceptance criteria.

- KISS and LEAN are the default: no speculative feature, no
  no-benefit abstraction, no extra framework layer, no fallback, no
  future-proofing, no scope expansion beyond the Issue's acceptance
  criteria.
- 如无必要勿增实体 — do not multiply entities beyond necessity: every new
  file, dependency, state, label, command and abstraction must map
  to an acceptance criterion of the Issue.
- When two designs both satisfy the requirements, choose the simpler one:
  fewer concepts, fewer files.
- The MVP scope below stays unchanged: no database, queue, DAG, daemon,
  risk engine or fallback.

## Implement vs review

- Implementer session: plan, TDD, tests, commit the delivery; the Runner
  pushes the task branch and opens one PR (Issue #186: the deterministic
  Git/GitHub closeout — base fetch and absorb, push, PR creation — is the
  Runner's job, not the agent's).
- After the PR exists, the Runner starts independent review: a new
  `pi --print` with `prompt_review.md` and a new JSONL on the same worktree.

## Read first

- Read the GitHub Issue, the configured context files, `README.md`, and the
  relevant code before touching anything.

## TDD and coverage

- TDD: write a failing test first, then the smallest implementation, then
  refactor.
- External APIs, CLI flags, and HTTP paths are asserted against official docs
  or one real call.
- Blocking commands (Issue #95): any shell command that can block (running
  tests, generator/polling verification, network waits, interactive tools)
  is wrapped in `timeout <seconds> ...` — a timeout is the signal that the
  path needs a fix, never ignorable noise. Testing an unbounded-loop
  function (`while True` poller) requires a termination guard (monkeypatched
  `time.sleep` raising on the Nth call, an injected iteration cap, or
  pytest-timeout): the red phase must fail fast and never hang.
- Python code keeps 100% line and branch coverage:

  ```bash
  /usr/bin/python3 -m coverage run --branch -m pytest tests/ -q
  /usr/bin/python3 -m coverage report --show-missing
  ```

## UI work

- Any UI task must drive the real running app with Playwright: real
  interaction, assertions on the changed flow, console and network error
  checks, and screenshots saved under the run artifacts.

## Fail fast

- Command errors fail fast: log the command, return code, stdout and stderr,
  then raise. Never swallow an error or add a fallback path.

## Automatic observability

- Normal operation publishes progress automatically: no human status
  command, no polling, no supervision. `muyan-pilot status` is a debug
  attachment only — never part of the normal workflow or acceptance
  evidence.
- Journal: while a session runs, the journal gets a heartbeat at most every
  30 seconds and an immediate event on phase/action change. Every line
  carries issue, run id, role (implement/review/merge), phase, elapsed,
  last activity, last action, session and branch. No model/session activity
  for 5 minutes logs an idle warning; the first new activity after it logs
  a resumed event. A stalled (non-`model_wait`) session is recovered
  automatically, not only warned (Issue #94, evidence-based since Issue
  #169): one step per idle window of `PI_IDLE_WARN_SECONDS` since the
  stall was first seen — window 1 checks the Pi descendants that already
  existed before the window (the hung tools: ppid chain + start time from
  `/proc/<pid>/stat`, never a name guess; only Pi descendants, never
  other system processes, never a process spawned after the window began,
  never a zombie that has already exited). If one of them is a coreutils
  `timeout <seconds> ...` command still inside its deadline (start time +
  duration, computed in the monotonic clock domain so NTP realtime steps
  cannot skew it), it is a legitimate long-running command, not a hang:
  the Runner logs one `pi_idle_wait` line (run id, pid, cmdline,
  deadline), reports `recovery=wait` in the progress comment, and pauses
  the escalation — every later window re-evaluates. The nominal deadline
  is BEST-EFFORT (Issue #181): the wrapper's own deadline handling
  (alarm → signal delivery → exit) is best-effort and can be delayed by
  scheduling, so a single "past deadline and still alive" observation is
  not evidence the wrapper failed. The grace is measured in idle windows,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orbi-build/orbi](https://github.com/orbi-build/orbi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
