---
trigger: always_on
description: - All assistant responses for this repository must be in Japanese. Keep explanations concise and avoid excessive logs.
---

# Repository Guidelines

## Agent Response Language
- All assistant responses for this repository must be in Japanese. Keep explanations concise and avoid excessive logs.

## Project Structure & Module Organization
- `src/` — library source. Core classes in `src/core/` (e.g., `VoxelGrid.js`, `VoxelRenderer.js`), utilities in `src/utils/` (camelCase files), entry at `src/index.js` and main class `src/Heatbox.js`.
- `test/` — Jest tests mirroring `src/` (e.g., `test/core/VoxelGrid.test.js`), setup at `test/setup.js`, Cesium mock in `test/__mocks__/cesium.js`.
- `dist/` and `types/` — build outputs (generated). Docs in `docs/`.

## Build, Test, and Development Commands
- `npm run dev` — start webpack-dev-server for local development.
- `npm run build` — produce ESM and UMD bundles and TypeScript types.
- `npm test` / `npm run test:coverage` — run tests with Jest (jsdom env) and coverage.
- `npm run lint` / `npm run lint:fix` — lint and auto-fix with ESLint.
- `npm run type-check` — TypeScript type checking (no emit).
- `npm run docs` — generate JSDoc to `docs/api/`.

## Coding Style & Naming Conventions
- JavaScript with 2-space indent and semicolons enforced; prefer `const`, no `var`.
- Class files use PascalCase in `core/` (e.g., `ColorCalculator.js`); utilities use camelCase (e.g., `validation.js`).
- Linting: ESLint + TS plugin; fix warnings.

## Modularity for New Implementations
- Do not implement new features as a single monolithic file.
- Split by feature and responsibility:
  - Core logic in `src/core/<Feature>/` or a dedicated file under `src/core/` using PascalCase.
  - Shared helpers in `src/utils/` using camelCase, reusable across features.
  - Keep one primary class or cohesive module per file; avoid “god” files.
- Keep files reasonably small and focused (aim for <= ~300–400 LOC where practical).
- Design for testability: mirror structure under `test/` (e.g., `test/core/<Feature>/<File>.test.js`).
- Prefer incremental composition over inheritance for extensibility.
 - When implementing new features, proactively create new files as needed instead of enlarging existing ones. Extract helpers into new modules to keep file sizes modest and responsibilities well distributed across the codebase.

## Testing Guidelines
- Framework: Jest + jsdom; tests in `test/**/*.{test,spec}.js`.
- Coverage thresholds: branches 65%, functions 80%, lines 80%, statements 80%.
- Mock Cesium via `test/__mocks__/cesium.js`.

### Codex Context Considerations (Testing)
- Keep test output minimal to avoid overflowing the agent context window.
- Prefer `npm test --silent` or `jest --silent -t <pattern>` to narrow scope and suppress noise.
- When coverage is needed in this environment, run with reduced reporters:
  - `npm run -s test:coverage --silent -- --coverageReporters=text-summary --reporters=default`
  - Only surface the coverage summary in messages; omit per-file tables unless specifically requested.
- Show only failing test excerpts; omit successful test logs. If failures are large, re-run narrowed: `jest -t <pattern> --silent`.
- Limit displayed logs to concise chunks (roughly <= 200–250 lines) and focus on actionable errors.
- For large snapshots/diffs, share as patches or file references rather than pasting full contents inline.

## Context Window Safety (Codex CLI)
- Always minimize streamed output in this repo to avoid context overflows.
- Default test run for agents: `npm test --silent -- --reporters=summary --bail=1`.
  - If something fails, re-run focused: `jest --silent -t <name> --bail=1` and show only the failing excerpt.
- For CI-style local checks, run steps separately and summarize outcomes instead of piping full logs:
  - Lint: `npm run -s lint` then output only the final error/warning count or "lint ok".
  - Type-check: `npm run -s type-check` then output only errors or "type-check ok".
  - Tests: `npm test --silent -- --reporters=summary --bail=1` and surface only the summary plus failing test snippets.
- Do not paste PASS lines or per-test successes; include only failing tests and the final Jest summary.
- If any single command yields > ~200 lines, abort and re-run with a narrower scope (e.g., add `-t <pattern>`, `--silent`, or smaller file reads `sed -n 1,200p`).
- Prefer `rg`/chunked reads over dumping full files; cap file previews to ~200–250 lines.
- For coverage, use: `npm run -s test:coverage --silent -- --coverageReporters=text-summary --reporters=default` and paste only the summary block.
- When chaining commands, avoid echoing large intermediate outputs; report a compact checklist of pass/fail.

## Commit & Pull Request Guidelines
- Follow Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:` with optional scope (e.g., `feat(core): add diverging colors`).
- PRs include summary, rationale, screenshots/logs, linked issues. Ensure `npm run lint && npm test` pass.
- Update docs and examples when changing public APIs; regenerate API docs with `npm run docs`.
- Commit messages and PR titles must be in English (even though assistant responses are in Japanese).
- Git 操作（特に `git add` や `git commit`）は Codex システムの仕様でユーザー承認が必須。実行前に承認依頼を行い、許可なくコマンドを発行しないこと。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiro-nyon/cesium-heatbox](https://github.com/hiro-nyon/cesium-heatbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
