---
trigger: always_on
description: * Don't write junior dev dogshit
---

# AGENTS.md

* Don't write junior dev dogshit
* This is gamedev not a webdev
* BABYLON is loaded globally
* Never use RAF (use babylon observables)

## Code Like Ben (Voxels)

- **Don't think, just do**: This is a gross old code base written by people that made lots of dumb decisions, don't try and fix them unless the user explicitly asks you to.
- **Act locally, don't think**: No "discovery" or "architecture." If you see the fix, execute it now.
- **Path of least resistance**: Fix type errors with `any`. Use `// todo` for anything you can't fix immediately.
- **Zero duplication**: Never repeat code. Move shared logic to `utils.ts`.
- **Edit, don't rewrite**: Provide surgical diffs. Never resupply whole files.
- **One-and-done reading**: Do not re-read files you have already accessed.
- **ASCII only**: No em-dashes, smart quotes, or Unicode. Use standard hyphens and straight quotes.
- **No fluff**: No "I understand" or "Certainly." No opening or closing sycophancy.
- **Be direct**: Concise responses, thorough reasoning, simple solutions. No over-engineering.
- **Stay grounded**: If unsure, say so. Never invent file paths or function names.
- **User is truth**: If corrected, treat it as ground truth. User instructions always override this file.

## Committing

* run `pnpm run precommit`and fix the errors before committing.

## Public API changes

If you add, remove or change the behaviour of any route described in
`server/openapi.yaml`, update it in the same PR. Not a follow-up, the same PR.
That file covers the public reads today and is where writes go when they are
documented, so the rule follows the file, not the verb.

`server/test/openapi-routes-test.ts` hard fails when the yaml documents a route
that no longer exists, and warns when a route is missing from it. The hard half
is deliberate: docs that lie are worse than docs with gaps. A renamed or deleted
route without a yaml change is a lie, and it ships to everyone reading the api
page, `llms.txt` or the spec.

The page and `llms.txt` are generated, so run `npm run docs:api` after editing
the yaml and commit what it writes.

## CI

`.github/workflows/check.yml` runs tsc and prettier on every PR. Before adding
anything to it:

1. it must not slow down development
2. it must be insanely fast, 60 seconds or less
3. it must not be subjective (no code complexity analysis)
4. bonus: it must speed up the workflow for all devs and reviewers

Measure the job before you propose it. Anything that makes CI slower may get
backed out.

## UI and "zinestyle" naming things

* direct over fluffy
* human language, not jargon
* specificity instead of abstraction
* warmth and personality
* lean into constraint
* zine/diy energy—intimate and community-focused
* no corporate hedging
* write like you're talking to someone, not a demographic
* lower case for subheadings

If you want your PR merged: **code like Ben**.

This is not “best practices”. This is **ship practices**.

## Ben principles (non‑negotiables)

- **Fix the problem, not the worldview**: one PR = one problem. No “architecture journey”.
- **Surgical diffs**: minimum lines, maximum impact. If it’s noisy, it’s wrong.
- **Delete first**: dead/confusing/duplicated/unused code gets removed. Don’t museum it.
- **Runtime reality wins**: browsers lie, APIs break, users do dumb stuff. Guard it and move on.
- **Fail soft**: prefer “do nothing / return / fallback” over crashing the app for edge cases.
- **Deterministic beats clever**: if detection is flaky, hardcode the sane value and ship it.
- **No ceremony**: fewer layers, fewer abstractions, fewer files, fewer “patterns”.
- **Stop allocating in hot paths**: cache/freeze singletons where it matters.
- **Logs aren’t a lifestyle**: remove spam. Keep only high-signal logs that explain real state.
- **Comments justify constraints**: comment only when there’s a real constraint or weirdness.
- **Name things like a human**: plain names, not corporate nouns. `frames`, `pageHtml`, `iDoc`.
- **Be direct**: commit messages and PR descriptions can be blunt. No marketing. No TED talk.

## The “No Resurrection” rule (maintenance reality)

Voxels is being open-sourced so it can live, **not** so it can be bloated.

- **No feature bloat**: Do not “add” things. If it wasn’t in the core feature set of the final production version, I don’t want it.
- **The UI stays as‑is**: I spent years fighting UI churn. I do not care if you liked the 2021 menu better. I am the one who has to support this code; I want 1/10th of the code for the same features. If you want a different UI, maintain a branch.
- **Dead means dead**: Do not try to bring back “classic” features or “better” old versions of systems that were stripped out. They were stripped for a reason (usually because they were buggy, heavy, or broken).
- **Minimalist stewardship**: This repo is a finished product, not a canvas for your “best possible version” ideas. PRs that add complexity or revert to old, heavy patterns will be closed without debate.

## What Ben-style PRs look like

- **Small surface area**
  - Touch the fewest files you can.
  - Avoid drive-by formatting and lint churn.
- **Clear causality**
  - Every line changed should have a reason you can say in one sentence.
- **Dead code removal is a feature**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cryptovoxels/retro](https://github.com/cryptovoxels/retro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
