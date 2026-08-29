---
trigger: always_on
description: Browser Control is a local browser driver for trusted agents. It controls the
---

# Browser Control

Browser Control is a local browser driver for trusted agents. It controls the
user's existing Chromium-family browser through a small MV3 extension shim and a
local Node relay.

## Source Of Truth

- Keep `PLAN.md` updated when architecture, scope, install flow, or product
  preferences change.
- Keep `CONTEXT.md` updated when domain language changes.
- Keep `skills/browser-control/SKILL.md` updated when the agent-facing workflow,
  commands, setup steps, or troubleshooting behavior changes.
- Keep the installed OpenCode skill at
  `~/.config/opencode/skills/browser-control/skill.md` synced with
  `skills/browser-control/SKILL.md` after agent-facing workflow changes.
- If a code change affects how agents should use Browser Control, update the
  skill in the same change.
- `browser-control skill` must print the current `skills/browser-control/SKILL.md`
  text so another agent can fetch the installed workflow instructions.

## Architecture Preferences

- Browser Control is a driver, not an LLM agent.
- Use the user's already-running Chromium-family browser first.
- Keep tabs in a loose attached-tab pool for v1.
- Prefer a code-first `execute(code)` interface over many tiny action tools.
- Execute runs inside relay-backed sessions. Bare CLI execute atomically creates
  a fresh readable id such as `cosmic-otter-866` and prints how to continue with
  `--session`; it never infers agent identity from shared current-session state.
- Relay-backed CLI commands auto-start a detached relay when needed. `status`
  and `doctor` remain observational, and `serve` is only the foreground/debug
  path. MCP uses the same detached relay lifecycle instead of owning an
  in-process relay, so an MCP restart cannot interrupt CLI handoffs. The first
  session is created atomically in the execute request.
- Each Browser Control session owns one default page and persistent JavaScript
  `state`; do not default to arbitrary shared tabs for normal execute calls.
- Use stock `playwright-core` for v1.
- Use Effect v4 for Node-side code. Treat a local `Effect-TS/effect` checkout as
  the source of truth for current Effect APIs and patterns; `effect-smol` is the
  archived former v4 repository.
- Prefer `Effect.fn` / `Effect.fnUntraced` for functions that return Effects,
  and use scoped resources (`Effect.acquireRelease`, `Effect.scoped`) for
  Playwright and relay lifecycles.
- Read application runtime configuration through Effect `Config`. Direct
  `process.env` access is reserved for synchronous process-fault reporting and
  child-process environment forwarding at Node adapter boundaries.
- Keep the relay/extension protocol as custom JSON-over-websocket unless there is
  a concrete reason to adopt Effect RPC across that boundary.
- Keep the extension as a stable shim over Chrome APIs. Put behavior in the
  relay when possible so iteration usually requires only restarting Node, not
  reloading the extension.
- Relay HTTP wire shapes live in `src/relay-schema.ts` (Effect Schema). Both the
  HTTP responders and clients must derive types from those schemas; do not
  hand-roll relay JSON parsers. Error responses use the shared coded
  `ErrorEnvelope`; keep the relay message top-level while mapping tagged domain
  errors to stable codes and HTTP statuses.
- Tie relay HTTP effects to the response lifetime with an `AbortSignal`.
  Execute workers outlive an interrupted request once browser work starts;
  retain the session permit through final journal and catalog writes so aborted
  clients cannot lose aftermath bookkeeping or overlap later page mutations.
- The CLI and MCP server talk to the relay only through the shared
  `src/relay-client.ts` service (`RelayClient.Service`), never through ad-hoc
  fetch/node:http calls. Failures are tagged errors that keep the relay's own
  error message as the top-level message.
- Human session-management commands keep an endpoint-scoped current id in
  `~/.browser-control/session.json`; execute and adopt never use it implicitly.
  Invalid persisted session JSON is reported and preserved, never treated as an
  empty store that a later write may overwrite.
- Relay session descriptors persist per port under
  `~/.browser-control/relays/<port>/sessions.json`. After a relay restart,
  restore session ids, read-only mode, and exact target ownership when that tab
  reappears; JavaScript `state` and snapshot refs intentionally reset and warn.
  Win the endpoint port before loading or writing this catalog. Successful
  durable lifecycle operations await atomic replacement plus file and directory
  sync. Corrupt catalogs fail relay startup and are never overwritten.
- An extension RPC timeout fails only that command; the extension socket is
  closed only when a websocket-level ping probe also fails.
- CDP guardrails are pure logic in `src/cdp-guardrails.ts`, enforced at the top
  of `routeCdpCommand`. Destructive browser-state methods are always blocked;
  read-only sessions additionally reject `Input.*`.
- Browser-context CDP methods route through a session-owned root for named
  clients or exactly one visible root for raw clients. A named client never
  falls back to an unrelated unowned tab.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anomalyco/browser-control](https://github.com/anomalyco/browser-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
