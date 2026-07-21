---
trigger: always_on
description: Canonical instruction set for any coding agent working in this repo. **Read this
---

# AGENTS.md

Canonical instruction set for any coding agent working in this repo. **Read this
first**, before touching any file.

- **Claude Code** loads this via the `@AGENTS.md` import in `CLAUDE.md`.
- **Codex CLI** reads `AGENTS.md` natively (no import needed).

Both tools see identical guidance. Tool-specific additions live in `CLAUDE.md`
(below the `@AGENTS.md` import) — keep them short.

## Mission

`claude-leverage` is Filip Podstavec's personal **AI-dev stack for Claude Code
and Codex**, built to ship **secure and long-term-maintainable software for
clients** when working primarily through AI agents.

The premise: shipping client work with AI agents at velocity is easy; shipping it
so the *next* agent (human or AI) opening the repo in six months can still safely
modify it is the hard part. This stack is the guardrails, conventions, and
on-demand skills that make the second part automatic — continuously as the repo
grows, not just at session start.

Three properties guide every decision here:

1. **Security by default** — deterministic shell hooks block secrets, dangerous
   git operations, and force-push before the model can rationalize past them.
2. **Self-maintaining as the repo grows** — non-blocking nudges flag missing
   AIDEV anchors, missing per-dir AGENTS.md, stale anchors, and review-worthy
   diffs, so maintenance debt surfaces while it's still cheap to fix.
3. **Cross-tool by design** — the same `AGENTS.md`, `SKILL.md` files, and hook
   scripts work in both Claude Code and Codex. Author once.

The point is **not** to save tokens (that thesis was disproven — see "Honest
history" below). It's to make the *next* agent's job easier than the previous
one's, every time.

For what ships in the stack and how it installs, see the
[README](README.md#whats-inside).

## Reading order for new agents

Opening this repo for the first time? Read in this order (progressive
disclosure: minimum context at session start):

1. **This file** (`AGENTS.md`) — what this repo is and how to work in it.
2. [`docs/adr/`](docs/adr/) — *why* the architecture looks the way it does. Skim
   the index; read the ones relevant to your change. Without these you'll propose
   refactors away from load-bearing constraints.
3. [`docs/sessions/`](docs/sessions/) — the last 1–3 session logs. Where the
   previous human + agent left off. Often the highest-leverage orientation per
   token.
4. [`docs/conventions.md`](docs/conventions.md) — repo layout + the full code
   conventions, when you're about to write code.
5. The code itself — by following imports from the relevant entrypoint.

## Code conventions

Essentials below. Full depth — repo layout, structured logging spec, per-dir
AGENTS.md, when to invoke `/adr-new` & `/session-log`, module organization — is
in [`docs/conventions.md`](docs/conventions.md); read it before writing code in
an unfamiliar area. These conventions also ship to client repos via
[`templates/AGENTS.md.example`](templates/AGENTS.md.example).

### Write less, fit in

The north star: the *next* agent should find this code easier to work on than you
did. More code, comments, or abstraction usually makes that *harder*. Prefer the
smallest change that works.

- **Match the surrounding code.** Naming, structure, error handling, comment
  density, and idioms should look like the rest of the module — new code
  shouldn't be identifiable as "the AI-written part." Where existing code is
  inconsistent, follow this AGENTS.md and the cleanest nearby example.
- **Name to fit in.** Detect the repo's casing/separator style (camelCase /
  snake_case / kebab-case; PascalCase for types) and follow it — don't impose
  your language default. Pitch granularity at the function's intent — neither
  `get()` nor `getting_data_from_mobile()`.
- **Comments explain WHY, not WHAT.** The code already says what it does; a
  comment restating the line below it is noise that goes stale. Comment the
  non-obvious — the constraint, the gotcha, the reason for the unusual choice.
- **No speculative abstraction.** Don't add config, parameters, layers, or
  "flexible" interfaces for a use case that doesn't exist yet. When a change makes
  code dead, delete it (git remembers; if the dead code carried an `AIDEV-`
  anchor, note the removal in the commit message).

### AIDEV-* anchor comments

Three grep-able prefixes for load-bearing facts in code:

- `AIDEV-NOTE:` — why this constraint exists / non-obvious invariant
- `AIDEV-TODO:` — known follow-up with enough context to resume
- `AIDEV-QUESTION:` — genuine unknown for the next person (or agent)

Rules: ≤120 chars per line, all-caps prefix. **Before editing a module, run
`grep -rn 'AIDEV-' <module>` first.** Do not silently remove anchors — removing
one requires an explicit decision in the commit/PR message. Add them at
non-obvious decision points (carve-outs, perf workarounds, ordering dependencies,
idempotency tricks); do NOT decorate every function. The `ai-first-nudge` hook
nudges when ≥50 net-new LOC ship without any anchor. Anchors accept an optional
`(by: YYYY-MM-DD)` deadline — see [`docs/conventions.md`](docs/conventions.md).

## Security guardrails

These hooks run on every Bash tool call regardless of which agent invoked them:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Filip-Podstavec/claude-leverage](https://github.com/Filip-Podstavec/claude-leverage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
