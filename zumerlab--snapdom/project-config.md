---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**The source is the documentation.** Every file in `src/` opens with a header that says what it
owns and what a change must not break; every export carries its reason, the number it was
measured at, and the test that goes red ("Pinned by …", greppable). This file holds only what
the code cannot: where the repos are, the goals, the commands, the testing traps, the release
and verification process, and a map from each mechanism to the file that explains it. When
this file and a source comment disagree, the source wins; fix this file.

## Where the code lives (2026-08-15)

Three SEPARATE repos, three separate clones. Getting this wrong pushes private work to a public remote, so check which directory you are in.

| repo | visibility | clone | branch |
|---|---|---|---|
| `zumerlab/snapdom` | **public** | `~/GitHub/zumerlab/snapdom` | `main` (shipped, 2.24.x) and `dev` |
| `zumerlab/snapdom-v3` | private | `~/GitHub/zumerlab/snapdom-v3` | `main` (default) — the v3 line, this file included |
| `zumerlab/snapdom-agent` | private | its own repo | the agent oracle (was `packages/agent`) |

In each clone `origin` is its OWN repo, so a bare `git push` is always correct. **This was not
true until 2026-08-15**: `snapdom-v3` used to be a git WORKTREE of the public clone, which meant
one shared object store (v3 commits physically lived inside the public `.git`, one stray
`git push origin --all` from leaking), one shared set of refs (only one branch could be called
`main`, and the public line owned the name), and a `node_modules` symlinked to the public clone
so its dependency upgrades silently drove v3's tests. All three are gone: separate clones,
separate objects, separate deps.

- The v3 repo has exactly one branch, `main`. `experimental` was deleted on both sides.
- A GitHub Action on the v3 repo commits `chore: update contributors list` after a push, so the
  remote is one commit ahead right after pushing. Fast-forward, do not panic.
- **`package-lock.json` is COMMITTED** (2026-08-15) and `playwright` is pinned to an exact
  version, no caret. Both exist for the same reason: the browser build is part of the visual
  fixture, not a floating dependency. While the lockfile was gitignored, a fresh `npm install`
  pulled Playwright 1.62.1 where the baselines had been recorded against 1.55.1, and two
  CJK-text demos failed DETERMINISTICALLY until 1.55.1 went back. Upgrading Playwright is a
  deliberate act that re-records baselines, never a side effect of installing. If visual demos
  move after any dependency work, check the Playwright version before reading the code.
- `packages/agent` and the `agent-lab` branch no longer exist here — that product lives in its
  own repo with its history. Do not recreate them.
- v3 is a breaking release. Anything that must reach users NOW (a fix that also affects the
  public `main`) belongs in a separate public-main release, not gated behind v3.

## Non-negotiable project goals

Every change must respect these, in this order:

1. **Speed** — no change may regress capture performance. If a fix or feature adds work to the hot path (`captureDOM` and anything it calls per-node), justify the cost and, when in doubt, measure with `npm run test:benchmark` before/after.
2. **Fidelity** — the rendered capture must match the live DOM. Never trade visual accuracy for convenience. Safari workarounds, bleed math, font embedding, and the clone-in-document measurement pass exist for fidelity reasons; don't "simplify" them without understanding what breaks.
3. **Minimal code** — no decorative code. No helpers for single callers, no speculative abstractions, no options for hypothetical future needs, no defensive checks for impossible states. The library is deliberately small; keep it that way. If a change adds bytes, it has to pay for them.

If a proposed change conflicts with (1) or (2), don't ship it — surface the trade-off instead.

## Commands

All scripts are npm-driven. Tests run in a real browser via Vitest + Playwright (chromium), so `npx playwright install` is required once.

- Build: `npm run compile` (esbuild → `dist/`) · `npm run build` = `npm pack` (a `prepack` hook compiles first)
- Lint: `npm run lint` · auto-fix: `npm run lint:fix`
- Tests: `npm test` (runs the check-only `lint`, `test:types`, `test:release-checks` (`node --test scripts/release-checks.node.mjs`), `test:bundle` (compile + the IIFE global-leak check), then `vitest run --browser.headless`)
- Coverage: `npm run test:coverage`
- Benchmarks: `npm run test:benchmark` · real page, three arms (liquidGL's NaughtyDOM, published v2, this checkout): `npm run compile && node scripts/bench-liquidgl.mjs` (clones liquidGL into the gitignored `.bench-liquidgl/` once; 2026-09-03: 11 / 53.3 / 53.1 ms)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zumerlab/snapdom](https://github.com/zumerlab/snapdom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
