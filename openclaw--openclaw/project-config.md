---
trigger: always_on
description: Telegraph style. Root rules only. Read scoped `AGENTS.md` before touching a subtree.
---

# AGENTS.MD

Telegraph style. Root rules only. Read scoped `AGENTS.md` before touching a subtree.

## Start

- Repo: `https://github.com/openclaw/openclaw`
- Replies: repo-root file refs only, e.g. `extensions/telegram/src/index.ts:80`. No absolute paths, no `~/`.
- CODEOWNERS: maintenance/refactors/tests are ok. For larger behavior, product, security, or ownership-sensitive changes, get a listed owner request/review first.
- First pass: run docs list (`bin/docs-list` or `pnpm docs:list`; ignore if unavailable), then read only relevant docs/guides.
- Missing deps: run `pnpm install`, rerun once, then report first actionable error.
- Use "plugin/plugins" in docs/UI/changelog. `extensions/` remains internal workspace layout.
- Add channel/plugin/app/doc surface: update `.github/labeler.yml` and matching GitHub labels.
- New `AGENTS.md`: add sibling `CLAUDE.md` symlink to it.

## Repo Map

- Core TS: `src/`, `ui/`, `packages/`
- Bundled plugins: `extensions/`
- Plugin SDK/public contract: `src/plugin-sdk/*`
- Core channel internals: `src/channels/*`
- Plugin loader/registry/contracts: `src/plugins/*`
- Gateway protocol: `src/gateway/protocol/*`
- Docs: `docs/`
- Apps: `apps/`, `Swabble/`
- Installers served from `openclaw.ai`: sibling `../openclaw.ai`

Scoped guides:

- `extensions/AGENTS.md`: bundled plugin rules
- `src/plugin-sdk/AGENTS.md`: public SDK rules
- `src/channels/AGENTS.md`: channel core rules
- `src/plugins/AGENTS.md`: plugin loader/registry rules
- `src/gateway/AGENTS.md`, `src/gateway/protocol/AGENTS.md`: gateway/protocol rules
- `src/agents/AGENTS.md`: agent import/test perf rules
- `test/helpers/AGENTS.md`, `test/helpers/channels/AGENTS.md`: shared test helpers
- `docs/AGENTS.md`, `ui/AGENTS.md`, `scripts/AGENTS.md`: docs/UI/scripts

## Architecture

- Core must stay extension-agnostic. No core special cases for bundled plugin/provider/channel ids when manifest/registry/capability contracts can express it.
- Extensions cross into core only via `openclaw/plugin-sdk/*`, manifest metadata, injected runtime helpers, and documented local barrels (`api.ts`, `runtime-api.ts`).
- Extension production code must not import core `src/**`, `src/plugin-sdk-internal/**`, another extension's `src/**`, or relative paths outside its package.
- Core code/tests must not deep-import plugin internals (`extensions/*/src/**`, `onboard.js`). Use plugin `api.ts` / public SDK facade / generic contract.
- Extension-owned behavior stays in the extension: legacy repair, detection, onboarding, auth/provider defaults, provider tools/settings.
- Legacy config repair: prefer doctor/fix paths over startup/load-time core migrations.
- If a core test asserts extension-specific behavior, move it to the owning extension or a generic contract test.
- New seams: backwards-compatible, documented, versioned. Third-party plugins exist.
- Channels: `src/channels/**` is implementation. Plugin authors get SDK seams, not channel internals.
- Providers: core owns generic inference loop; provider plugins own provider-specific auth/catalog/runtime hooks.
- Gateway protocol changes are contract changes: additive first; incompatible needs versioning/docs/client follow-through.
- Config contract: keep exported types, schema/help, generated metadata, baselines, docs aligned. Retired public keys stay retired; compatibility belongs in raw migration/doctor paths.
- Plugin architecture direction: manifest-first control plane; targeted runtime loaders; no hidden paths around declared contracts; broad mutable registries are transitional.
- Prompt-cache rule: deterministic ordering for maps/sets/registries/plugin lists/files/network results before model/tool payloads. Preserve old transcript bytes when possible.

## Commands

- Runtime: Node 22+. Keep Node and Bun paths working.
- Install: `pnpm install` (Bun supported; keep lockfiles/patches aligned if touched).
- Dev CLI: `pnpm openclaw ...` or `pnpm dev`.
- Build: `pnpm build`
- Smart local gate: `pnpm check:changed` (scoped typecheck/lint/guards + relevant tests)
- Explain smart gate: `pnpm changed:lanes --json`
- Pre-commit view: `pnpm check:changed --staged`
- Normal full prod sweep: `pnpm check` (prod typecheck/lint/guards, no tests)
- Full tests: `pnpm test`
- Changed tests only: `pnpm test:changed`
- Extension tests: `pnpm test:extensions` or `pnpm test extensions` = all extension shards; `pnpm test extensions/<id>` = one extension lane. Heavy channels/OpenAI have dedicated shards.
- Shard timing artifact: `.artifacts/vitest-shard-timings.json`; auto-used for balanced shard ordering. Disable with `OPENCLAW_TEST_PROJECTS_TIMINGS=0`.
- Targeted tests: `pnpm test <path-or-filter> [vitest args...]`; do not call raw `vitest`.
- Coverage: `pnpm test:coverage`
- Format check/fix: `pnpm format:check` / `pnpm format`
- Typecheck:
  - `pnpm tsgo`: fastest core prod graph
  - `pnpm tsgo:prod`: core + extensions prod graphs; used by `pnpm check`
  - `pnpm check:test-types` / `pnpm tsgo:test`: all test graphs
  - `pnpm tsgo:all`: all prod + test project refs
  - Debug slices exist; do not present as normal user flow.
  - Profile: `pnpm tsgo:profile [core-test|extensions-test|--all]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openclaw/openclaw](https://github.com/openclaw/openclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
