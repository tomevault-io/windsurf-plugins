---
trigger: always_on
description: enables every non-stylistic category at `error`: a11y, complexity,
---

# crew — Contributor Guide

This file is the briefing for anyone (human or AI) working on `crew`.
Read it before making changes.

`crew` is a **package manager for Agent Skills** specified by `PRD.md`.
The PRD is the contract; this document is about how we implement it.

---

## Working agreement: specification changes

If you are asked to make a change that is **not specific to our
implementation** — that is, a change that any conformant implementation
of the spec would also need to make — **update `PRD.md` first**.

Examples of what triggers a PRD update:

- A new command, flag, or subcommand.
- A new error name or a change to an exit code.
- A change to the marker format, state schema, or config schema.
- A change to the reference grammar or resolution precedence.
- A change to a default (tap URL, autoupdate interval, lock timeout, …).
- A change to the hash algorithm or what it covers.
- Adding, removing, or renaming an agent adapter (the set in §7.2).
- A change that shifts any of the §18 conformance criteria.

Examples of what does **not** need a PRD update:

- Internal refactors, file moves, renamed internal functions.
- Test additions or helper tweaks.
- Dependency bumps that don't change behavior.
- Bug fixes that make behavior match the existing spec.
- Performance improvements within existing guarantees.

When in doubt: if an external observer running the conformance suite
could notice the change, update the PRD. The PRD is what a second
implementation would read to build the same tool — keep it accurate.

After updating the PRD, update `src/` and `tests/` to match. Don't
leave a PRD change unimplemented across a commit.

---

## Working agreement: user-facing surfaces

If a change affects user-visible behavior — a new command, a new
flag, a change to install/update/tap semantics, a new way for a
tap to be structured, a renamed default, anything a user of the
CLI would notice — also check whether these user-facing surfaces
describe the thing you just changed, and keep them in sync:

- `site/` — the landing page at `crew.logic.inc`. Sections most
  likely to go stale: `Commands.tsx` (command reference),
  `HowItWorks.tsx` (what the flow looks like), `Taps.tsx` (how
  taps are structured), `Faq.tsx` (common questions), and the
  hero terminal demo in `Hero.tsx`.
- `README.md` — the GitHub front page. It mirrors the site's
  content journey; the "What is Crew?", "How does it work?", and
  FAQ sections are the usual suspects for drift.

You don't need to update every mention of a thing — only update
what's now *wrong* or *misleadingly incomplete*. If the site's
ASCII diagram shows a flat layout and you just added a nested
layout option, the diagram isn't wrong (flat still works) but the
surrounding prose might now overclaim that "only the top level is
indexed." Fix the prose; leave the diagram.

When reviewing a PR, flag changes that touch `src/` behavior but
don't update `site/` or `README.md` — ask whether those surfaces
still describe the thing accurately.

---

## Architecture

### Runtime and shape

- **Language**: TypeScript, strict mode (`tsconfig.json`).
- **Runtime**: [Bun](https://bun.sh) only. We ship bundled native
  macOS and Linux executables produced by `bun build --compile`.
- **Minimal host dependencies**: the `dist/crew` binary carries the Bun
  runtime. The host-level requirements are `git`, plus `launchctl` on
  macOS or `systemctl --user` on Linux for autoupdate (mandated by
  PRD §17.1).
- **Libraries**: use well-established libraries for solved problems,
  hand-roll anything that's trivial or spec-prescribed. Current
  dependencies:
  - [`js-yaml`](https://github.com/nodeca/js-yaml) — YAML parse/write.
    Wrapped by `src/yaml/parse.ts` to keep exactly one import site.
  - [`proper-lockfile`](https://github.com/moxystudio/node-proper-lockfile)
    — cross-process advisory locking. Used by `src/state/lock.ts`.
  - [`yargs`](https://github.com/yargs/yargs) — argv parser. Used by
    `src/cli/args.ts`, configured as a pure parser (no auto-help, no
    auto-exit). The rest of the CLI machinery (dispatch, output
    formatting, error mapping) is still our own.

  Things we **don't** pull in as libraries and why:
  - **SHA-256 / content hash** — Node's `crypto.createHash` is in stdlib,
    and PRD §12.1 prescribes the exact algorithm; a third-party library
    would only re-wrap Node crypto.
  - **launchd plist** — it's ~15 lines of static XML template. A general
    plist library is overkill.
  - **git** — we shell out, per PRD §17.1. `isomorphic-git` would bundle
    git into the binary and duplicate what the user already has.

  When adding functionality, use the well-established library for the job
  unless the above reasons apply. "Well-established" means widely used,
  actively maintained, and not ad-hoc — if you'd be the first to try a
  library in production, write the code instead.

### Directory layout

```
src/
├── index.ts              # entry point — reads argv, calls runCli
├── cli/                  # argv parser, dispatcher, output formatter
├── commands/             # one file per subcommand; complex commands are directories
├── core/                 # types, errors, paths, version
├── config/               # config.yaml read/write + defaults

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [with-logic/crew](https://github.com/with-logic/crew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
