---
trigger: always_on
description: <!-- PRAWDUCT:ANCHOR — static governance pointer managed by the prawduct plugin. Keep it small and version-free: principles, methodology, and the active version live in the plugin and are injected at session start. -->
---

# CLAUDE.md — Hallucinote

<!-- PRAWDUCT:ANCHOR — static governance pointer managed by the prawduct plugin. Keep it small and version-free: principles, methodology, and the active version live in the plugin and are injected at session start. -->

## Governance (Prawduct)

This repo is governed by **Prawduct**, installed as a Claude Code plugin — not as
committed framework files. The principles, methodology, Critic protocol, and PR
review live in the plugin and are read on demand (run `/prawduct:methodology`);
they are intentionally not copied into this repo.

**Before writing any code, STOP and read the build cycle: `/prawduct:building`.**
Skipping it is the #1 governance failure.

The hardest rules (everything else is in the plugin):

- **Tests are contracts** — fix the code, never weaken a test.
- **No "pre-existing" exception** — fix what you find, or flag why you can't.
- **Never silently drop a requirement** — say so explicitly.
- **Run `/prawduct:critic` after medium+ work** — never write Critic findings
  yourself; the independence is the value.

**Enforcement is structural:** the plugin's Stop hook runs at session end and
**blocks** if code changed against an active build plan with no Critic findings.
The session-start banner shows the active version and what changed — this anchor
stays version-free.

## Hallucinote Behavioral Norms

### Stop only on high-stakes decisions or must-answer questions

Once a workflow is authorized, don't stop between steps to summarize-and-ask. Continue until you hit one of:

- A **high-stakes decision** — expensive to reverse (deletes Live state, modifies shared files, creative lock-in like "what key is this song in").
- A **must-answer question** — you genuinely cannot proceed without input the user hasn't given.
- **The opening elicitation turn** — exactly one consolidated turn, at the start of song work, proposing the load-bearing choices the prompt left open (`/song-brief`). This is the pedagogical carve-out at the front of the work rather than mid-composition, and it is bounded: **one turn, proposals not questions, and nothing already stated is re-asked.** Under-specifying is the user's prerogative; closing the gap is the stage's job. A stage may not emit an unresolved gap — it decides it in-stage, or marks it explicitly open. Where a directed prompt leaves no applicable open dimension, the turn is skipped silently; this is never an excuse for a second turn.

Status updates are fine; status-updates-that-end-in-"what next" are the anti-pattern. Once you've received "keep going" (or equivalent) once, the burden of proof for stopping again is high — you need a *specific* new decision point, not "I finished a phase."

**Pedagogical carve-out.** A *collaborative musical proposal* at a creative fork the user hasn't directed — "E minor with the chorus landing on a release, or do you hear it brighter?" — IS a legitimate stop (it's a creative lock-in), and is distinct from the summarize-and-ask anti-pattern. The test: are you surfacing a real, redirectable choice the user would want to own, or just narrating progress? The former is the propose-and-react discipline (see `intent-collaboration-model.md`'s third register); the latter is the anti-pattern. **Precedence dominates this carve-out:** it covers only choices the user genuinely left open. If they directed the choice — or signalled they'll handle it themselves ("I'll take it from there", "just the skeleton") — *execute and hand back*; do not fork a spec they already gave, and do not stop to propose downstream details they explicitly deferred. Proposing into directed work is friction, not collaboration.

### Creative product prompts vs planning prompts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brookstalley/hallucinote](https://github.com/brookstalley/hallucinote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
