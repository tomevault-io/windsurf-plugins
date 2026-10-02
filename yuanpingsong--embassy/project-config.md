---
trigger: always_on
description: Embassy is a personal, same-user gateway between live Claude Code sessions and
---

# Repository guidance

Embassy is a personal, same-user gateway between live Claude Code sessions and
Codex CLI tasks, locally and across user-owned Macs over SSH. Treat identity,
process control, private state, native protocol parsing, provider writes,
federation, and delivery settlement as security-sensitive boundaries.

## Working style

- Prefer the smallest direct implementation of the signed product contract.
  Remove responsibilities the product no longer owns; do not preserve an old
  abstraction merely because tests exist for it.
- Optimize for concrete progress and maintainable code. Avoid ticket ceremony,
  per-commit accounting, freeze rituals, status reports, and extra gates that
  do not improve the release candidate.
- Implementation is engineer-led. Contact the PM only for a decision
  that changes the signed contract, a blocker only the PM/founder can clear, or
  the final release-candidate handoff. Use Embassy as the exclusive PM channel.
- Use subagents for substantial independent work or hard review, with explicit
  non-overlapping file ownership. Do not delegate minute searches.
- Preserve unrelated user changes in a dirty worktree. Never write to public
  main, force-push, move tags, install global packages, or change live service,
  provider, skill, or sandbox configuration unless explicitly instructed.

## Required verification

Run `TMPDIR=/tmp npm run check` after source or test changes and before a
release candidate. Use the soak suite for scheduling, restart, native
transport, or settlement changes. Keep CI green when practical during
development; CI must be green at a release candidate.

Routine tests use test-owned temporary directories, fake Claude sockets, fake
App Server transports, and fake SSH processes. They must not inspect the live
Claude registry, connect a live provider or SSH host, install a service, or
make a model request.

A live provider read, connection, message, SSH drill, service mutation, or
global install requires the user's explicit authorization for that exact
operation. Previous authorization does not make later live operations routine.
Never enable live provider activity in CI.

## Core architecture

- `ledger.ts` is the pure synchronous transition core. It owns endpoint,
  delivery, rate, and retirement state but no filesystem, provider, callback,
  or timer work.
- `owned-state.ts` owns the single private schema-7 atomic JSON document.
- `endpoint-directory.ts` owns current alias lookup and exact endpoint
  resolution.
- `coordinator.ts` owns batching, scheduling, authorization, and phase-derived
  loss handling.
- `native-destinations.ts` provides the Claude-socket and Codex-operation write
  adapters. `federation.ts` provides the direct SSH adapter and protocol 3.
- `broker.ts` composes application operations. `broker-control.ts`,
  `local-control.ts`, and `core-cli.ts` are the closed private control and CLI
  surfaces. `runtime.ts` owns startup and shutdown ordering.

Do not add another delivery state machine, store, provider-independent engine,
catalog authority, callback service, activity journal, migration layer, or
generic provider RPC without an explicit contract change.

## Product and safety invariants

The governing doctrine is
[What Embassy defends, and what it deliberately does not](SECURITY.md#what-embassy-defends-and-what-it-deliberately-does-not).
A new audit check must cite a current doctrine sentence. If none applies,
propose a contract change rather than expanding the boundary through a test.

- Support Claude→Claude, Claude→Codex, Codex→Claude, and Codex→Codex locally
  and across directly configured SSH gateways. Sending is one CLI command;
  receiving wakes the agent natively and never requires polling.
- Endpoint `(opaque ID, host, provider)` is identity. An alias is current lookup
  and display data. Resolve a name once; never silently retarget admitted work
  after rename, replacement, retirement, or catalog change.
- Discover Codex agents from bounded same-user App Server metadata. Keep the
  immutable native ID private, list only the recency top 20 roots, and treat
  native names as mutable aliases. `register-codex` remains a fallback using
  inherited `CODEX_THREAD_ID`; discovery and registration must reconcile one
  endpoint identity. Never accept, print, or guess the native ID. A Claude
  caller is derived from its inherited absolute
  `CLAUDE_CODE_MESSAGING_SOCKET`; never accept, print, or persist that path.
  Native IDs may exist only in closed private route state.
- Every provider write revalidates the exact current local endpoint and exact
  prepared bytes after preparation. Provider I/O never runs inside an
  owned-state transaction.
- Keep delivery phases `queued`, `reserved`, `armed`, `accepted`, and
  `terminal` distinct. Only positive no-write evidence may requeue work.
  Reserved work may recover after restart; armed and accepted uncertainty is
  terminal and is never replayed. Late callbacks cannot downgrade a terminal
  result.
- Keep bodies, queues, batches, deadlines, rates, retained rows, protocol
  frames, and concurrent operations bounded. One wake may carry a bounded FIFO
  batch, but every message retains its own identity, provenance, receipt, and
  result.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuanpingSong/embassy](https://github.com/YuanpingSong/embassy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
