---
trigger: always_on
description: You are an expert Node.js developer contributing to MySQL2, a high-performance MySQL driver focused on compatibility.
---

# MySQL2 — Agent Instructions

You are an expert Node.js developer contributing to MySQL2, a high-performance MySQL driver focused on compatibility.

## Project

- Minimum compatibility: **Node 14** (ignore the `engines` field in `package.json`).
- Core: `/lib` → exposed via `index.js` (callback) and `promise.js` (promise-based).
- Types: `/typings` (`.d.ts` files) → exposed via `index.d.ts` (callback) and `promise.d.ts` (promise-based). See `.github/skills/types/SKILL.md` for the full architecture, known gaps, and decision tree.
- Documentation: `website/docs/` (Docusaurus).
- Tests:
  - Unit: `test/unit` (parallel)
  - Integration: `test/integration` (parallel, runs alongside unit tests)
  - Global: `test/global` (sequential — robust tests that affect MySQL Server global state with advanced cleanup)

---

## Skills

- Use the `/lagune` skill ([.claude/skills/lagune/SKILL.md](.claude/skills/lagune/SKILL.md)), whenever you build or change code, so it improves safety by default. If it is not available, run `npx -y lagune@latest pull` to install it from its manifest.

---

## Tests

**Read `test/common.test.mts` before writing any test.** It provides shared helpers: `createConnection`, `createPool`, `createPoolCluster`, `getConfig`, `createServer`, `getMysqlVersion`, etc.

The test runner is **Poku** ([docs](https://poku.io/docs) · [repo](https://github.com/wellwelwel/poku)). Test files use `.mts` (ESM TypeScript) and support top-level `await`.

Assertions, utilities, and test structure come from Poku:

```ts
import { describe, it, assert, skip, sleep, strict } from 'poku';
```

- `skip` — Skips the entire test file and reports it (e.g., Deno-only tests, specific Node versions, etc.).
- `sleep` — Waits for a given duration when needed: `await sleep(100)`.

| File                      | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| `test/common.test.mts`    | Shared helpers — **read before writing new tests**                  |
| `poku.config.js`          | Poku config: parallel/sequential suites, timeouts, test directories |
| `test/docker-compose.yml` | Local environment with MySQL, Node, Deno, Bun, and coverage         |

```sh
npm run typecheck                        # type-check the project
npm run lint:fix                         # fix lint and formatting
FILTER=test/unit/my-test.mts npx poku    # run a specific test via Poku
npx tsx test/unit/my-test.mts            # run a test directly
```

### Connection scope and resource cleanup

Never close the connection in the same scope as an assertion that may fail. If the assertion throws, `end()` will never be reached and the process will hang indefinitely.

Separate creation/cleanup into an outer scope:

```ts
// ❌ Wrong — end() in the same scope as the assertion
await describe('test', async () => {
  await it('should do something', async () => {
    const connection = await createConnection();
    assert(false);
    await connection.end(); // never reached
  });
  // process hangs
});

// ❌ Wrong — try-finally is a workaround, not a fix
await describe('test', async () => {
  await it('should do something', async () => {
    const connection = await createConnection();
    try {
      assert(false);
    } finally {
      await connection.end();
    }
  });
  // process hangs
});

// ✅ Correct — end() in an outer scope
await describe('test', async () => {
  const connection = await createConnection();

  it('should do something', () => {
    assert(false); // fails in its own scope
  });

  await connection.end(); // always reached
});
```

- Applies to any teardown method (`close`, `end`, `destroy`, `release`) and any connection type (`Connection`, `Pool`, `PoolCluster`, etc.).
- Use nested or dedicated `describe` blocks to isolate each connection.
- The same applies to callbacks — `end()` may be inside a nested callback that is never invoked if an assertion fails first.

Prefer `await conn.promise().end()` instead of wrapping callbacks in `new Promise`:

```ts
// ❌ Avoid
await new Promise<void>((resolve) => pool.end(() => resolve()));

// ✅ Prefer
await pool.promise().end();
```

### Avoid timer-dependent tests

Avoid writing tests that depend on `setTimeout` or `sleep` to wait for internal timers (e.g., idle timeout cleanup). These can produce flaky results in CI environments like GitHub Actions, where execution timing is unpredictable. Instead, call the internal method directly or assert the state synchronously right after the action. If a timer-based behavior must be tested (e.g., a "timebomb"), isolate it in a way that does not depend on wall-clock timing.

### `async`/`await`

Poku treats `async`/`await` just like standard JavaScript: use `await` on `describe`/`it`/`test` **only** when the callback is asynchronous. Otherwise, do not include `async` or `await`.

**Asynchronous:**

```ts
await describe('test', async () => {
  const connection = createConnection().promise();

  await it('should do something', async () => {
    const result = await connection.query('SELECT 1');

    assert(result);
  });

  await connection.end();
});
```

**Synchronous:**

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidorares/node-mysql2](https://github.com/sidorares/node-mysql2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
