---
trigger: always_on
description: **If we choose to do something, we do it perfectly — right now, in
---

# Octopus Development Rules

## Do It Right The First Time (no MVPs, no future-polish)

**If we choose to do something, we do it perfectly — right now, in
this session.** No "minimal fix", no "MVP for now", no "we'll polish
this later". No deferral of cleanup to a "follow-up" item.

This rule is non-negotiable. Specifically that means:

- Never ship a half-done implementation and document the rest as
  "future work". If the full thing isn't worth doing right now, then
  don't start it at all.
- Never park the cleaner version in a "deferred work" note
  *instead* of doing it. Genuine deferrals (work that needs a real
  second use case, an external dep, or a user decision) belong in
  the relevant plan doc's §10 "What this defers"; nothing else is a
  legitimate place to stash "felt too long".
- Never add a comment like `# TODO: handle X properly later` or `#
  HACK: works for now`. If `X` matters, handle it in this change.
  If it doesn't matter, delete the comment.
- When the user asks "fix this", interpret it as "fix it the way a
  careful engineer with infinite time would" — not "ship the
  smallest patch that no longer crashes".
- "MVP" is not a status the user has to accept. There is no future
  in which a later session will go back and polish; in the AI era
  we have the bandwidth to do it right *now*, here, in one go.

This rule exists because past sessions repeatedly took the shortcut
and then had to be told to go back and do the real thing. Skip the
shortcut. Do the real thing the first time.

## After Every Code Change

You MUST verify your changes before considering them done:

1. **Backend unit tests**: `.venv/bin/pytest tests/ -v` (882 tests; the
   real-CLI tests auto-skip unless their binary is on PATH —
   `test_backend_claude_code_real.py` + `test_schedule_ai_real.py` +
   `test_showme_ai_real.py` + the 2-hop / question-loop / 3-hop cases in
   `test_delegations_real.py` need `claude`; `test_backend_codex_real.py` +
   `test_codex_login_real.py` need `codex`; `test_agent_memory_real.py` +
   the claude→codex case in `test_delegations_real.py` need **both** —
   run with the nvm bin prepended, see Conventions)
2. **Frontend unit tests**: `cd web && bun run test` (84 tests)
3. **TypeScript check**: `cd web && npx tsc --noEmit`
4. **E2E tests**: `cd web && bun run test:e2e` (67 tests, ~3.5 min, Playwright
   auto-starts servers). Split into two buckets for dev iteration —
   `bun run test:e2e:fast` (35 pure-UI tests, ~16 s — login / sessions /
   dialogs / sidebar / virtualized chat / attachments / etc.) and
   `bun run test:e2e:llm` (32 real-LLM tests, ~3 min — chat, /schedule,
   /showme, /archive, mcp__bg__run, AskUserQuestion, agent-collaboration,
   notifier, codex sign-in, handoff/pull). Anything that drives a real
   `claude` / `codex` turn carries `@llm` in its describe title; the
   `:fast` script uses `--grep-invert @llm`. Telegram bridge tests have
   their own config and run via `test:e2e:bridge`.

**Zero test failures are acceptable.** All tests must pass before committing. If a test fails, investigate and fix it — do not ignore, skip, or dismiss any failure as "flaky" or "pre-existing".

## Test Coverage

| Suite | Tool | Count | What it covers |
|-------|------|-------|----------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [archeryue/Octopus](https://github.com/archeryue/Octopus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
