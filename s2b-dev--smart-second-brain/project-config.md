---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Project

Smart Second Brain is an Obsidian plugin (`smart-second-brain`) that turns the vault into an AI-assisted second brain: chat-with-your-notes via a RAG pipeline, a smart graph view, and provider-agnostic LLM support (OpenAI, Ollama, oMLX, Anthropic, OpenRouter, OpenAI-compatible endpoints). Runs on desktop and mobile (`isDesktopOnly: false`). Stack: Svelte 5 (runes) + TypeScript + Vite + LangChain/LangGraph, with Tailwind for styles, Biome for lint/format, Vitest for tests, and Bun as the package manager.

## Contributing

Human-facing contribution rules (branch model, PR expectations, the AI-assistance policy,
provider and skill recipes) live in [CONTRIBUTING.md](CONTRIBUTING.md). CI (`.github/workflows/ci.yml`)
runs the Biome format and lint checks, `bun run check`, `bun run test`, and `bun run build` on every pull request; a
change that fails any of those locally will fail there too.

**Disclose AI involvement; never hide it.** Most of this repo is written by coding agents
working from the maintainer's briefs, and the history should say so:
- Commits made from Claude Code carry a `Co-Authored-By: Claude <noreply@anthropic.com>`
  trailer, configured in `.claude/settings.json` (it overrides the user-level setting). Keep
  the trailer. If you commit through another agent, add a trailer naming *that* agent instead;
  never attribute a change to an agent that did not make it.
- Every PR body follows `.github/PULL_REQUEST_TEMPLATE.md`, including the italic
  `_AI assistance: ..._` line under "How I tested it". Fill it in honestly in one sentence: which agent wrote the
  change, from what brief, and what the human reviewed and tested. Do not leave the line out.
  "none" is the right answer only for a change no agent touched, which a change you are making
  is not.
- Replies to review-bot findings are written by whoever is driving the loop, agent or human;
  that is fine. Replies to a *person's* review are the maintainer's to write.

## Commands

Use `bun` (not npm/yarn). The lockfile is `bun.lock`.

- `bun run dev` — Vite watch build to `build/smart-second-brain/` (development).
- `bun run build` — production build to `build/prod`.
- `bun run check` — `svelte-check` over both `tsconfig.json` (src) and `tsconfig.test.json` (tests + integration). **Run after each implementation.** `check:src` / `check:test` run one pass each.
- `bun run format` — Biome formatter (writes) over `src`, `test`, `integration`. **Run after each implementation.**
- `bun run lint` — Biome linter, safe autofixes only, same three dirs.
- `bun run lint:unsafe` — adds `--unsafe`. Review the diff afterwards; these fixes can
  change behaviour rather than just style. Two real examples from this repo: `delete obj[k]`
  → `obj[k] = undefined` (leaves the key present, so a rename stops being a rename), and
  `x && x.f()` → `x?.f()` where `x` is a non-nullable string (turns an emptiness check into
  a nullishness one). Prefer fixing by hand when the rule touches control flow.

Tests are type-checked, not just formatted: `tsconfig.json` covers only `src/**`, so
`tsconfig.test.json` extends it to `test/**` and `integration/**` (it stays separate so test
files can never influence the production build). This is not cosmetic — while the suites went
unchecked, a `resolveSegments` call kept passing a removed argument, silently returned `[]`,
and made four assertions pass vacuously.

Two Biome settings exist for the test tree, both in `biome.json` (which is strict JSON and
rejects comments, hence the note here):
- `files.ignore` excludes `integration/S2B Test Vault/**` — third-party plugin bundles,
  themes, and Obsidian's own config, which Obsidian rewrites on every launch. Never format it.
- An `overrides` entry disables `style/noNonNullAssertion` for `test/**` and `integration/**`.
  `expect(store.getAgent(id)!.name)` is the point in a test: if the lookup fails, the test
  should throw. The rule stays on for `src/`.
- `bun run test` — Vitest unit tests (single run). `bun run test:watch` for watch mode. `bun run test:coverage` for coverage.
- `bun run test -- <pattern>` — single file/pattern, e.g. `bun run test -- test/providers/openai.test.ts`.
- `bun run test:integration` — end-to-end tests against a live Obsidian instance (see Integration tests below).
- `bun run setup-vault` — symlinks `build/smart-second-brain/` into `integration/S2B Test Vault/.obsidian/plugins/`. Run once after the first build.

There is no separate `bun install` step needed beyond what `bun.lock` records; do **not** edit `package.json` versions manually without re-locking.

## Documentation lives on the site, not in the README

User-facing documentation lives in the **`s2b-dev/site`** repo (checked out at
`../site`), published at `smartsecondbrain.dev`. The README here is deliberately
minimal — pitch, install, development setup, links. Do not add feature lists,
provider tables, or FAQ content back into it; that duplication is what caused
the README to drift out of date.

**Before cutting a release**, refresh the site's enumerable facts — it is not
automated. The checklist lives in `../site/CLAUDE.md` under "Before a plugin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [s2b-dev/smart-second-brain](https://github.com/s2b-dev/smart-second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
