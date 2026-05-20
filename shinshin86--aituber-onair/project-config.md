---
trigger: always_on
description: - Monorepo via npm workspaces under `packages/*` with `src`, `tests`, `images`, `dist`, and `examples` per package.
---

# Repository Guidelines

## Project Structure & Modules
- Monorepo via npm workspaces under `packages/*` with `src`, `tests`, `images`, `dist`, and `examples` per package.
- Key packages: `core` (integration/orchestration), `chat` (LLM providers), `voice` (TTS engines), `manneri` (conversation pattern detection), `bushitsu-client` (WebSocket + React hooks), `kizuna` (relationship/points).
- Each package is usable independently; import from package entry points, not `dist`.

## Architecture Overview (from CLAUDE.md)
- Event‑driven core with strong typing: AITuberOnAirCore (orchestrator), ChatProcessor (LLM flow), MemoryManager (auto‑summarization to avoid token bloat).
- Chat abstracts OpenAI/Claude/Gemini under a unified interface; Voice supports multiple TTS engines with emotion cues; packages avoid tight coupling.
- Cross‑runtime focus (browser/Node/Bun/Deno). Optional Node audio: `speaker` or `play-sound`.

## Build, Test, and Development
- Install: `npm ci`
- Build all: `npm run build --workspaces` (CI builds in dependency order on Node 20)
- Build one: `npm -w @aituber-onair/chat run build`
- Test all: `npm run test --workspaces` (watch: `npm -w @aituber-onair/core run test:watch`)
- Lint/Format: `npm run lint --workspaces` • `npm run fmt --workspaces` (check: `fmt:check`)
- Type check (per pkg): `npm -w <name> run typecheck`
- After any code fix/change, always run `fmt`, `build`, then `test` for the affected scope before completing the task.

## Definition of Done (DoD)
- Before push/PR, always run the following repository-wide commands in this order and ensure all pass:
  - `npm run fmt`
  - `npm run lint`
  - `npm run test`
  - `npm run build`

## Coding Style & Naming
- TypeScript + Biome (2‑space indent, single quotes, 80‑char width). Code/comments in English.
- Use barrel exports (`index.ts`) to define public API; minimize external deps.
- Tests live in `tests` with `*.test.ts` naming.

## Testing Guidelines
- Framework: Vitest (jsdom where browser APIs are needed).
- Prefer AAA pattern and focused unit tests; include environment differences (browser vs Node).
- Coverage: `npm -w <name> run test:coverage`.

## Starter / CLI Smoke Testing
- When smoke testing generated starter projects, create the project outside the
  repository or in a package-local ignored `tmp/` directory. Prefer an
  OS-managed temporary directory for dependency-isolation checks.
- Use a dedicated npm cache for each starter smoke test via `npm_config_cache`.
  Avoid reusing the repository npm cache or a cache nested inside an existing npm
  workspace when validating fresh installs.
- Keep generated starter paths and cache paths generic in notes and commits; do
  not record machine-specific absolute paths, user names, or other local
  environment details.
- For `create-aituber-onair` template validation, test the packaged CLI flow
  from a local tarball, then run the generated app's normal user-facing steps:
  `npm install`, `npm run build`, and `npm run dev` when applicable.
- If a fresh starter install fails with native or optional dependency version
  mismatches, first retry in an isolated temporary directory with a fresh npm
  cache before changing template dependencies. Monorepo/workspace context and
  stale caches can affect GitHub dependencies and optional binary packages.

## Commit & Pull Requests
- Conventional Commits: `feat(core): ...`, `fix(manneri): ...`, `test(voice): ...`.
- PRs: clear description, linked issues, screenshots for example/UI changes.
- Must pass CI, lint/format, tests. Keep per-package CHANGELOG and version bumps in sync; do not add files under `.changeset/`. Do not publish locally—CI handles releases; never commit `dist` or secrets.
- **Version bump commits must include `package-lock.json` and dependent `package.json` range updates.** All packages use `0.x` versioning, so `^0.x.y` does NOT span minor bumps (e.g. `^0.22.0` excludes `0.23.0`). Omitting these causes `npm ci` to fail in CI. Always run `npm install --package-lock-only` then `npm ci` before committing.

## Release Notes / Process
- Applies to all packages: do not use the Changeset CLI or create `.changeset/*` files.
- For releases, update each package’s `CHANGELOG.md` and `package.json` manually (align dependent version ranges as needed).
- Refer to `README.md` for the current release flow; when in doubt, re-read it before acting.
### Release Flow & Failure Recovery
- `release.yml` runs `changesets/action@v1` with `createGithubReleases: true`.
- On merge to `main`, it publishes updated packages to npm, creates tags like `@aituber-onair/<pkg>@x.y.z`, and creates GitHub Releases **only for packages published in that run**.
- `prerelease-next.yml` only updates the `next` prerelease; it does not create stable releases.
- If the release CI fails after some packages were published, re-running will skip already-published packages and only publish the remaining ones; missing GitHub Releases must be created manually.
- Manual recovery (when a Release is missing): ensure the tag exists, then create the Release with changelog notes (e.g., `gh release create "@aituber-onair/chat@0.10.0" -t "@aituber-onair/chat v0.10.0" -F /tmp/chat-0.10.0.md`).

## Security & Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shinshin86/aituber-onair](https://github.com/shinshin86/aituber-onair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
