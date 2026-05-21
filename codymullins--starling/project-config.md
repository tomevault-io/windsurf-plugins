---
trigger: always_on
description: This file is the **first thing an implementation agent should read** when
---

# Agents — rules of engagement

This file is the **first thing an implementation agent should read** when
opening this repo. It explains how multiple agents share work without
stepping on each other, and how a single agent can stop mid-task and have a
later session resume cleanly.

## TL;DR

```bash
# 1. Orient
cat AGENTS.md                            # this file
less browser-plan/13_MILESTONES.md       # where we are
less tasks/INDEX.md                      # what's available

# 2. Claim something unblocked
./tasks/lib/claim.sh claim wp:M1-03-dom-core "agent-claude-<your-handle>"

# 3. Read the package file and start working on main
less tasks/M1/wp-M1-03-dom-core.md

# 4. Commit often, with the wp id in the subject:
#    "wp:M1-03 — Node hierarchy + tests"

# 5. Build + test before completing
dotnet build && dotnet test

# 6. Mark complete:
./tasks/lib/claim.sh complete wp:M1-03-dom-core
```

**All work happens on `main`.** This repo doesn't use a per-package branch
workflow — agents commit directly to main. (If a remote with PR review is
ever wired up, the optional `in_review` state in `tasks/SCHEMA.md` is
available for it; today it's unused.)

If you have to stop early, **leave a handoff log entry** in the task file
and either keep the claim (you'll resume) or release it
(`./tasks/lib/claim.sh release wp:…`). Either way: commit so the next
agent sees the state.

## The contract

| You can rely on | You must do |
|---|---|
| One file per work package under `tasks/M*/wp-*.md` | Touch only your claimed package's file (plus your code changes) |
| `tasks/INDEX.md` reflects the current status | Update INDEX.md when you change a status |
| Work happens on `main`; commits carry the wp id in the subject | Prefix commit messages with `wp:<id> —` so history is greppable |
| Dependencies are explicit in `depends_on` | Don't start a package whose deps aren't `complete` |
| Stale claims age out at 72 h | Add a handoff log entry every session, even if you're "still going" |

## Repo map

```
starling/
├── AGENTS.md                  ← you are here
├── README.md                  ← human-facing intro
├── browser-plan/              ← the design (immutable except by deliberate edit)
│   ├── 00_INDEX.md            ← start here for design context
│   ├── 13_MILESTONES.md       ← what milestone we're in
│   └── 14_AGENT_TASKS.md      ← authoritative package catalog
├── tasks/                     ← work coordination (this repo's queue)
│   ├── README.md              ← detailed workflow
│   ├── SCHEMA.md              ← frontmatter contract
│   ├── INDEX.md               ← current status of all packages
│   ├── lib/claim.sh           ← atomic claim/release helper
│   └── M<n>/wp-*.md           ← one file per work package
├── src/                       ← engine + Headless CLI + Avalonia Gui (win/mac/linux)
├── Starling.AppHost/          ← Aspire AppHost (orchestrates Gui + Headless)
├── Starling.ServiceDefaults/  ← Aspire OTel + health-check shared bootstrap
├── tests/                     ← one xUnit project per src/ module + E2E
├── bench/Starling.Bench/      ← BenchmarkDotNet
└── testdata/                  ← fixtures + golden PNGs + WPT subsets
```

## Build + test (must be green before merge)

```bash
dotnet --version            # expect 10.0.x
dotnet restore
dotnet build -c Debug
dotnet test  -c Debug
```

If `dotnet build` errors with permission-denied apphost deletions in a
sandbox or container, pass `-p:UseAppHost=false`. This is a sandbox quirk
only — CI runs without the flag.

## Spec coverage & the bug-fix workflow

Most bugs in this engine are a spec compliance gap wearing a disguise. Before
you "just fix it", run this loop. It is **not optional** — a fix without a
test is not done.

1. **Check whether the behavior is covered by a spec.** Identify the spec and
   section the buggy behavior belongs to (CSS 2.x / Sizing / Flexbox / DOM /
   HTML / URL …). `tasks/SPEC_COVERAGE.md` is the map of where we stand;
   `tasks/SPEC_CATALOG.md` is the upstream list of what each spec defines.
2. **Check whether we already have a test for it.** Grep the `[Spec]` traits:
   `dotnet test --filter "TestCategory~Spec:<spec-id>"`, or grep the source
   for `[Spec("<spec-id>"`. If a `[PendingFact]` already documents this gap,
   you're about to promote it — don't write a duplicate.
3. **Reproduce with a failing test first.** When you report or start a bug,
   write the test that demonstrates the failure *before* touching the fix.
   Tag it `[Spec(id, url, section)]`. If you can't make it pass yet, commit it
   as `[PendingFact]` with a `trackingWp`; if you're fixing it now, watch it
   go red, then green.
4. **Every fix ships with a test.** The test that reproduced the bug becomes
   the regression test — promote it to `[SpecFact]` in the same change as the
   fix. Put it where the exercised code is tested (e.g. layout behavior →
   `Starling.Layout.Tests`), not in a catch-all. No "fixed it, trust me" —
   the diff must contain a test that fails without your change.

There is **one** way spec tests are tracked: real test methods tagged with
`[Spec]` + `[SpecFact]`/`[PendingFact]` from `Starling.Spec.Common`. There is
no stub generation. See `tests/Starling.Spec.Common/README.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codymullins/starling](https://github.com/codymullins/starling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
