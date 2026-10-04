---
trigger: always_on
description: Notes for AI agents on this repo. Read once, in full. This is the must-follow
---

# AGENTS.md

Notes for AI agents on this repo. Read once, in full. This is the must-follow
contract, not the runbook — keep it short.

## Git: Commit And Push Only When Told

- Never commit, and never push to `main` on GitHub, unless the user says to in
  the current conversation. Finishing a task, passing tests or a green build is
  not permission; ask, then wait. A previous go-ahead does not carry over.
- Every commit message is proposed to the user first, in full (subject and body),
  and is used only after the user confirms it. If the user asks for changes,
  propose the revised message and confirm again. No exceptions — this applies to
  every commit, including fix-ups, doc-only changes and version bumps.
- Never amend, rebase, force-push or reset published history; never run `git
  add -A` without showing the file list first.

## First Stops

- Testing, diagnostics, VICE workflows: `docs/TESTING.md`.
- Performance methodology + harnesses: `docs/PERFORMANCE-ANALYSIS.md`.
- Subsystem models: `docs/*-ARCHITECTURE.md` (`VIC2-`, `MACHINE-`, `SID-`,
  `DRIVE-`, `DATASETTE-`, `MEMORY-`, `CPU-`, `RETROVIBES-`).
- Screenshot-tool specifics: each tool's header comment
  (`test/commit-screenshots.mjs`, `test/demo-status.mjs`, `tools/guide-shots.mjs`,
  `tools/guide-extra-shots.mjs`, `tools/guide-dialog-shots.mjs`,
  `tools/guide-setup-shot.mjs`, `tools/guide-theme-shots.mjs`, `tools/vibes-guide-shots.mjs`,
  `tools/vibes-strip.mjs`, `tools/datasette-anim.mjs`, `tools/pick-frames.mjs`).
  `tools/guide-shots.mjs` heads the full guide-asset regeneration order.

## ROMs

The C64 + 1541 ROMs (`kernal.bin` 8K, `basic.bin` 8K, `chargen.bin` 4K,
`1541.bin` 16K) are copyrighted and NOT in this repository. The developer
supplies them in `roms/` (git-ignored); the tests read them from there via
`test/external-assets.json`, while the app takes them through its ROM setup
dialog and keeps them in browser storage (`src/roms.js`). Never commit a ROM,
embed ROM bytes in source or tests, or copy ROMs into `public/` (`public/roms/`
is git-ignored and excluded from the PWA precache for that reason).

## Workspace Hygiene

- Temp scripts + scratch output go under `investigation/` (git-ignored; create it
  on demand). At end of run, prune
  transient files (`.log`, `.txt`, `.csv`, `.bin`, `.mon`, `.png`, `.cpuprofile`,
  `.bak*`, `.DS_Store`); keep scripts (`.mjs`, `.js`, `.sh`, `.py`, `.asm`).
- Never touch `investigation/commitscreenshots/` or `demo-status-shots/` — the
  user owns those.
- No `.d64`/`.prg`/disk images in `investigation/`; reference external
  collections by real path (see Screenshot And Demo Tools). Committed test inputs
  go under `test/`.
- Never hand-edit generated `public/docs/`; edit source docs and run the build.
- `public/sitemap.xml` is a tracked `build:docs` byproduct: whenever the docs build
  updates it, that change ships with the push to `main` — never leave it behind as
  unrelated work-in-progress.

## Code And Docs

- Add tests for new features. Register every `test/*-test.js` in `test/all-test.js`;
  a CLI spec goes in `test/cli/` and registers in `test/cli/all-test.js`.
- Env switches go in `src/switches.js`, read via `switchOn('name')` — no inline
  `process?.env`.
- Don't create new `*.md` docs without an explicit request.
- No em dashes in docs prose: none in paragraphs, lists, or table cells.
  Use a comma, colon, parentheses, or a sentence split instead. The
  `## <version> — <date>` release titles in `WHATS-NEW.md` keep theirs.
- Keep user docs current in the same change (`FEATURES`, `GETTING-STARTED`,
  `COMPONENT-STATUS`, `KNOWN-ISSUES`, `TESTING`, `SPECIFICATIONS`,
  `PERFORMANCE-ANALYSIS`). Docs state current behavior, not history — remove
  fixed known-issues, don't annotate them.
- Update `docs/*-ARCHITECTURE.md` when a change alters a subsystem model,
  pipeline, timing rule, device mode, or perf design; skip it for small fixes.
- No comments referencing fix history, callers, tickets, or the current task. Fix
  inaccurate comments near what you change; add present-tense ones where needed.
- Every shipped source file gets the two-line SPDX/copyright header (`src/`,
  `docs/`, `index.html`, build config, `tools/*.mjs`); `test/` and
  `investigation/` only on request.
- A user-facing change is written up in `docs/WHATS-NEW.md` as it lands, under
  the **Next release** section at the top of the file — plain language, no
  internals, since it is a published docs page. Add that section if it is gone
  (a bump consumed it); leave it out when the change is invisible to users.
- Bump `src/version.js` (`YEAR.MONTH.FIX`) only on request. In the same commit:
  retitle **Next release** to `## <version> — <Month D, YYYY>`, write its lead
  paragraph if the release deserves one, and sync `package.json`'s `version`.
- The CLI versions separately (`cli/package.json`, semver) and is bumped only
  when the user says so, since a bump is a publish decision: a fix or feature
  landing in `cli/` is not permission, and neither is an emulator release.
- A user-facing CLI change is written up as it lands, like the emulator's, in
  `cli/README.md` under a **Next version** heading at the top of **Release
  notes** (add the heading if a bump consumed it). On a CLI version bump,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mortyeriksen/c64ready](https://github.com/mortyeriksen/c64ready) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
