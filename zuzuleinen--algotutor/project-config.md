---
trigger: always_on
description: - `main.go` — the only Go file for problems. Always replaced with the current problem template.
---

# Algorithmic Training Project

## Project Structure

- `main.go` — the only Go file for problems. Always replaced with the current problem template.
- `problems/` — one `.md` file per problem, saved as `001.md`, `002.md`, etc.
- `current.md` — always points to the current problem (contains the problem number and description).
- `progress.md` — tracks the user's level (0–N) for each concept.
- `progress.template.md` — blank progress table (all zeros), used for initialization.
- `cards.json` — spaced repetition review cards (created automatically during practice).
- `mistakes.json` — log of recurring error categories (created automatically during practice).
- `resolve.json` — re-solve schedule for solved problems (created automatically on first solve).
- `mix.json` — mix-session state (created automatically when the first mix session starts).
- `retention.json` — per-concept retention score + last-touched timestamp (created automatically on first solve).
- `cmd/review/` — the review TUI program (run with `go run ./cmd/review`).
- `problem-bank.md` — curated problem bank organized by concept and level.
- `docs/` — detail files loaded on demand. See pointers below.
- `claude.md` — this file.

Detail files in `docs/` (read these when the relevant flow fires):

- `docs/concepts.md` — the 32-concept list with prerequisites and teaching order.
- `docs/cards.md` — spaced-repetition card format, SuperMemo rules, examples.
- `docs/mistakes.md` — `mistakes.json` schema, full taxonomy, drill rules.
- `docs/resolve.md` — re-solve mode: ladder, outcomes, `resolve.json` schema.
- `docs/mix.md` — mix mode: retention, timing, outcomes, `mix.json` schema.
- `docs/go-gotchas.md` — Go semantic traps (bytes vs runes, slice aliasing, nil
  maps, integer division on negatives, etc.). **Consult before writing any
  problem statement, example, or nudge that touches the affected mechanic.**

## Initialization

On first interaction, if `progress.md`, `current.md`, or `problems/` don't exist:

1. Copy `progress.template.md` to `progress.md`.
2. Create `current.md` with empty content.
3. Create the `problems/` directory.

## Language

Always Go. `main` function always comes first. Every file must be a valid, runnable Go program.

## Problem Format

Each problem file (`problems/NNN.md`) contains:

- Problem statement
- Function signature
- Example inputs/outputs
- Concept being trained
- Status: `pending` | `solved`

`current.md` contains the current problem number, optionally with a mode suffix:

- `003` — normal mode (first solve).
- `014:resolve` — re-solve mode for problem 014 (see `docs/resolve.md`).
- `034:mix` — mix mode for problem 034 (see `docs/mix.md`).

## Concepts and Progression

The 32 concepts and their prerequisites live in `docs/concepts.md`. Read it when you need to pick a candidate concept,
look up prerequisites, or verify ordering. Track the user's level for each concept in `progress.md`.

### Level Progression Within a Concept

- **Level 0**: Never seen. Start with the simplest possible problem for this concept.
- **Level 1**: Can do the basic pattern. Give a slightly harder variation.
- **Level 2**: Comfortable. Introduce edge cases or combine with a previously learned concept.
- **Level 3**: Strong. Give problems that require this concept as a tool within a larger problem.
- **Level 4+**: Mastery. Interview-level problems featuring this concept.

### Teaching New Concepts (Level 0)

**Assume the user is brand new to algorithms and data structures.** When training a concept at level 0, the user may
never have heard of it before. Do not jump straight to a LeetCode-style problem.

- **Introduce the concept first.** Before (or in) the first problem, briefly explain what the data structure or
  technique is: what it looks like, what operations it supports, what invariant it maintains, why it exists, and when
  to reach for it. A couple of sentences + a tiny concrete example is enough.
- **Name the recognition cue explicitly.** When introducing any concept, state the problem signal that tells you to
  reach for it — e.g., "you reach for sliding window when you see a contiguous subarray or substring problem where
  you're optimizing over all windows." One sentence. This is how pattern recognition gets trained implicitly across
  every concept rather than as a separate topic.
- **Use ASCII art to show structure.** When introducing a concept, include ASCII art diagrams that show the data
  structure's shape, pointer relationships, or how the algorithm transforms data step by step. Walk through the
  diagram with a step-by-step explanation of what happens at each stage.
- **Start with a construction/mechanics problem.** The first problem at level 0 should force the user to *build or use
  the raw structure directly* (e.g. "insert these values into a min-heap and print them out in order" before "find the
  kth largest"). The user should internalize how the structure works before applying it.
- **Progress very gradually within level 0.** If the bank lists a medium-difficulty problem at level 0, it is still too
  hard for a first-exposure problem — precede it with one or more warmup problems you invent, even if they are not in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zuzuleinen/algotutor](https://github.com/zuzuleinen/algotutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
