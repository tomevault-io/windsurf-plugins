---
trigger: always_on
description: This file gives guidance to Claude Code (claude.ai/code) when it works with code in this repository.
---

# CLAUDE.md

This file gives guidance to Claude Code (claude.ai/code) when it works with code in this repository.

---

## Behavioral guidelines

These bias toward caution over speed. Use judgment for trivial tasks.

### 1. Think before coding
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them. Do not pick silently.
- If a simpler approach exists, say so. Push back when warranted.

### 2. Simplicity first
- Minimum code that solves the problem. No speculative features, no abstractions for single-use code, no "configurability" that was not requested, no error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

### 3. Surgical changes
- Do not "improve" adjacent code, comments, or formatting.
- Do not refactor things that are not broken. Match existing style.
- Remove imports and variables that **your** changes orphaned. Leave pre-existing dead code unless asked.
- Every changed line must trace directly to the user's request.

### 4. Goal-driven execution
- "Add validation" means: write tests for invalid inputs, then make them pass.
- "Fix the bug" means: write a test that reproduces it, then make it pass.
- For multi-step tasks, state a brief plan with verification steps before starting.

---

## Done means done

Not half done. Not done except for the part you decided to skip. And not a report about how it will be done.

Five things asked means five things delivered, no matter how long they take. If the fifth is blocked, finish the other four and name the blocker in one sentence. The specific blocker. Not "this needs more investigation."

## Act. Don't ask.

Reversible and cheap? Do it, then tell me. Research, data pulls, analysis, drafts, refactors inside the scope I gave you, testing an API. A question costs me more than a re-run costs you.

Ask first only for: anything that reaches an audience, anything we cannot undo, anything expensive.

Something is broken? Fix it. Reporting an issue you could have fixed turns your work into my to-do list.

## A question is a question

When I ask a question, answer it. Do not implement it.

"Should we use X?" is not "migrate everything to X." "What would it take to add Y?" is not "add Y."

When in doubt, assume it is a question. Answer first. Act when I say go.

## Speed

Optimize for wall-clock speed.

- Parallelize. Independent tasks run at the same time: batch tool calls, spawn subagents concurrently.
- Delegate by complexity: `fast-worker` (Sonnet) for routine work (search, bulk edits, boilerplate, verification), `deep-reasoner` (Opus, read-only) for hard reasoning.
- Keep working in the main thread while subagents run.
- Enough info to act means act. No long option surveys for decisions with an obvious default.
- Speed never trades away quality: same rigor, same verification, same "done means done".
- No conflicts from parallelism: one owner per file. Merge and reconcile in the main thread.

## Short responses

Small words, short sentences, short paragraphs. If you must use a big word, explain it right after. Only return what is necessary.

Tell me what you did, did it work, what I do now.

If I have to decide something: 2 options max, the context I need to pick fast, and which one you would pick.

Keep paths and commands exact. Use ASD-STE100 Simplified Technical English. Never use em dashes.

---

## What this repo is

**Prism**: a Flutter wallpaper app for Android and iOS (Hash Studios, `Hash-Studios/Prism`, default branch `master`). Users browse, favourite, download and set wallpapers, follow creators, earn and spend Prism Coins, buy Prism Premium, and generate AI wallpapers.

| Part | Path | Notes |
|---|---|---|
| App | `lib/`, `test/` | Flutter 3.47 pinned in `.fvmrc`, run through `fvm` |
| Cloud Functions | `functions/` | TypeScript, Firebase Functions v2, region `asia-south1` |
| Firestore | `firestore.rules`, `firestore.indexes.json`, `firebase.json` | project `prism-wallpapers` (`.firebaserc`) |
| Website | `web/` | Next.js on Cloudflare (`wrangler.toml`) |
| Native | `android/`, `ios/`, `pigeons/` | Pigeon host APIs for media and platform calls |
| Vendored plugin | `packages/cloud_functions/` | local fork of the FlutterFire plugin |

Design intent lives in `.impeccable.md` (users, brand, themes, principles). Read it before UI work.

---

## Commands

The Makefile is the source of truth. `make ci` runs the full local gate.

### Daily

```sh
make setup-dev          # fvm + pub get + Doppler check
make run                # flutter run with Doppler dart-defines (DOPPLER_CONFIG=dev by default)
make test               # fvm flutter test
make format-check       # dart format at 120 columns
make ci                 # get, format-check, env/secrets/version guards, analytics-check, analyze, no-dynamic-guard, find-unused-ci
make file-gen           # build_runner (freezed, json, injectable, auto_route), then format
make pigeon-gen         # regenerate Pigeon host APIs
make find-unused        # dead-code report (allowlist: tool/find_unused_allowlist.json)
```

`make analyze` fails on info-level findings. Use `fvm flutter analyze --no-pub --no-fatal-infos` for a clean read.

### Functions and web

```sh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hash-Studios/Prism](https://github.com/Hash-Studios/Prism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
