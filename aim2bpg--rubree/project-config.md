---
trigger: always_on
description: This file defines conventions for contributors and AI assistants (Claude Code) working on this repository.
---

# CLAUDE.md

This file defines conventions for contributors and AI assistants (Claude Code) working on this repository.

---

## Project Background

Rubree is a browser-based regular expression editor for Ruby developers — a modern, open-source
successor to [Rubular](https://rubular.com). The name is short for "Ruby-based regular expression
editor" (first syllables of each word), deliberately echoing Rubular.

**Why it exists.** Rubular was a long-loved tool, but it had grown hard to keep using:
- It ran on an old Ruby version (2.5.9), which made it slow
- It was closed source, so nobody could fix or extend it
- Every match required a round-trip to a server, adding latency
- It had no way to visualize regex structure or preview substitutions

**Design philosophy: "fully in the browser".** Rubree runs Ruby and Rails entirely client-side via
WebAssembly (`ruby_wasm` + `wasmify-rails`). This removes the server dependency, gives instant
feedback, and lets the whole app be hosted as a static site (GitHub Pages). Features that grew out
of this philosophy:
- Railroad diagrams that visualize regex structure (`regexp_parser` + `railroad_diagrams`) —
  the Ruby gem, not a JavaScript library; JS-based alternatives like regexper were evaluated and
  rejected because all diagram logic must compile to WASM and run client-side
- Ruby code snippet generation for the pattern under test
- Shareable permalinks that encode the full editor state in the URL
- ReDoS checks via an embedded recheck Playground

**For more background**, see the launch article:
- はてなブログ: https://aim2bpg.hatenablog.com/entry/2025/11/25/083000

When proposing changes, keep this philosophy in mind: favor solutions that work fully client-side,
keep the app deployable as a static site, and avoid reintroducing a server dependency.

---

## Architecture

Domain logic belongs in `app/models` as ActiveModel-based POROs — no `app/services/` or
`app/forms/`. Read the `architecture-philosophy` skill before adding domain logic, naming classes,
or reaching for a service object; it explains the reasoning so the direction holds going forward.

---

## Task Lifecycle

Every non-trivial task follows this arc. Skipping a phase rarely saves time — it usually shifts
the cost to a later phase where it's harder to fix.

### 1. Plan before writing code

Before touching any file, read the relevant code, map out the approach, and surface unknowns.
Use plan mode for tasks that span multiple files, touch the WASM build, or require a design
decision.

If the task is ambiguous — missing context, conflicting constraints, a genuine choice between
approaches — ask before starting. One clarifying question up front costs far less than building
in the wrong direction and walking it back.

Before starting, confirm:
- What is the scope? (What's in, what's out)
- Is there a hard constraint — WASM compatibility, static-site deployability, the no-services/forms
  rule — that will shape the approach?
- Is a branch + PR needed, or is a direct push to `main` appropriate?
- For tasks expected to span multiple sessions or requiring a design decision up front, create
  `.steering/[YYYYMMDD]-[feature]/tasklist.md` before touching any code — it preserves progress
  across sessions. See the `steering` skill for the format and template.

### 2. State the approach and wait for approval

Briefly describe the plan and get a nod before implementing. If the plan turns out to be wrong
mid-work — a surprising constraint, a misread of the codebase — stop, say so, and re-plan. Don't
silently adapt; the user can't review a decision they didn't know was made.

### 3. Work incrementally

Make one coherent change at a time. After each meaningful step — a passing test, a working
partial — check that things still hold before moving to the next step. This makes regressions easy
to catch and the change easy to explain.

**Pause and ask when genuinely uncertain**: an ambiguous requirement, a surprising finding in the
code, a decision with tradeoffs that belongs to the user. A wrong guess compounds; a quick
question resolves.

### 4. Finish it

When a task turns out harder, slower, or more tedious than expected, finish it — don't quietly
trim scope or push part of it to "later." Reasons like "this is taking too long," "this part is
tricky," or "I'll add the test afterward" are not good reasons to stop partway — they're exactly
the kind of shortcut an AI agent is statistically inclined to take, because the shortcut *looks*
like progress.

The only acceptable reason to skip part of a task is a **technical** one — the plan changed, the
direction changed, or that part turned out to be unnecessary. When that happens, say so plainly
and explain why, rather than letting it quietly disappear from the result.

If a task is too large to finish in one pass, split it into smaller pieces and finish each one —
don't leave any of them half-done. Silently-incomplete work looks identical to complete work
until someone hits the gap later, and by then the reasoning behind it is gone.

### 5. Verify before declaring done

Before reporting a task as complete:

- **Linters**: `bin/rubocop`, `bin/erb_lint --lint-all`, `bin/yarn biome check` (if code changed)
- **Tests**: `bin/rspec`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aim2bpg/rubree](https://github.com/aim2bpg/rubree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
