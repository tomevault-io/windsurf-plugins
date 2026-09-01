---
trigger: always_on
description: A GPU-rendered (GPUI, à la Zed/Ghostty), Rust, Excel-compatible spreadsheet built to be
---

# FreeCell

A GPU-rendered (GPUI, à la Zed/Ghostty), Rust, Excel-compatible spreadsheet built to be
**stupid-fast on huge sheets** (Excel-max = 1,048,576 rows × 16,384 cols). Engine =
**IronCalc**; UI = **GPUI** (custom raw-gpui grid + gpui-component for chrome).

Built agentically in **staged de-risking rounds**. There is **no production app yet** —
current work is experiments + specs that decide whether/how to build it.

## Layout

- **`specs/projects/`** — spec-driven **planning + build** artifacts per phase (overview
  → functional spec → architecture → implementation plan → phase plans), managed via the
  `spec` skill.
- **`experiments/`** — de-risking experiments. Phase 1 = `00`–`06` + `SYNTHESIS.md`;
  Phase 2 = `round-2/` (`SP1`–`SP5` + `SYNTHESIS.md`). Each is an independent Cargo
  project with a `findings.md` + committed `results/`. `experiments/shared/` and
  `experiments/round-2/harness/` are **frozen** (read-only) shared crates.
- **`experiments/round-2/SYNTHESIS.md`** — the current Stage-3 recommendation, **adopted
  baseline decisions**, and Round-3 agenda (the closest thing to a real-app plan of
  record).

## Projects backlog — `PROJECTS.md` + `projects/`

`PROJECTS.md` (root) and the `projects/` folder are our **"save for later" list**. When
we spot an optimization, feature, or goal we want but that is **off the critical path /
not needed for MVP**, we capture it here instead of building it now or losing track of
it:

- Add a short entry to the list in **`PROJECTS.md`**.
- Write a design/goal note as **`projects/<name>.md`** (status: `Future`).

This keeps good ideas tracked without dragging them onto the critical path. It is
distinct from `specs/projects/`, which holds *active* spec-driven build planning.

## Specs are point-in-time — `specs/`

A spec under `specs/projects/` is a **planning artifact of one project, frozen when that
project ended** — not a description of how the code behaves today. An old spec may simply be
wrong now; **don't treat it as truth.** Verify against the code (or `GAPS.md`) before acting
on what it claims.

- **Don't go update finished projects' specs** to match new reality while doing unrelated
  work. They're the historical record, not documentation to maintain.
- **Exception:** the spec of the project you are *actively building* is live and **does** get
  maintained as the plan evolves — that's what the `spec` skill does.

## Known gaps — `GAPS.md`

`GAPS.md` is the **live register of currently-known holes**, each tagged with a target release
(**v0.5 / v1.0 / v2.0**) — a gap is the observed *hole*, where the `projects/*.md` note it
often links to is that hole's *work plan*. Everything in it is something we intend to close;
the only question is *when*.

- **A fixed gap is DELETED** — remove the row. Don't strike it through, don't leave it marked
  "✅ RESOLVED" in place. (Older entries still carry in-place resolved markers; that shape is
  obsolete.)
- **Add a gap when you find one.** Any behavior that diverges from the popular spreadsheet
  apps (Excel, Sheets, Numbers) earns an entry: what's missing, current behavior, root cause,
  target release.
- **Frame it "not yet — targeted at `<release>`", never "accepted limitation."** Out of scope
  for now = a gap aimed at a later release. `GAPS.md` is not where scope decisions get logged.

## Engine: we ride our IronCalc fork (fix upstream, don't hack FreeCell)

FreeCell depends on **our fork** `scosman/ironcalc`, not crates.io directly. When you hit an
IronCalc bug or missing capability, **fix it in the fork and contribute it back upstream**
(`ironcalc/IronCalc`) as a clean single-fix PR — do **not** add a compensating workaround in
FreeCell. This is the standing way of working, not a one-off.

- FreeCell's `app/Cargo.toml` pins `ironcalc`/`ironcalc_base` via `[patch.crates-io]` → the fork's
  **`freecell-fixes`** branch (the sum of our not-yet-upstreamed fixes).
- Fork branches: `main` = clean mirror of upstream; `fix/<slug>` = one branch per fix (off `main`,
  with upstream-style tests) = one clean PR; `freecell-fixes` = integration branch FreeCell builds
  against. Sync `main` from upstream periodically (rebase `fix/*` + `freecell-fixes`); expect
  incidental drift to reconcile on the FreeCell side.
- **One fix = one branch = one focused single-feature upstream PR. Never fold multiple fork fixes
  into a single `fix/` branch (or a single FreeCell phase).** Upstream wants independent,
  single-feature PRs they can review + merge in isolation; a bundled branch is not acceptable
  upstream and is harder to revert. If a FreeCell phase needs two unrelated fork capabilities, each
  gets its own `fix/<slug>` branch + PR.
- An agent can work both repos in one container (FreeCell here; fork at `/workspace/ironcalc` via
  `add_repo scosman/ironcalc`). **Full process + the per-issue loop:**
  [`specs/projects/ironcalc-upstreaming/implementation_plan.md`](specs/projects/ironcalc-upstreaming/implementation_plan.md)
  §Operating model.
- **Autonomous-run gotchas** (detail in §Operating model → "Agent operating notes"): call
  `add_repo` **upfront while the user is present** — it needs interactive approval and fails

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scosman/freecell](https://github.com/scosman/freecell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
