---
trigger: always_on
description: Large monorepo (55 packages, 200+ e2e fixtures) managed with Lerna-lite and Yarn 4 (Berry, `node-modules` linker, not PnP). TypeScript everywhere, compiled with Webpack per package. Node engines `^18.14.0 || ^20.0.0 || ^22.0.0 || >=24.0.0`.
---

# Jest Repository — Coding Agent Instructions

Large monorepo (55 packages, 200+ e2e fixtures) managed with Lerna-lite and Yarn 4 (Berry, `node-modules` linker, not PnP). TypeScript everywhere, compiled with Webpack per package. Node engines `^18.14.0 || ^20.0.0 || ^22.0.0 || >=24.0.0`.

## Setup

```bash
corepack enable
yarn install        # ~45 s. Python is required (node-gyp).
yarn build:js       # ~5 s. Run this before tests — they import from build/, not src/.
```

Tests transform on the fly via `babel-jest`, but their `import {x} from '../'` resolves to each package's `build/`. `yarn build:js` is required after every checkout. Full `yarn build` (`build:js && build:ts && bundle:ts`) is 3–5 min and only needed when working on type declarations or API Extractor output.

Iterative: `yarn watch` (webpack), `yarn watch:ts` (declarations). Clean: `yarn build-clean`; full reset: `yarn clean-all`.

## Testing

```bash
yarn jest <path>                  # specific file or directory
yarn jest-runtime-vm-modules      # jest-runtime with --experimental-vm-modules (ESM tests)
yarn workspace <name> test        # one package
yarn jest-coverage                # with coverage
yarn jest-jasmine-ci              # CI mode with jasmine2 runner
yarn test-leak                    # detectLeaks across jest-mock/jest-diff/pretty-format
yarn test-types                   # tstyche (type-level tests in __typetests__/)
yarn test-ts                      # TypeScript-config integration tests (separate config)
yarn test-ci-partial:parallel --max-workers <N> --shard=<M>/<N>   # CI-mode sharded
```

- Default runner is `jest-circus` (`JEST_JASMINE=1` swaps to `jest-jasmine2`, kept for compat). Default timeout 70 s.
- Configs: `jest.config.mjs` (main), `jest.config.ci.mjs` (CI reporters), `jest.config.ts.mjs` (the `test-ts` integration).
- New test files are `.ts` (some legacy `.js` remain).
- Each `__tests__/` directory under packages covered by `yarn typecheck:tests` has its own `tsconfig.json` extending `tsconfig.test.json`. Add `"node"` to its `types` array when using Node globals like `Console`/`Stats`/`__dirname`.
- **`yarn typecheck:tests` is gated in CI** — must exit 0. Adding a new package's tests means appending it to the glob in `package.json`.
- **Type tests for `expect` matchers belong in `packages/jest-types/__typetests__/expect/`**, not in `packages/expect/__typetests__/`. The `jest-types` suite tests the public `@jest/globals` surface (what users import); `packages/expect/__typetests__/` covers internal `expect`-package concerns only (e.g. `MatcherFunction`, `JestExpect` shape). When adding type tests for matcher signatures, add them to `jest-types`.
- **E2E tests (`e2e/__tests__/`) can't use `jest.mock`/`jest.fn`** — ESLint enforces this. Use fixture files instead.
- Some e2e tests need Mercurial: `brew install hg`.
- **Docblock pragmas** in test files: `@jest-environment <name>` overrides the test environment; `@jest-environment-options {"key": value}` merges into `testEnvironmentOptions`. Both are extracted by `jest-runner` and apply only to that file.

To run an e2e fixture manually:

```bash
cd e2e/<test-directory>
node ../../packages/jest-cli/bin/jest.js --no-cache
```

CI runs the test matrix with `nick-fields/retry` (10-min timeout, up to 3 retries on flake) across Ubuntu/macOS/Windows × Node 18/20/22/24/25. If a test is consistently failing locally but green in CI, suspect a retry-masked flake.

### Test gotchas worth memorizing

- **Snapshot updates with ANSI colors**: many snapshots contain chalk-rendered ANSI escape sequences. Always update snapshots with `FORCE_COLOR=1 yarn jest <path> -u` so the color output is preserved. Running without `FORCE_COLOR=1` strips the sequences and produces wrong snapshots.
- **Windows CI on path-shaped assertions**: when comparing against a value built via `path.join`/`path.dirname`/`path.basename`, build the expected value with `path.join` too. Hardcoded POSIX strings (`'/path/to/x'`) fail on Windows.
- **Throwing-getter regression on `globalThis` scans**: iterating `Object.keys(scope)` and reading `scope[key]` crashes if a user installed a throwing getter. Use `'key' in scope` (the `has` trap, not `get`) as the gate.
- **ESM helpers from `@jest/test-utils`**: `testWithVmEsm` (Node 18+ with `--experimental-vm-modules`), `testWithLinkedSyntheticModule` (Node 22.21+/24.8+, gates on `linkRequests`), and `testWithSyncEsm` (Node 24.9+, gates on `hasAsyncGraph`). `yarn jest packages/jest-runtime` does **not** include the ESM suite — use `yarn jest-runtime-vm-modules`.

## Linting

Lint changed files after every edit:

```bash
yarn eslint --cache --fix <files>
```

Full lint (`yarn lint`) before pushing. ESLint 9.x flat config (`eslint.config.mjs`), with a local plugin at `.eslintplugin/index.mjs` providing `local/no-restricted-types-eventually`, `local/prefer-rest-params-eventually`, `local/prefer-spread-eventually`. Markdown code blocks are linted too.

### Hard rules (CI fails)

- **`graceful-fs`, never `fs`/`node:fs`** — both are banned by `no-restricted-imports`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jestjs/jest](https://github.com/jestjs/jest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
