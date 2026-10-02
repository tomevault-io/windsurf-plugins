---
trigger: always_on
description: Repo-scoped rules. General operating standards live in the user's global agent rules;
---

# AGENTS.md — Refine Cycle

Repo-scoped rules. General operating standards live in the user's global agent rules;
this file covers only what is specific to this project.

---

## What this is

A **Hermes Agent plugin** that makes the agent improve itself. It reads the agent's own
conversation history from Hermes's SQLite store, finds failures that actually repeat, asks an LLM
for one minimal edit to the agent's skills or memory, applies it through Hermes APIs, and journals
it with a backup so it can be undone.

The mental model that matters: **this code writes into the agent's own future context.** A skill it
creates is loaded in later sessions. A bad edit is not a bug that throws — it is a permanent,
silent behaviour change. That asymmetry is why the invariants below are not negotiable and why
"it didn't crash" is never evidence that a change is correct.

Design reference: `PrimeIntellect-ai/prime-agent`,
`packages/coding-agent/src/core/refinement/refinement.ts` — the mature implementation of the same
idea. Consult it for intent, never transliterate it; it is TypeScript welded to its own session
runtime. When it does something differently, work out the production reason before dismissing it.

---

## Invariants — never weaken these

1. **Refine may create or patch. It may never propose a delete.** The one legitimate delete is in
   the rollback path (`journal.py`), undoing a `create` that refine itself made. That is reverting
   its own work, not removing the user's.
2. **`state.db` is opened `mode=ro`.** Always. Tests build throwaway SQLite files instead.
3. **The base system prompt is immutable.** No feature justifies editing it.
4. **Credential scrubbing happens at the single point where rows leave the database.** Not at call
   sites. Every downstream consumer — the LLM call, the journal, the tool result echoed back into
   context — inherits it. If you add a new path out of the database, it goes through the same
   choke point. This rule exists because scrubbing at call sites already leaked once: the `no_op`
   return path carried raw evidence into the tool result and back into the model.
5. **Backup before edit; every applied edit is rollback-able by journal id.**
6. **The daily edit budget stands** (3/day). It is the blast-radius limit.
7. **No new dependencies.** Python 3, standard library only.

---

## Layout

| File | Owns |
|---|---|
| `__init__.py` | plugin registration: `/refine` command, `refine_run` tool, hooks |
| `config.py` | config reading, and `hermes_home()` / `state_db_path()` — the only place paths are resolved |
| `core.py` | evidence collection, guardrails, apply, run orchestration |
| `sanitization.py` | credential redaction, shared by the evidence and persistence paths |
| `patterns.py` | error normalization, fingerprinting, aggregation, signal gate — pure functions |
| `llm.py` | structured proposal call, schema, json_mode fallback, parse/validate |
| `journal.py` | JSONL journal, backups, rollback, dedup |
| `ledger.py` | usefulness tracking and `/refine audit` |
| `tests/run_tests.py` | the whole suite |

`patterns.py` and `sanitization.py` are pure and have no Hermes dependency — they are testable
anywhere, so anything provable there belongs there rather than in `core.py`.

---

## Platform

The same file runs on a **Linux server** and a **Windows/macOS desktop**. This is not theoretical:
the plugin was completely inert on Windows for a while because a path was hardcoded to `~/.hermes`,
while Hermes actually stores data in `%LOCALAPPDATA%\hermes`. It failed silently — no error, just
nothing, forever.

- Resolve every Hermes path through `config.hermes_home()`. Never build one from `Path.home()`.
- No `fcntl` — it does not exist on Windows. Cross-platform locking only.
- No shell-outs, no POSIX assumptions, no hardcoded `python3` binary name.
- Paths through `pathlib`, never string concatenation.

---

## Verification standard

Running a command is not evidence. **Reading its output is evidence.**

- Run the suite from the repo root: `python -m tests.run_tests`
  `python` on PATH is the Hermes virtualenv interpreter, so `import core, patterns, journal` works.
  Use `PYTHONIOENCODING=utf-8` on Windows — the console is cp1251 and the suite prints ✅.
- **A test that needs real session data skips when there is none; it does not fail.** An empty
  install is not a defect, and a suite that goes red for environment reasons trains people to
  ignore red.
- Green tests are a secondary signal. For anything touching aggregation, thresholds or heuristics,
  **measure against a real database before claiming it works.** Every serious defect in this
  project so far was invisible on synthetic input and obvious on real data — one bogus pattern with
  398 hits swallowing every real failure; successful results counted as errors because they contain
  `"error": null`.
- State plainly what you verified and what you did not. If a path could not be exercised, say so.

---

## Privacy

The trajectory is the user's private conversations. Treat it that way.

- Never send real session content to an external service or model. Synthetic fixtures only.
- `refine_journal.jsonl`, `backups/`, `skill_stats.json` are gitignored because they contain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bergschloss/Refine-Cycle-for-Hermes-Agent](https://github.com/Bergschloss/Refine-Cycle-for-Hermes-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
