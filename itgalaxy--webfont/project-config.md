---
trigger: always_on
description: Instructions for AI agents and contributors automating work in this repository.
---

# Agent guidelines

Instructions for AI agents and contributors automating work in this repository.

**GitHub Copilot:** repository custom instructions live at [`.github/instructions/webfont.instructions.md`](./.github/instructions/webfont.instructions.md) (symlink to this file). Edit **AGENTS.md** only — not the symlink target path in `.github/instructions/`. [CLAUDE.md](./CLAUDE.md) symlinks here for Claude Code.

## Monorepo layout

npm workspaces monorepo ([ADR 0013](./docs/adr/0013-npm-workspaces-monorepo.md)): published library in **`packages/webfont`** (`name: "webfont"` on npm); VitePress docs and user-facing markdown at the **repo root**. Run `npm ci` at root; `npm test` / `npm run build` / `npm run test:package` delegate to the `webfont` workspace.

## Testing (Vitest)

### Vitest globals (all workspaces)

Workspace Vitest configs use **`test.globals: true`** ([ADR 0015](./docs/adr/0015-vitest-globals-in-workspaces.md)). Typecheck configs include `"vitest/globals"` so `describe` / `it` / `expect` / `expectTypeOf` / `vi` / hooks need **not** be imported. Explicit `import { … } from "vitest"` is optional. Emit/build tsconfigs must exclude `*.test.ts` and must not require Vitest types on production sources.

### Do not mix sync-throwing `fs` calls inside async callbacks

In `beforeAll`, `beforeEach`, `afterAll`, `afterEach`, or any `new Promise((resolve, reject) => { ... })` callback, avoid synchronous APIs that throw (`fs.mkdirSync`, `fs.symlinkSync`, `fs.unlinkSync`, bare `throw`, etc.).

If they throw, the Promise is not rejected cleanly. Vitest may report an uncaught exception instead of a failed hook, which is harder to diagnose.

**Prefer** `async` hooks with `fs/promises`:

```ts
beforeAll(async () => {
  await fsPromise.mkdir("temp/fixture", { recursive: true });
  await fsPromise.symlink(source, link, "dir");
});
```

**If you must use a callback-style API** (for example `rimraf`), keep follow-up work outside the callback or wrap sync code in `try/catch` and call `reject(error)`.

### Sync `fs` in test bodies is fine

Calling `fs.mkdirSync` / `fs.rmSync` directly in an `async` `it(...)` block is acceptable. Vitest attributes synchronous failures to the test. Use `try/finally` for cleanup.

### Clean up `mkdtempSync` directories in hooks

When tests create temp trees with `mkdtempSync`, remove them in an **`async` `afterEach`** via `fs/promises` — do not only reset a tracking array. Clearing the array without `rm` leaks directories under the OS temp folder and skips cleanup when a test fails mid-run.

```ts
import { rm } from "node:fs/promises";

const tempDirs: string[] = [];

afterEach(async () => {
  await Promise.all(
    tempDirs.splice(0).map((dir) => rm(dir, { recursive: true, force: true })),
  );
});
```

Use sync `mkdtempSync` / `mkdirSync` in the test body if convenient; keep **deletion async** in the hook (same rule as other async hooks — no sync-throwing `fs.rmSync` in `afterEach` callbacks).

### Promisify callback-only helpers once

When a dependency only exposes callbacks (`rimraf`, legacy `fs.mkdir`), extract a small `promisify` helper and use it from `async` hooks instead of nesting callbacks.

### `it` descriptions must use `should` or `should not`

Every `it("...")` title in this repo must read as a behavior statement with **`should`** or **`should not`**. This keeps Vitest output scannable and consistent across unit, contract, and CLI integration tests.

```ts
// Good
it("should return default webfont options", () => { ... });
it("should not emit result.svg when only woff2 is requested", async () => { ... });
it("should document that is-svg throws TypeError for non-string input", () => { ... });

// Avoid
it("returns default webfont options", () => { ... });
it("documents that is-svg throws TypeError for non-string input", () => { ... });
it("throws when given binary font buffers", async () => { ... });
```

When adding or renaming tests, follow this rule even in contract-style `describe` blocks that document library behavior.

### Tests are required for code changes

Every PR that changes **runtime behavior** (features, fixes, refactors with observable effects, CLI/API/demo wiring) must include **automated tests in the same change**. Do not land production code and defer tests to a follow-up.

| Change type | Minimum expectation |
|-------------|---------------------|
| New option, flag, or pipeline step | Unit test(s) for the module + integration test when the public entry (CLI, `webfont()`, worker) is affected |
| Bug fix | A test that **fails without the fix** and names the regression |
| Guard or workaround | Focused unit test documenting **why** the guard exists (see table below) |
| Packaging / build (`packages/webfont/vite.config.ts`, `packages/webfont/package.json#exports`, `files`, `main`, `module`, `browser`, `bin`, `types`, or `dist/` layout) | Run `npm run test:package` locally (publint + attw + pack-smoke) and rely on the CI step; extend `packages/webfont/scripts/pack-smoke-test.mjs` when a new consumer entry point ships. See [ADR 0012](docs/adr/0012-published-package-validation.md). |
| Docs-only | No new tests; say so in the PR **Testing** section |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itgalaxy/webfont](https://github.com/itgalaxy/webfont) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
