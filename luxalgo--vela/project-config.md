---
trigger: always_on
description: Working guide for an AI agent contributing to Vela: how to analyze requests, implement,
---

# Agent Guide

Working guide for an AI agent contributing to Vela: how to analyze requests, implement,
test, verify, and prevent regressions. Throughout this file, **"the user"** means the
human directing the work.

Vela is an open-source charting library: a headless core, a native renderer, swappable
data providers and scripting engines, a vanilla UI kit, a batteries-included widget, and
a public plugin SDK. Much of its behavior is visual and runtime — it can only be _truly_
confirmed by running it, not by reading code. The method below is built around that
reality.

---

## Before you start

1. **Scratchpad.** Make sure a `.scratchpad/` folder exists at the repo root and is
   listed in `.gitignore`. It is yours — temporary data, notes, throwaway scripts,
   captured output, screenshots. It is **local and internal**: never reference it from
   repo code, tests, or docs.

2. **Browser capability.** Confirm you can drive and inspect a real browser (e.g. a
   Playwright tool) before starting. Most Vela features can only be verified for real in
   a browser. If you cannot, you can still work — but tell the user up front that your
   verification is weaker and some bugs may slip through.

3. **Scope.** Keep changes within this repo. Reading neighboring code is fine; do not
   modify anything outside it without the user's explicit permission.

---

## Output style

Write your answers in the style of the
[Google developer documentation style guide](https://developers.google.com/style):
address the reader directly, prefer the active voice and the present tense, lead with
what matters, and keep sentences short and concrete. When unsure what the guide
prescribes, consult it at that URL instead of guessing.

## The working loop

For anything beyond a trivial edit:

1. **Understand before touching code.** Restate the goal, then read the _actual_ code it
   touches — the relevant ports, the composition root (`src/Vela.ts`), the lint/boundary
   config — instead of assuming. The output of this step is _where_ the change belongs
   and _what the real decisions are_.

2. **Settle the open decisions before implementing.** Research first, form a view,
   propose with a clear recommendation, and let the user confirm or redirect. Don't
   over-ask: pick sensible defaults for obvious choices and state them. Ask before
   implementing, not during.

3. **Sequence so the risky part is isolated and proven first.** If a change needs a
   refactor of working code _plus_ new code on top, land and gate the refactor on its
   own first.

4. **Make the smallest correct change at the right seam**, matching the surrounding
   code's style and conventions. Don't duplicate logic that exists nearby — extend it.

5. **Gate it** (see _The gate_).

6. **Prove it in the real runtime** (see _Testing and verification_).

7. **Diagnose root causes; distrust the happy path — including your own output.** Prove
   a bug's cause (capture, log, reproduce) before fixing it. Re-check what you produced
   adversarially, as if someone else wrote it.

8. **Present, don't commit.** Implement → gate → verify → present → wait for explicit
   approval before committing.

---

## The gate

A change is not done until **four checks pass together** — each catches a different
class of problem:

- `npm run typecheck` — the types line up.
- `npm run lint` — the architecture boundaries hold. Treat a boundary failure as a real
  design violation, not a style nit.
- `npm run test` — behavior did not regress, **plus a targeted test for any new
  behavior**.
- `npm run build` — it still packages (all entries, including type declarations).

None of the four is optional. Passing three and failing the fourth is not finished.

---

## Testing and verification

**Never assume code works. Always test.** Reach for the cheapest tool that can actually
prove the thing:

- **Unit tests** for pure logic and port contracts. The dependency-injection seams let
  you drive the system with fake renderers, fake providers, and fake engines — use them.
- **Throwaway scripts in `.scratchpad/`** for experiments and captured output.
- **The browser for anything visual or interactive.** `npm run playground` serves
  `playground/` on `http://localhost:5190` with Vela imported **straight from `src/`**
  (vite, HMR — no build step): edits are live on save, so you always exercise fresh
  code. What you verify there is the real renderer, the real widget chrome, the real
  event paths.

Two verification rules that earn their keep:

- **Positive proofs, not green gates.** A green suite proves nothing about a NEW
  capability. Every feature needs a check that would _fail if the feature were absent_ —
  a targeted test, or a browser probe that exercises it end to end.
- **Probe computed reality, not presence.** A browser probe must assert what the user
  would actually see: computed visibility (`display`, `backgroundColor` resolved inside
  the themed host, a non-empty bounding rect), real clicks through the UI, actual
  counts changing. An element that exists but portals outside the theme variables, or a
  `[data-state=open]` that renders transparent, passes lazy probes and fails users.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LuxAlgo/Vela](https://github.com/LuxAlgo/Vela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
