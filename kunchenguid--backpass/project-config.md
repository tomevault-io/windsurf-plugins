---
trigger: always_on
description: backpass is an npm CLI that runs a "backward pass" over a repo's agent memory files: it
---

# Project agent memory

backpass is an npm CLI that runs a "backward pass" over a repo's agent memory files: it
discovers past agent-session transcripts tied to the repo, analyzes them, and proposes
evidence-backed edits to `AGENTS.md` / `CLAUDE.md` under a token budget.

## Orientation

- `README.md` documents the user-facing surface; `src/cli.js` is the authoritative flag list.
- The pipeline is one stage per module, in order: `src/discovery/` -> `src/sample.js` (cap) -> `src/distill.js` ->
  `src/analyze.js` -> `src/fold.js` -> `src/synthesize.js` (with `src/workspace.js` + `src/diff.js`) ->
  `src/proposal.js` -> `src/apply/`. Each module's header comment explains its role; read those
  before changing a stage.
- **User-facing step names are the training-loop terms**, not the module names: discover =
  "collect samples", analyze = "calculate loss", fold = "aggregate gradients", synthesize =
  "gradient descent" (`STAGE_LABELS` in `src/tui/render.js`). Internal keys, event names, and
  the typed subcommands (`scan`, `analyze`, `propose`) keep their short names; only what the
  user reads changes. Never introduce a multi-word subcommand.
- Zero runtime dependencies, ESM, no build step. Node >= 22.5 (for `node:sqlite`).
- pnpm is the package manager; `pnpm run check` runs lint, format:check, typecheck, and
  tests. All tests are offline and use fixtures under `test/fixtures/`. Supply-chain
  settings (release-age cooldown, build-script deny) live in `pnpm-workspace.yaml`.
- Releases are automated by release-please (`.github/workflows/release-please.yml`,
  npm trusted publishing). Never hand-edit `CHANGELOG.md` or
  `.release-please-manifest.json`; CI guards reject PRs that touch them.
- `.github/workflows/no-mistakes-required.yml` is a thin caller of the shared
  `kunchenguid/no-mistakes/.github/actions/require-no-mistakes` composite action, pinned
  to an immutable commit SHA and never `@main`. Enforcement logic and tests live upstream
  in the no-mistakes repository; change enforcement there, never by copying it locally,
  and bump the pin in a deliberate separate PR. This repo still owns the `on:`,
  `concurrency`, `permissions`, job name, and author-exemption `if:`. The action binds the
  attestation to the head SHA, so a PR whose body no-mistakes did not rewrite for the
  current head goes red by contract: push through `git push no-mistakes`.

## Sharp edges

- **Transcript formats are undocumented and drift.** Every adapter in
  `src/discovery/adapters/` is pinned by a golden fixture in `test/fixtures/`. When a
  harness changes its on-disk shape, fix the adapter and update its fixture together.
  Adapters must stay fail-soft: an unreadable store warns and is skipped, never throws.
- **The live progress view is an enhancement layer, never a dependency.** Pipeline stages
  emit events through `src/progress.js`; `src/tui/` renders them on stderr during the
  default run and buffers/replays the plain logger lines on teardown. Every path must
  behave identically when it is inactive (non-TTY, CI, NO_COLOR, --quiet, --json) - plain
  line output on stderr and clean stdout are the contract. Rendering logic stays pure
  (`src/tui/render.js`) so it is testable as text.
- **`src/apply/writer.js` is the only module that writes to the repo.** Keep it that way -
  every other stage is read-only analysis, which is what makes a run safe to interrupt.
  Bootstrap (`src/commands/bootstrap.js`, a repo with no memory file) is the one run that
  writes without the apply gate, and it only ever creates files, never overwrites.
- **Synthesis edits natively, in a staging copy, never by describing text.** The agent
  gets `--approve-all` with `cwd` = `.backpass/synthesis/` (`prepareWorkspace`), which holds
  only the memory file and the skills dir; backpass measures the copy (`measureWorkspace`,
  `anchoredHunks`) and the agent annotates the measured changes by id in a second turn of
  the same session. The model never supplies `find` text - every hunk is cut from the raw
  file, widened until unique, so "find text does not appear" cannot recur. Never pass
  `approveAll` with the repo as `cwd`; the repo is fingerprinted and a harness that
  writes there fails the run loudly.
- **Skills only count if a harness loads them.** Extractions target `.agents/skills` with
  `.claude/skills -> ../.agents/skills` as a symlink (`ensureSkillsLayout` in
  `src/skills.js`, run at write time); a bare `skills/` dir is never auto-detected and a
  real `.claude/skills` directory is warned about, never replaced.
- **Memory resolution is pointer-aware** (`resolveMemoryFiles` in `src/memory.js`): the
  first configured file is canonical, a `@AGENTS.md`-only CLAUDE.md is a pointer, and a
  second full file is warned about, never silently ignored or double-written.
- **Never trust model-reported numbers.** Token deltas and budget projections are measured
  in `src/proposal.js` from the actual text; the synthesis model's own figures are ignored.
  Usage accounting comes from acpx's `[acpx] tokens:` stderr line, which acpx prints
  when the ACP adapter returns usage (codex, claude do; pi-acp does not), with one
  harness-store fallback: pi's per-turn usage is read back from its own session file,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/backpass](https://github.com/kunchenguid/backpass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
