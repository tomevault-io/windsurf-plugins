---
trigger: always_on
description: `skill/` is the source of truth for the Impeccable skill: `SKILL.src.md`, `reference/`, `scripts/`, and `agents/`. Build logic lives in `scripts/`, with provider configs in `scripts/lib/transformers/`. The CLI and anti-pattern detector live in `cli/`, the browser extension in `extension/`, the Astro website in `site/`, Cloudflare Pages Functions in `functions/`, and regression coverage in `tests/` with fixtures under `tests/fixtures/`. `dist/` and `build/` are generated and gitignored. The root 
---

# Repository Guidelines

## Project Structure & Module Organization

`skill/` is the source of truth for the Impeccable skill: `SKILL.src.md`, `reference/`, `scripts/`, and `agents/`. Build logic lives in `scripts/`, with provider configs in `scripts/lib/transformers/`. The CLI and anti-pattern detector live in `cli/`, the browser extension in `extension/`, the Astro website in `site/`, Cloudflare Pages Functions in `functions/`, and regression coverage in `tests/` with fixtures under `tests/fixtures/`. `dist/` and `build/` are generated and gitignored. The root harness folders (`.agents/`, `.claude/`, `.cursor/`, etc.) and `plugin/` are generated distribution artifacts that are tracked for direct repo installs, not hand-authored source.

## Build, Test, and Development Commands

- `bun run dev` - start the local Bun server.
- `bun run build` - source-first build: regenerate `dist/`, derived site assets, and validation output without syncing tracked harness folders.
- `bun run build:release` - release/distribution build: run the full build and sync tracked root harness folders plus `plugin/`.
- `bun run rebuild` - clean and rebuild everything from scratch without syncing tracked harness folders.
- `bun run rebuild:release` - clean and rebuild everything, including tracked harness output sync.
- `bun test tests/build.test.js` - run a focused Bun test.
- `bun run test` - run the full Bun + Node test suite.
- `bun run test:live-e2e` - opt-in live-mode E2E against framework fixtures (~2 min; needs `npx playwright install chromium` once).
- `bun run test:skill-behavior` - opt-in LLM-backed checks that the SKILL.md Setup flow actually drives the agent (~5 min; runs claude-sonnet-4-6 / gpt-5.5 / gemini-3.1-flash-lite, roughly $0.50-1.50 per run on the production-tier models, needs `.env` with provider keys).
- `bun run build:browser` / `bun run build:extension` - rebuild browser-specific bundles.

Run `bun run build` after changing anything in `skill/`, transformer code, or user-facing counts. It validates the generated distribution under `dist/` without touching tracked root harness outputs. Use `bun run build:release` only when intentionally refreshing generated provider permutations for release/main-sync or build-system work.

## Generated Provider Output Policy

The root harness folders (`.agents/skills/`, `.claude/skills/`, `.cursor/skills/`, `.gemini/skills/`, `.github/skills/`, `.kiro/skills/`, `.opencode/skills/`, `.pi/skills/`, `.qoder/skills/`, `.rovodev/skills/`, `.trae*/skills/`) and `plugin/` stay tracked so `main` remains installable for direct GitHub, `npx skills`, and submodule users. They are still generated artifacts.

Normal development should be source-first: stage changes in `skill/`, `scripts/`, `cli/`, `site/`, `extension/`, `functions/`, and `tests/`; leave generated harness churn unstaged unless the user asked for it. After source changes land on `main`, `.github/workflows/sync-generated-output.yml` runs `bun run build:release` and commits generated provider output directly back to `main`. Treat generated harness diffs as release artifacts and keep them out of feature PRs unless they are the point of the PR.

## Sandbox gotchas for Codex agents

Some repo workflows need to run outside the sandbox in the desktop app:

- GitHub SSH operations that depend on the 1Password SSH agent, such as `gh pr checkout`, may fail in the sandbox with `sign_and_send_pubkey` or no 1Password approval prompt. Rerun them outside the sandbox instead of falling back to unrelated workarounds.
- `bun run build:release` rewrites committed harness directories such as `.agents/skills/`. In the sandbox, Bun can hit filesystem errors while removing/recreating those trees (for example `EFAULT` on `.agents/skills`). Rerun the release build outside the sandbox before treating it as a real build failure.
- Puppeteer/headless-Chrome tests, especially `node --test tests/detect-antipatterns-browser.test.mjs` and the browser portion of `bun run test`, can hang in the sandbox while launching Chrome. Run them outside the sandbox for authoritative results.
- The jsdom fixture suite is intentionally run with Node, not Bun: use `node --test tests/detect-antipatterns-fixtures.test.mjs` or the `bun run test` script. A direct `bun test tests/detect-antipatterns-fixtures.test.mjs` can time out and is not the supported signal.

## Coding Style & Naming Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunotrolo/ImpeccableStyle](https://github.com/brunotrolo/ImpeccableStyle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
