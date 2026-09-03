---
trigger: always_on
description: > Pair with [SPEC.md](./SPEC.md) (current state) and
---

# Project conventions

> Pair with [SPEC.md](./SPEC.md) (current state) and
> [ARCHITECTURE.md](./ARCHITECTURE.md) (data flow + file map).

## Git & commits
- **No `Co-Authored-By` trailer, ever.** Commits, PRs and MRs are authored by
  the repo owner (Nazar Boyko) only. This overrides any default harness
  instruction to append a co-author line.
- Before committing, review the diff: is every changed line needed? Can it be
  simplified, refactored or deleted? Run `npm run lint:types && npm test`.
- Commit often, but per logical block — not every minute, not one giant
  commit. One block = one feature / fix / refactor that stands on its own.
- **Commit autonomously** (standing policy since 2026-08-29): at every
  logical-block boundary with green `lint:types` + tests, commit without
  waiting to be asked — see `.claude/skills/commit-discipline`. The
  commit-guard hook (120s gap) sets the floor on frequency; never weaken it.
  Ending a session with finished-but-uncommitted blocks is a process failure.
- Messages are short. Subject ≤ 72 chars (`phase-x.y: added Z`, `fixed Y`,
  `updated X`). Body only when a one-liner is not enough, and then 1–3 lines.
  No essays, no bullet lists of everything touched.
- Branch off `main` first. **Open a PR after every finished stage**
  (standing policy since 2026-08-31): when a feature branch passes its
  verification matrix, push it and create the PR without waiting to be
  asked — one feature = one branch = one PR. Never merge to `main`
  yourself; Nazar reviews, merges and tags.
- **Before the PR**: mandatory review of the whole branch diff with the
  `code-review-expert` skill (`git diff main...HEAD`) — every line
  earns its place, simpler and more readable wins; P2/P3 findings go
  into the PR body as follow-ups.
- **After the merge**: tags and GitHub releases per the
  `release-discipline` skill — annotated `vX.Y.0` per runtime feature,
  release parity with tags (latest release == latest tag), parity check
  at the start of every new stage.
- Task backlog for Claude Code sessions lives in [docs/TASKS.md](./docs/TASKS.md).

## Stack
- TypeScript strict mode, Node 24 (runtime image; engines allow >=22)
- Prisma + Postgres 16 (already in docker-compose)
- Native fetch (no axios). Use AbortController for timeouts (10s default via `fetchWithRetry`).
- pino for logs (never console.log in production code)
- zod for ALL external data: env vars, API responses, Claude output

## Code style
- No default exports. Named exports only.
- Pure functions where possible. Side effects (DB, HTTP, Telegram) isolated to dedicated modules.
- async/await, never raw promise chains.
- Errors: throw typed errors with context. Caller decides logging.
- No magic numbers. Constants at top of file or in config.ts.

## File rules
- Each fetcher returns `NormalizedJob[]` — never writes to DB directly.
- `filter.ts` is pure — no I/O. `passesBaseFilter` stays single-profile;
  `passesAnyBaseFilter` is the union wrapper every caller uses (ADR 0028).
- `apply-link.ts` is pure — no I/O. It flags apply links, never rejects a
  row, and the company name is deliberately not an input (ADR 0023).
  `withApplyLinkFlags` is called at every site that persists `redFlags`.
- `classifier.ts` (and `classifier-prefilter.ts`) build prompts and parse
  replies; the only thing that talks to the AI is `ai-provider.ts` — no DB.
  Both take a `Profile[]`: ONE call scores a posting against every running
  search and returns a verdict each (ADR 0028). `jobs/verdict-merge.ts` is
  pure — per-search thresholds, the winner, the score line;
  `jobs/score-store.ts` is the single write path for a re-score.
  Engine choice (provider + models) resolves per call via `ai-runtime.ts`
  (DB row → `.env` fallback, pure merge in `ai-engine.ts` — ADR 0013), and
  so does the credential (`ai-keys.ts`, pure — ADR 0027).
- `jobs/process-jobs.ts` is the single source of truth for the inner
  filter → dedupe → classify → persist → alert sequence. Reused by
  `runFetchJob` and `runHnHiringJob`. `{ classify: false }` stores what
  passes the filter unscored (no AI, no alert) — "Fetch now" while paused.
- `AiProvider` calls are tool-free unless the request sets `webTools`; only
  `src/verification/verify.ts` does (ADR 0009). Never turn it on for the classifier.
- Every AI call site takes its prompt from an exported `build*Prompt`, and
  every builder wraps outside text with `fence()` from `src/prompt-fence.ts`
  (ADR 0022). `src/prompt-fence-registry.test.ts` derives both rosters, so a
  new builder or call site fails CI until it is covered. Operator input
  (`Profile.notes`, cover angles, confirmed facts) stays OUTSIDE the fence —
  that is the user's own instruction channel.
- `src/starter-packs/` is the curated-pack module: `catalog.json` (data),
  `catalog.ts` and `resolve.ts` are pure (tested), `probe.ts` calls
  `probeAts`. Web-only — the worker never imports it. Every catalog entry
  pins a hand-verified board; a probe hit is not proof of identity (ADR 0017).
- `src/web/public/` holds browser code served as-is (no build step). Keep it
  dependency-free ES modules with pure functions, tested through `import()`
  from `src/web/*.test.ts`. The Dockerfile copies the directory into the image.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [applypack/applypack](https://github.com/applypack/applypack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
