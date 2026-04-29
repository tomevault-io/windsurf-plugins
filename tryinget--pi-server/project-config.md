---
trigger: always_on
description: This document captures patterns, anti-patterns, and gotchas discovered during development. Read this before working on pi-server.
---

# AGENTS.md — Crystallized Learnings for pi-server

This document captures patterns, anti-patterns, and gotchas discovered during development. Read this before working on pi-server.

Because parent/global AGENTS files are concatenated into every session, keep this file repo-specific and prefer pointing to canonical docs instead of duplicating large policy blocks.

---

## Canonical References

Start here before changing behavior that crosses repo boundaries:
- `README.md` — package shape, architecture overview, release flow
- `PROTOCOL.md` — wire contract and client-visible semantics
- `docs/release-policy.md` — commit, changelog, and breaking-change policy
- `next_session_prompt.md` — current handoff / immediate priorities

---

## Architecture Principles

1. **The protocol IS the architecture** — `types.ts` is the single source of truth
2. **Server should be thin** — AgentSession does the work, we just multiplex
3. **Pass-through pattern** — Session commands are thin wrappers around AgentSession methods
4. **Handler map > switch** — `Record<string, CommandHandler>` for O(1) dispatch, easy extension
5. **Extract to stores** — State with independent lifecycle gets its own module
6. **SessionResolver is the NEXUS seam** — Interface for session access enables test doubles

---

## ADR-0001: Atomic Outcome Storage

**Status:** Accepted (2026-02-22)

Full ADR: `docs/adr/0001-atomic-outcome-storage.md`

### The Invariant

> Same command ID must ALWAYS return the same response. Not "usually." Not "after the callback completes." ALWAYS.

### Three Critical Rules

1. **Store BEFORE return** — Outcomes are stored before the response is returned, not in async callbacks
2. **Timeout IS a response** — Timeout responses are stored as valid outcomes (with `timedOut: true`)
3. **Replay is FREE** — Replay operations are O(1) lookups, exempt from rate limiting

### Code Pattern

```typescript
// WRONG: Async callback creates race condition
commandExecution.then((response) => {
  this.storeOutcome(commandId, response);  // After return
});
return withTimeout(commandExecution, ...);

// CORRECT: Atomic storage before return
let response: RpcResponse;
try {
  response = await executeWithTimeout(commandExecution, ...);
} catch (error) {
  response = { success: false, error: error.message, timedOut: true };
}
this.storeOutcome(commandId, response);  // BEFORE return
return response;
```

### Reject, Don't Evict

When in-flight command limit is reached, **reject new commands** instead of evicting old ones. Eviction breaks dependency chains.

```typescript
// WRONG: Eviction breaks dependencies
if (this.inFlight.size >= max) {
  const oldest = this.order.shift();
  this.inFlight.delete(oldest);  // Dependent commands fail!
}

// CORRECT: Reject preserves dependencies
if (this.inFlight.size >= max) {
  return { success: false, error: "Server busy - please retry" };
}
```

### Free Replay

Replay reads from stored outcomes — no execution cost. Rate limiting applies only to NEW executions.

```typescript
// Check replay FIRST (free)
const replayResult = this.replayStore.checkReplay(command, fingerprint);
if (replayResult.found) {
  return replayResult.response;  // No rate limit charge
}

// THEN rate limit (only for new executions)
if (!this.governor.canExecuteCommand(sessionId)) {
  return { success: false, error: "Rate limit exceeded" };
}
```

---

## File Responsibilities

| File | Responsibility | Don't Put Here |
|------|----------------|----------------|
| `server.ts` | Transports (WebSocket, stdio) only | Command logic, session lifecycle |
| `session-manager.ts` | Orchestration: coordinates stores, engines, sessions | Direct command handling (delegate to handlers) |
| `server-command-handlers.ts` | Server command handlers, handler map dispatch | Session lifecycle, broadcast |
| `command-router.ts` | Session command handlers, routing | Session lifecycle, broadcast |
| `command-classification.ts` | Pure command classification (timeout, mutation) | State, side effects |
| `command-replay-store.ts` | Idempotency, duplicate detection, outcome history | Execution logic |
| `session-version-store.ts` | Monotonic version counters per session | Replay semantics |
| `command-execution-engine.ts` | Lane serialization, dependency waits, timeouts | Storage |
| `session-lock-manager.ts` | Per-session-ID mutual exclusion for create/delete | Long-running locks |
| `extension-ui.ts` | Pending UI request tracking | Command handling, transport |
| `types.ts` | Protocol definitions | Implementation |
| `validation.ts` | Command validation, reserved ID enforcement | Execution logic |

---

## pi Integration Gotchas

### Extension UI is NOT an AgentSessionEvent

`extension_ui_request` comes through `ExtensionUIContext`, not the event stream. To receive extension UI requests:

```typescript
// WRONG: Waiting for AgentSessionEvent with type "extension_ui_request"
session.subscribe((event) => {
  if (event.type === "extension_ui_request") { /* NEVER FIRES */ }
});

// RIGHT: Provide custom ExtensionUIContext to bindExtensions
await session.bindExtensions({
  uiContext: {
    async select(title, options, opts) {
      // This is called when extension needs user input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tryingET/pi-server](https://github.com/tryingET/pi-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
