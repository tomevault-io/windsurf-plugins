---
trigger: always_on
description: > Wires this repo into Tjakoen's personal standards (how I build with AI, voice, badges, AI-use
---

# CLAUDE.md: Course Console

> Wires this repo into Tjakoen's personal standards (how I build with AI, voice, badges, AI-use
> posture). The standards live publicly at `https://tjakoen.github.io/standards` (source: the
> [`standards/`](https://github.com/tjakoen/tjakoen.github.io/tree/main/standards) dir in the
> portfolio repo) - **reference them, don't fork them.** `AGENTS.md` is a symlink to this file, so any
> tool that reads the cross-tool `AGENTS.md` convention gets the same instructions.

## What this is

Course Console (this directory, formerly the standalone grader-ui repo) is the hosted management and review surface of the GitHub-Native Course Platform.
It is a **data-free GitHub Pages shell** (`site/`) that reads the teacher repos' gradebooks live from
the GitHub API in the browser, shows one grading-review dashboard, and generates the prompts an AI
runs to apply grading decisions (write grades, publish feedback, push to Canvas). The human reviews
and decides; the AI does the writing. The single most important thing to know before touching it:
**writes are tiered, and grades never bypass the review gate.** Anything touching grades, feedback,
or student-facing delivery flows only through an emitted intent prompt that a human runs in a Claude
Code session. The console's own direct writes are limited to two teacher-attested, engine-gated
kinds: filing those intent files, and committing attendance scan batch CSVs from the Scan tab
(validated server-side by the per-repo verify-attendance workflow). Keep every new mutation either
behind an intent or behind an existing dry-run-gated repo workflow - never an unreviewed in-app
grade write. (The local static build was retired 2026-07-24 in favor of the hosted shell; the
maintenance CLIs under `src/` - audit/fix/blanks - stay. The per-repo attendance scanner was
absorbed as the Scan tab 2026-07-25.)

## How I work here (non-negotiables)

The full rulebook is **[AI-DEVELOPMENT.md](https://tjakoen.github.io/standards/ai-development)** +
**[SESSION-LOOP.md](https://tjakoen.github.io/standards/session-loop)**. The short version:

- **I build with AI, out loud, on purpose.** Co-authored with Claude as a practice, not a git
  trailer. The receipt is the README badge + footer, not commit metadata.
- **AI multiplies, it doesn't add.** The AI types; I keep the judgment, the architecture, the final
  call. If I can't explain it, I didn't build it.
- **Definition of done = code + docs synced + green gate.** For this repo the gate is `npm install`
  (it imports `@tjakoen/grain`), `node --check lib/config.mjs src/*.mjs site/*.mjs site/lib/*.mjs`,
  and a clean `npm run bake` (the theme must build from the installed grain package into
  `site/theme.css`). Not one of these, all of them.
- **Write the decision down.** Keep a short record of *why* non-obvious choices were made so the next
  session inherits the reasoning.
- **Hand off when a task finishes.** Gate green, committed, decisions recorded, then emit a compact
  handoff.

## Voice (for any prose in my name)

Follow **[VOICE.md](https://tjakoen.github.io/standards/voice)**. The short version: honest, quirky,
self-deprecating, concrete, opinionated-with-the-why; **no backticks in prose** (fenced code blocks
and this kind of reference doc are exempt, where a literal token has to be exact); **no em-dashes**;
contractions in casual writing, expanded in formal docs. Never claim a benefit I haven't shown.

## README presentation

Follow **[README-STANDARD.md](https://tjakoen.github.io/standards/readme-standard)**: one title emoji,
a curated honest badge row led by the Made with Claude badge, and the text footer. Done at repo start;
re-run the standard's prompt if the stack changes.

## Commit convention

Gitmoji subject prefix. **No AI attribution trailers** (`Co-Authored-By: Claude` etc.). The receipt
behind the "built with Claude" claim is the README badge + footer and the flagship note.

## Repo-specific rules

- **Demo mode swaps the TRANSPORT, nothing else.** `?demo=1` (or the first-run
  "Open the demo" button) makes `lib/gh.mjs` route to `lib/demo.mjs`, an in-memory
  virtual GitHub serving synthetic repos from `lib/demo-fixture.mjs`. Never add an
  if-demo branch to a view, a parser, or a builder: the whole value of demo mode is
  that everything above the transport is the production code path, so a demo that
  renders right is evidence the real one does. When a fixture and a real file shape
  disagree, the fixture is wrong (it already cost us once: pretty-printed
  `assignments.json` silently pushed every flag toggle onto config-writes' whole-file
  fallback instead of its surgical one-line diff). Demo mode must also stay
  non-persistent and non-destructive: caches bypassed, its own decisions key, the
  real Settings config never read, and the flag in sessionStorage. Keep the fixture
  in `.mjs` with invented data only - the Pages tripwire fails the build on a
  shipped `.json`/`.csv` carrying gradebook identity columns, and a real name in
  there would be a PII leak in a public artifact. `npm run test:demo` is the gate.
- **Demo shas are content-derived, like git's. Do not "simplify" them back to a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjakoen/github-native-course-platform](https://github.com/tjakoen/github-native-course-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
