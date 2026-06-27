---
trigger: always_on
description: You are likely Claude / Codex / Copilot / Cursor working inside the **VibeChard**
---

# AGENTS.md — Notes for AI coding agents working on this repo

You are likely Claude / Codex / Copilot / Cursor working inside the **VibeChard**
repository. Read this before making changes.

## What this repo is

A Swift CLI named `vch` that gives Apple developers **isolated parallel
worktrees** so multiple AI agents can build / test / iterate on the same
Apple project without trampling each other's `DerivedData`, `ModuleCache`,
SwiftPM caches, or simulator state.

Locked v1 plan, with rationale and acceptance criteria, lives in agent memory
under `/memories/repo/vibechard-plan.md`. Read it for the *original Q-decision
rationale* if you have access — but treat it as a historical artifact, not a
current-state spec. The plan itself carries a STALE WARNING; per Engineering
discipline #1 below, when the plan and the source tree disagree, **the source
tree wins.**

## Hard rules (do not break)

1. **Apple-only.** Do not add Linux / Windows support, or any "cross-platform
   compatibility" abstractions. The whole point is depth, not breadth.
2. **BYO Agent.** Do not import any AI SDK, do not call any AI HTTP API,
   do not bake in support for a specific agent (Claude/Codex/Copilot/etc.).
   The only agent integration point is the generic `--exec "<cmd>"` flag.
3. **No telemetry.** No network calls. No analytics. Ever.
4. **Two dependencies only:** `swift-argument-parser` and `swift-system`.
   Justify any new dep in the PR description; default answer is "no".
5. **Three targets, fixed:** `VibeChardCore` (library), `vch` (CLI),
   `vch-xcodebuild-shim` (tiny standalone). Do not split further.
6. **The shim must stay minimal.** `Sources/vch-xcodebuild-shim/` cannot
   import `VibeChardCore` or any third-party module. Every xcodebuild call
   from inside an isolated worktree forks this binary; cold start matters.
7. **No config files in v1.** All state goes into per-worktree
   `.vch/state.json`. No `~/.vchrc`, no `.vch.toml`, no env-var-based config
   knobs beyond the documented `VCH_*` set.
8. **Reserved subcommand names.** `vch new <name>` rejects names that
   collide with an existing subcommand or alias, or that start with `-`.
   The authoritative list lives in
   `Sources/VibeChardCore/Domain/TaskName.swift` — read the code, not
   this file, when you need the current set.
9. **Don't touch the user's `~/Library/Developer/` outside the
   simulator-clone exception.** Every byte vch writes directly must
   land inside the worktree's `.vch/` or `.agent-build/`. The single
   permitted footprint under `~/Library/Developer/` is
   `~/Library/Developer/CoreSimulator/Devices/<UDID>/`, because
   `xcrun simctl` does not accept an alternate storage root — there
   is no `--data-dir` or equivalent flag, so the OS owns the layout.

   Within that exception, the *kinds* of vch-managed devices and
   their lifecycle rules (per-task vs. shared, who creates / destroys
   them) are defined in code — see `SimulatorService` and the
   `WarmTemplate*` types under `Domain/`. The load-bearing principle
   is that **the user owns the lifecycle of any shared resource**:
   vch may auto-create and auto-reap state that belongs to a single
   task, but never state that is shared across tasks. New device
   kinds are fine when the trade-off is justified in the PR
   description. `ci.yml` smoke-checks the shim's `xcrun -f xcodebuild`
   exec path on every push to ensure no other directories under
   `~/Library/Developer/` ever get written to by vch.
10. **Multi-language README sync.** Substantive changes to `README.md`
    (features, commands, install steps, rules) must be mirrored to
    `README.ja.md`, `README.ko.md`, `README.zh-CN.md`, `README.zh-TW.md`
    in the same PR. If you cannot translate confidently, add
    `<!-- TODO: sync with README.md -->` at the top of the affected file
    rather than letting it silently drift. Pure typo / link / formatting
    fixes are exempt.

    **Scope: README mainline only.** Extension reference docs under
    `docs/` (e.g. `docs/cookbook.md`, `docs/commands.md`) are
    English-only — the single source of truth lives in English and
    is not mirrored to other locales. The 5-locale READMEs may link
    *to* `docs/...`; the link itself counts as a substantive change
    and must be present in all 5 READMEs, but the target document
    does not need a translated counterpart. The point of this carve-out
    is to keep README a high-quality first-impression artefact in 5
    languages without paying a 5× tax on every advanced-recipe edit.

## Engineering discipline

These are workflow expectations, not project policies. They live here
because past sessions repeatedly rediscovered them the hard way.

1. **Source code is the source of truth.** When evaluating whether a
   change is feasible, necessary, or correct, **read the code**.
   Plan documents (`/memories/repo/vibechard-plan.md`, milestone
   result notes, even your own previous CHANGELOG entries) drift
   between sessions. Treat them as hypotheses to verify, not facts
   to act on. If the plan and the code disagree, the code wins.
2. **Add tests when behaviour changes.** Any meaningful logic change
   to `VibeChardCore` or `vch` deserves a unit test in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Maples7/VibeChard](https://github.com/Maples7/VibeChard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
