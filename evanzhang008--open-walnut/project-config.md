---
trigger: always_on
description: Every code change MUST include tests. No exceptions. Tests are not an afterthought — they are part of the feature.
---

# AGENTS.md — Walnut Development Guide

## Testing Requirements

Every code change MUST include tests. No exceptions. Tests are not an afterthought — they are part of the feature.

### Test Philosophy

**Think before you write.** A bad test is worse than no test — it gives false confidence. Before writing any test:

1. **Trace the data flow.** For the feature you're testing, draw the path: User action → API → Core → Side effects → Event Bus → WS → UI. Each hop is a potential failure point.

2. **Identify what can break.** Don't test that `1 + 1 === 2`. Test the seams: what happens when the file doesn't exist? When the ID is ambiguous? When two clients are connected? When the input has `" / "` in it?

3. **Test behavior, not implementation.** Don't assert on internal state. Assert on observable outcomes: HTTP responses, WS events received, data persisted to disk.

### The Pyramid: What to Write

```
                    ┌──────────┐
                    │Playwright│  Tier 4: Real browser (visual, clicks, real-time)
                   ┌┴──────────┴┐
                   │   E2E       │  Tier 3: Real server + WS (full pipeline)
                  ┌┴─────────────┴┐
                  │  Integration   │  Tier 2: Supertest routes (HTTP contracts)
                 ┌┴────────────────┴┐
                 │     Unit          │  Tier 1: Isolated functions (logic, edge cases)
                 └───────────────────┘
```

**The rule: every feature needs at least 1 real E2E test.** Unit tests are for edge cases. The happy path MUST be tested with a real server, real HTTP, and real WebSocket.

### Tier 1 — Unit Tests (`tests/core/`, `tests/agent/`)

Isolated functions with mocked file paths (temp directory). Tests go here when you're testing:
- Core logic (task state transitions, memory writes, search indexing)
- Parsing and formatting (slash format, ID generation)
- Edge cases (empty input, ambiguous prefix, concurrent writes)

```bash
npx vitest run tests/core/my-feature.test.ts
```

### Tier 2 — Integration Tests (`tests/web/routes/`)

API routes tested via supertest (in-process Express, no real server). Tests go here for:
- HTTP status codes and response shapes
- Request validation (missing fields, bad params, URL encoding)
- Route registration (new endpoints actually respond)

```bash
npx vitest run tests/web/routes/my-feature.test.ts
```

### Tier 3 — E2E Tests (`tests/e2e/`)

Real server on random port + WebSocket clients. This is the **most important tier**. Tests go here for:
- Full data pipeline: REST → Core → Event Bus → WS broadcast → correct payload
- State persistence: POST then GET to verify
- Multi-client scenarios: 2+ WS clients both receive the same event
- Cross-feature interactions: one feature's output is another's input

```bash
npx vitest run --config vitest.e2e.config.ts tests/e2e/my-feature.test.ts
```

**E2E test template**: see `.claude/commands/test.md` for the full boilerplate.

### Tier 4 — Playwright Browser Tests

Use the Playwright MCP tools to test the actual UI in a real browser. Do this AFTER all server-side tests pass.

**When to use Playwright:**
- A feature changes what the user sees (new tab, new button, layout change)
- A feature involves real-time updates (WS event → UI re-render)
- A feature has interactive elements (click handlers, form submission)

**How to run Playwright tests:**

1. Start the dev server in the background:
```bash
walnut web --port 3457 &
```

2. Use Playwright MCP tools:
```
mcp__playwright__browser_navigate → http://localhost:3457
mcp__playwright__browser_snapshot → read the page structure
mcp__playwright__browser_click   → interact with elements
mcp__playwright__browser_take_screenshot → capture visual state
```

3. Verify:
- Elements render correctly (tabs, buttons, lists)
- Click interactions work (checkbox toggles, star buttons)
- Real-time updates appear (create task via API, verify it appears in UI)

4. Kill the server when done.

**Playwright checklist for UI features:**
- [ ] Page loads without errors
- [ ] New UI elements are visible
- [ ] Click interactions produce expected state changes
- [ ] Screenshot captured for visual verification

### What Makes a Good Test

**Good test:** Creates a task with `category: "idea / work idea"` via POST, then GETs it back and verifies `category === "idea"` and `project === "work idea"`. Connects a WS client and verifies the `task:created` event carries the parsed fields.

**Bad test:** Imports `parseGroupFromCategory`, calls it with `"idea / work idea"`, asserts the return value. (This tests the utility, not the feature. It doesn't verify the parsing is actually wired into `addTask`.)

**Good test:** Creates a task, toggles it complete via API, checks the response says `"done"`, connects a second WS client, toggles it back, verifies both clients get `task:updated` with `status: "todo"`.

**Bad test:** Calls `toggleComplete()` directly, asserts `task.status === 'done'`. (Doesn't test the route, doesn't test event emission, doesn't test multi-client delivery.)

### Test Quality Checklist

Before a feature is done, verify:

- [ ] **At least 1 E2E test** that exercises REST → Core → Bus → WebSocket

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvanZhang008/open-walnut](https://github.com/EvanZhang008/open-walnut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
