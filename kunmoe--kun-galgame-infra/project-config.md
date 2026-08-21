---
trigger: always_on
description: 1. **No background gradients in any UI, ever.** Never use gradient backgrounds in UI design (`bg-gradient-*`, `from-*/via-*/to-*`, `linear-gradient()`, `radial-gradient()`, `conic-gradient()`, etc.); use solid colors from the project's palette.
---

# Project Guidelines

## 铁律 (Iron Rules — non-negotiable; these override every other guideline in this file)

1. **No background gradients in any UI, ever.** Never use gradient backgrounds in UI design (`bg-gradient-*`, `from-*/via-*/to-*`, `linear-gradient()`, `radial-gradient()`, `conic-gradient()`, etc.); use solid colors from the project's palette.
2. **Prefer KunUI components; do not modify KunUI itself.** When adding or changing frontend UI, reach for a KunUI component (`@kungal/ui-*`) first — do not hand-roll a native/custom component unless there is genuinely no KunUI equivalent for what you need. If KunUI appears to have a bug or is missing a feature, **do not edit KunUI's code** (it is a shared upstream library) — report it to the user directly instead, and let them decide how to proceed.


## Core Engineering Principles

> Shared baseline across all KUN Galgame repositories. Defaults, not dogma — apply judgment.

1. All commit messages must be written entirely in English.
2. Comments are governed by the **Comments** section below — the default is none, and what survives is written in English.
3. Keep each source file under ~500 lines where practical; once a file grows past ~300 lines, consider splitting it (a guideline, not a hard rule).
4. Write every frontend function as an arrow function; compose/merge class names with `cn` wherever practical.
5. Deliberately balance elegant modularity against necessary duplication — choose per case instead of always favoring either.
6. Constantly verify that frontend and backend agree on the data: field shapes and response formats must match what each side expects.
7. After every change, watch for unintended side effects elsewhere.
8. If a change requires running a migration, tell the user explicitly at the end — which command, and against which database.
9. Always seek the most modern, elegant solution that fits the project's current state; consult the latest official docs and resources online when useful.
10. Never let the pursuit of elegance or modularity make the code complex or hard to follow, and don't write over-defensive code.
11. A Nuxt page — and any component used as a page/route root — must have a **single real root element**: never `display: contents` (generates no box, so the transition can't attach) and never a leading comment / whitespace / sibling at the template root (a comment is itself a root node). Either trips Nuxt's "does not have a single root node" warning and drops the page-transition enter animation (the page appears without animating). Keep explanatory comments *inside* the root element.
12. Reserve the scrollbar gutter globally — `html { scrollbar-gutter: stable }`, with an `overflow-y: scroll` `@supports` fallback — so the document width is constant across routes. Otherwise navigating from a scrolling page to a height-locked one (no scrollbar) removes the classic scrollbar's ~15px and the centered layout shifts sideways: a "teleport" at the tail of the page transition. This is a browser layout fact, not a transition bug. Use single-edge `stable` (`both-edges` is buggy in Chrome); it's a harmless no-op under overlay scrollbars (macOS/iOS).
13. **One task = one session, and every path has exactly one writer.** Parallel work is allowed only when the user assigns non-overlapping writable paths or system domains. Never rewrite shared Git state a peer may be standing on: on a shared checkout, no branch switch, reset, rebase, merge, cherry-pick, clean, stash or prune. Before editing, record the branch, HEAD, verified `origin/main`, dirty paths and your owned paths, and preserve every foreign change; commit with explicit paths (`git commit -- <paths>`), never `add -A` and never a repository-wide commit. An isolated worktree is the safer default for a long wave — base it explicitly on `origin/main`, not on a local branch that may be holding someone's unpushed work.
14. **Every DB-backed track gets its own test database.** Use the track-specific `TEST_DATABASE_DSN` placed in that session's process environment; if none is assigned, self-provision a throwaway one with `scripts/ephemeral-test-db.sh create <slug>` and drop it with the same script when the session's DB work ends (`sweep` clears leftovers from dead sessions). Never discover or fall back to a DSN from `.env`, and never put a password in a DSN or print one — the ephemeral script's DSN is credential-free by design (auth rides `~/.pgpass`). Give concurrently running services unique ports, and never stop a process whose owner is unknown. Keep `GOMAXPROCS=8` and run DB integration suites with `-count=1 -p 1`. `kun_catalog` is always read-only; `kun_catalog_rehearsal` belongs only to the explicitly assigned rehearsal/aggregation track and is never a general test target.

## Comments

**Default: none.** Code that can be understood by reading it gets no comment. Most code is that code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KunMoe/kun-galgame-infra](https://github.com/KunMoe/kun-galgame-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
