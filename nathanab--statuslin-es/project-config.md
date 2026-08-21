---
trigger: always_on
description: Community gallery of Claude Code status lines — browse configs as rendered previews and copy one to use.
---

# statuslin.es

Community gallery of Claude Code status lines — browse configs as rendered previews and copy one to use.

> **This is an agent-first codebase.** Most changes are written by AI agents. The standards below are enforced mechanically (linter, typecheck, tests, git hooks, CI) **and** by every agent on itself. Do not bypass the gates — no `git commit --no-verify`, no skipping tests, no claiming "done" without showing green output in the same message.

## Stack

Bun (runtime + toolchain) · Vite · TanStack Start (React, SSR) · Better Auth (GitHub) · Drizzle + Postgres · E2B (untrusted-script sandbox). Dependencies are pinned **exact** — no `^` ranges (TanStack Start is RC, Nitro is beta). Upgrades are deliberate, smoke-tested events, never incidental.

## Architecture

The lifecycle of a config: **submit** (`src/submit`) → a render job is **queued** → the
`worker` renders it in the E2B sandbox (`src/render`) → it lands in the **review queue**
(`src/review`) → an admin publishes → it appears in the **gallery** (`src/gallery`),
where visitors **copy it to use** (`src/adopt`).

`src/` map (one responsibility each):
- `routes/` — TanStack Start file-based routes: pages + API handlers
- `submit/` — submission form + flow (slug, obfuscation checks, allowed network hosts)
- `render/` — render pipeline: real E2B runner, fake runner (tests/no key), ANSI parsing, scenarios
- `review/` — admin review queue and publish/reject decisions
- `gallery/` — gallery list queries
- `votes/` — retained legacy voting code · `adopt/` — copy/install a config
- `og/` — Open Graph card images · `legal/` — terms page
- `db/` — Drizzle schema + migration client · `lib/` — shared utils (`env.ts`)
- `ui/` — closed design-system components · `styles/` — tokens (`app.css`)
- `server/` — Nitro server plugins (error context, PostHog)

## The quality bar — non-negotiable

1. **TDD.** Red → green → refactor for all new behavior. Write the failing test first and confirm it fails for the *right reason*. Deviations are allowed only for pure config, pure type definitions, or mechanical refactors covered by existing tests — and you must state the category out loud before deviating.
2. **Run the gate before every commit, show the evidence.** `bun run check` (typecheck + lint + test) must be green, and the output must appear in the same message as any "done / passing / fixed" claim. A run from three messages ago does not count.
3. **Single source of truth (DRY).** No magic strings or numbers duplicated across files. Environment-specific config — URLs, ports, secrets — lives in exactly one place (env vars); everything else derives from it. *(Incident 2026-05-29: `localhost:3000` was hardcoded in four files; now `BETTER_AUTH_URL` is the only source and the dev port + auth origins derive from it.)*
4. **YAGNI.** Build only what the spec/plan specifies. No speculative features, flags, or abstractions.
5. **Small, focused files.** One clear responsibility per file; split when a file starts doing too much.
6. **Clear names.** Name things for what they do, not how they work.
7. **Untrusted by default.** Submitted status line scripts are hostile until proven otherwise — the E2B sandbox is the safety boundary; trust comes from the supply-chain controls (open-source + human review + hash-pinned immutable versions + re-review on every update). See `SECURITY.md`.

## Scope control

- **Make the smallest correct change.** Implement only the requested behavior; do not address hypothetical edge cases or expand the feature without approval.
- **Ask before adding infrastructure.** Get explicit approval before introducing providers, queues, compatibility layers, broad abstractions, new frameworks, or other cross-cutting machinery.
- **Keep the diff focused.** If the work needs more than five production files or starts touching unrelated areas, stop and explain why before continuing.
- **Bound the feedback loop.** Run focused tests once after implementation, then run `bun run check` once. If either fails, make the minimal fix and rerun only what the fix invalidated. Do not repeatedly rerun green checks.
- **Use one review pass.** Treat review findings outside the acceptance criteria as follow-ups, not blockers. Stop and ask before acting on feedback that materially expands scope.
- **Stop after two blocked attempts.** Explain the blocker and options instead of continuing to add complexity.
- **Summarize before the gate.** Show what changed, which files were touched, and why the diff is still within scope before starting final verification.

## Conventions

- **Run it:** `bun run dev` (the app) + `bun run worker` (renders queued jobs locally, or nothing reaches the review queue); full setup in `README.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NathanAB/statuslin.es](https://github.com/NathanAB/statuslin.es) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
