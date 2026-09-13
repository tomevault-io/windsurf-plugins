---
trigger: always_on
description: An Obsidian plugin (and a standalone exporter used for testing) that draws a vault as one
---

# Vault Graph — read this first

An Obsidian plugin (and a standalone exporter used for testing) that draws a vault as one
disc: notes packed into folder wedges on a fixed lattice, animated by a cascade. The repo
is **public**. The recurring failure mode here is reasoning about the code instead of
measuring it: serve the page, drive it, read the numbers.

**Because the repo is public, this file carries only what is true for anyone who clones it.**
Absolute paths, session identity and naming, the session manager's commands, which physical
display a harness seizes, and anything about the maintainer's own setup live in an untracked
`CLAUDE.local.md` next to this one. If you are working on the machine that has one, it is
imported below; if you are a contributor, its absence is normal and nothing here depends on it.

@CLAUDE.local.md

## Laws — every one has a check in `scripts/smoke.mjs` and a section in `.ai-context/invariants.md`

- **The serpentine survives.** Nothing between a note's link weight and its position may step.
- **The rings are independent**, and their thickness is locked; a filter re-packs inside them.
- **The hub is a fraction of the disc, never a radius.** A row-0 dot may not eat into it.
- **The resting disc is on the lattice**, and a settled dot is the size a fresh relayout gives it.
- **`settle()` is a no-op**: the cascade converges before it lands; a jump at the end is a bug.
- **A zero-weight member costs nothing**: a fading note changes no plan, no row, no room.
- **A dot never outgrows its two resting sizes** while a cascade walks; a fade never reverses. With
  **Size dots from the frame** on (a view setting, on by default; `?nofit` turns it off on the page) a
  walking dot may also be held *below* them by its clearance on the frame being drawn, never above.
- **Only depth-1 subfolders with their own tint slot are pushed**; a sub-wedge earns a slot only if it can fill one.
- **The page is scoped**: every CSS rule under `.vault-graph`, every id through `$()`; nothing shipped reaches the network.
- **The layout matches its golden snapshot** on all three fixtures — never regenerate a golden to make a check pass.

## How to work here

- `node scripts/smoke.mjs --only "<substring>"` is the iteration loop. The full suite runs on
  the push to `develop` whose tree it has not measured yet (the pre-push hook; see
  `scripts/suite-stamp.mjs`); do not run it by hand unless asked.
- **Two things may not run twice at once, and `scripts/lock.mjs` is how you know.** Several
  agents work this repo in parallel worktrees, and they collide over two different resources.

  **A screen.** `record-demo.ps1` captures with `gdigrab -i desktop` — it copies a *region of the
  display*, so anything else drawn there lands in the take and ruins it silently: the file exists
  and looks plausible. A recording is not the only claimant — `smoke.mjs` parks every Chrome
  window it opens on one fixed display, and `spike-check.mjs` puts Obsidian there — so the
  lock is named after the **screen**, not the job: `screen-left`, `screen-right`,
  `screen-primary`. **Every harness that places a window takes its own screen lock and releases
  it on every way out** — `smoke.mjs`, `spike-check.mjs`, `record-demo.ps1` — so you do not have
  to remember, and so the claim names the physical display rather than the activity (github#87).

  **The shared fixture store.** Two full-suite runs do *not* fight over ports — ports are
  allocated free and each run gets its own Chrome profile. They fight over `.fixtures/`: a run that
  regenerates deletes every `<name>-*` directory there, including the one a concurrent run is
  reading. That is the `suite` lock, and **it is only about `.fixtures/`** — the display is a
  separate claim under its own name. It bites only when a fixture is stale, which is why it is
  rare and reads as a regression in your branch.

  Keeping them separate is what lets `pre-push` hold `suite` while the `smoke.mjs` it spawns
  holds `screen-left`: two names, two resources, no nesting. Aliasing the two instead — which
  this repo tried first — deadlocks that exact pair, because `aliasHold` blocks on whoever holds
  the alias, the asker included. A sister plugin hit the same deadlock and reached the same design
  independently.

  ```powershell
  node scripts/lock.mjs acquire screen-right --owner "#77 palette"   # blocks; exit 1 = give up
  node scripts/lock.mjs release screen-right --owner "#77 palette"   # always, even on failure
  node scripts/lock.mjs status                                       # who holds what
  ```

  You need those two by hand only for something that seizes a display and is **not** one of the
  three harnesses — a manual Chrome you are driving yourself, say. Never wrap one of the three:
  `smoke.mjs` takes `screen-left` itself, so an outer hold makes its own acquire wait out your
  stale window. `--no-lock` exists for the one caller that legitimately already holds it.

  The lock lives in the OS temp dir, not the worktree, so **every worktree shares one** — and the
  root (`obsidian-vault-locks`) is shared with a sister Obsidian plugin, so if you work on both,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luke321/vault-graph](https://github.com/luke321/vault-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
