---
trigger: always_on
description: This file applies to the entire repository.
---

# Repository Working Agreement

This file applies to the entire repository.

## Product boundary

Changes is an offline, deterministic jazz chord-progression studio. Runtime
music behavior comes from typed data, explicit theory laws, bounded algorithms,
and checked-in reviewed corpora. Do not add a model client, prompt, telemetry,
CDN, remote font, remote sample, or runtime network dependency.

The source tree is a ground-up replacement for the legacy monolithic HTML. Do
not move legacy inline JavaScript into new modules or preserve a behavior merely
because the old artifact happened to implement it.

## Read before changing code

1. Read `docs/ARCHITECTURE.md`.
2. Read the active Bead and all inherited context with `br show <id> --json`.
3. Read the relevant contract sections of `docs/REBUILD_PLAN.md`.
4. For theory/discovery work, also read `docs/THEORY_IDEA_WIZARD.md`.
5. For legacy regressions or cutover, read `docs/LEGACY_AUDIT.md`.

## Tracker workflow

- Use `br`, not `bd`, for issue state and dependencies.
- Use only `bv --robot-*` modes; never launch bare `bv`.
- Work the ready leaf tasks in dependency order:
  specification/fixtures, production implementation, then independent proof.
- Claim a leaf atomically before editing. Do not hoard multiple leaves.
- Close only after the task's named gates are green and the close reason records
  exact commands and results.
- Close a package epic only after all three child phases are complete.
- Run `br sync --flush-only` after Beads mutations. This command never runs Git.

## Source and generated files

- `src/index.html` and modules under `src/` are authoritative.
- `jazz_chord_progression_editor.html` is generated. Never hand-edit it.
- `dist/index.html` must be byte-identical to the tracked root artifact.
- The legacy root artifact may be replaced only by the guarded build after its
  commit, SHA-256, and size baseline in `docs/LEGACY_AUDIT.md` are verified.
- Keep manifests, locks, contracts, fixtures, and source tracked. Keep
  `node_modules/`, `dist/`, browser reports/caches, coverage, and temporary
  reproducibility roots ignored.

## Architecture invariants

- Preact is the only production package.
- Domain is spelling-first and uses exact rational musical time.
- Theory is pure and imports only domain. It receives content adapters by
  interface injection and never imports the compiled Atlas.
- Playback plans are immutable and shared by audio and MIDI.
- Audio owns one persistent graph and consumes only playback plans/serialized
  commands.
- UI renders selectors and dispatches application intents; it does not call
  audio, persistence, or export adapters directly.
- Only `application/document-validation.ts` may cast the opaque validated
  document brand.
- Type-only imports, re-exports, and private deep imports obey the same layer
  boundaries as runtime imports.
- Manual/Frozen pitches, source spellings, stable IDs, and exact durations are
  never silently repaired or optimized.

## Verification discipline

- Use independently authored fixtures; production output cannot certify itself.
- Every law has positive, negative/near-miss, transposition, and mutation proof.
- Every bounded search reports deterministic work/state/memory termination.
  Wall time is performance evidence, never a musical cutoff.
- Run real browser/audio/storage/download adapters wherever the active contract
  names them. Do not substitute a mock-only smoke test.
- No skipped, retried, quarantined, or silently relaxed release gate.
- Preserve detailed machine-readable diagnostics, seeds, hashes, versions,
  request logs, console/page errors, voice/listener counts, and exact diffs.
- Keep the worktree's unrelated changes intact. Do not commit, delete, move, or
  normalize another contributor's files unless the user explicitly asks.

## Foundation commands

The stable public commands are defined in `docs/ARCHITECTURE.md`. In
particular:

- Bun 1.3.14 owns package management, build, and non-browser tests.
- Playwright runs under a real supported Node 22, 24, or 26 process, never the
  Bun `node` shim.
- `bun scripts/validate-f0-contract.ts` validates the pre-build foundation
  specification without third-party packages.
- `bun run verify` is the aggregate release-facing gate once the scaffold
  exists.

## Deploying

The live site is Cloudflare Pages on the `jazzchords.org` custom domain, with
a byte-identical Vercel mirror. Both serve the tracked root artifact as
`index.html`. Every rule below was paid for by a real failure.

- Verify deployed bytes against `git show HEAD:jazz_chord_progression_editor.html`,
  never the working-tree copy. `bun run build` builds the TREE, so a tree
  carrying anyone's uncommitted edits yields an artifact reproducible from no
  commit at all. That has already shipped: the hosts served bytes no commit
  could regenerate, and comparing against the tree hid it precisely because
  the tree was what got built.
- Stage explicit paths. `git add -A` has twice swept in files that were not
  part of the change: another contributor's in-flight work, which landed it
  under an unrelated commit message and made the history lie about who wrote
  it, and a third-party MIDI file (`*.mid` is ignored now for that reason).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/jazz_chord_progression_editor_html](https://github.com/Dicklesworthstone/jazz_chord_progression_editor_html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
