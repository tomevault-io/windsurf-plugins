---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository. Read this
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repository. Read this
before making changes; more specific instructions take precedence.

## Project

dsh-feishu is a native [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)
(dsh) plugin that turns Feishu (Lark) into dsh's own surface: one Feishu chat
maps to one dsh session, the chat bot is the agent's avatar, and output
streams back as live Feishu cards.

**Core identity: DSH-native — born for dsh, not bridged to it.** The surface
targets exactly one agent (dsh) and integrates in-process; it does not bridge
external CLIs and does not reimplement agent capabilities. Three promises
follow (see README):
no bridge/capture (no CLI adapters, no tmux/screen/ANSI), full transparency
(every token/tool/question/approval streams out; the agent never does
anything to be seen), and everything-is-a-card (every dsh surface element
maps to a Feishu card).

It is built as a dsh **bundle** (an npm package whose manifest declares
`dsh.bundle.patch`) that rides on `@deepseek-ai/dsh-base`.

Work proceeds in **iterations**: each iteration ships a coherent slice of
functionality with unit tests and docs, and lands on `main`.

## Non-negotiable conventions

- **English only in code and shipped docs.** All code comments, identifiers,
  README, `docs/`, `AGENTS.md`, and the CHANGELOG are written in English.
  Chinese documentation is provided later as separate files (e.g.
  `README.zh-CN.md`); never mix languages in one file.
- **No machine-specific details in tracked docs.** Everything git tracks
  becomes public once the repository is open-sourced. Never commit absolute
  developer-machine paths (`/home/<user>/…`), ambient environment values
  observed on one machine (e.g. a harness-exported `DSH_HOME`), or other
  local-only state into README, `docs/`, `AGENTS.md`, or the CHANGELOG.
  Path examples are `$(pwd)`-anchored (`_dev/…` is git-ignored local state)
  or generic placeholders; environment anecdotes are phrased so any
  contributor's machine applies.
- **Git-tracked docs are public — no internal pointers.** Everything under
  `README`, `docs/`, `AGENTS.md`, the CHANGELOG is visible to anyone once
  pushed. Never reference internal-only artifacts (git-ignored `_dev/`
  files, local analyses, private reports, ambient environment values) or
  point readers at them ("see the internal report" is a dead link to
  everyone else). Every tracked doc must stand alone and be self-explanatory
  to an outside reader.
- **Every feature module ships unit tests.** A new module in `src/` must come
  with a co-located test in `tests/` covering its behavior. Fixing a bug
  first adds a failing test. No untested feature lands.
- **Write docs promptly after a feature.** Completing a feature updates the
  relevant `docs/` page(s) and the CHANGELOG in the same change. No feature
  lands without its documentation.
- **Docs move with their feature — never in a later PR.** Every change that
  touches behavior, commands, UX, setup, or architecture updates the
  corresponding doc in the SAME PR (map: `docs/development.md` →
  "Documentation map"). A PR that changes behavior without touching its
  mapped doc is incomplete; call out "no docs change needed" explicitly in
  the PR body.
- **README.md / README.zh.md are maintainer-gated.** They are the public
  face of the project: never commit ANY README edit — wording, badges,
  structure, links — without explicit maintainer review first. Other docs
  may be committed directly; README always needs review, so put README
  edits in their own commit that can be reviewed/dropped independently.
- **Feishu permissions manifest, kept in sync in the same change.** Any
  feature that needs a new Feishu scope, event, or card callback updates
  `src/setup/feishu-manifest.json` — the single source of truth for the
  quick-setup automation (`pnpm run setup:feishu`), its manual fallback, and
  `docs/feishu-setup.md` — IN THE SAME change. The setup tool grants exactly
  what that file lists; an unlisted scope is a bot that silently cannot do
  the feature. Example: `/export` (file messages) added `im:resource`.
- **Registrations are effects.** Every contribution goes through `ctx.on()` /
  `ctx.effect()`; a registry's `register()` returns a disposer, and tests
  verify disposal where a registry is involved.
- **Optional services use `ctx.get(name)`.** Reserve `ctx.<name>` for
  declared injections; feature-detect and degrade loudly when a service is
  absent (the dsh runtime is `0.1.0-rc` and its surfaces can move).
- **Type-only imports for `@deepseek-ai/*`.** Runtime dependencies are kept
  minimal; harness packages are peer/dev dependencies used for types only.
  Empty type imports carry Context merges (e.g. `import type {} from
  '@deepseek-ai/dsh-commands'`).
- **Misconfiguration fails loud.** Never silently skip a missing referent;
  log what is missing and why.
- **No truncation without user confirmation.** Never cut user-visible
  content (card size, list length, output length, collapsed sequences,
  details views) as a silent default. Physical platform limits (the Feishu
  ~109 KB card cap) are the only exception, and even those must be raised

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PGZXB/dsh-feishu](https://github.com/PGZXB/dsh-feishu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
