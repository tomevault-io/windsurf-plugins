---
trigger: always_on
description: This document helps AI agents work effectively in this codebase. It explains the philosophy, patterns, and pitfalls behind the code, so you can make good decisions on any task, not just scenarios explicitly covered.
---

# Agents

This document helps AI agents work effectively in this codebase. It explains the philosophy, patterns, and pitfalls behind the code, so you can make good decisions on any task, not just scenarios explicitly covered.

## Philosophy

This is a minimal, idiomatic WebSocket library. Simplicity is a feature.

Before adding code, articulate why it's necessary in one sentence. If you cannot justify the addition, it probably isn't needed.

Before adding a dependency, don't. Tests requiring external packages are isolated in `internal/thirdparty`.

## Workflow

Every task follows phases. Do not skip phases; if you realize you missed something, return to the appropriate phase.

**Making changes:**

1. **Research** - Understand the problem and codebase before acting
2. **Plan** - Articulate your approach before implementing (when asked, or for complex changes)
3. **Implement** - Make changes in small, verifiable steps
4. **Verify** - Confirm correctness using external tools

For trivial changes (typo fixes, comment updates), an abbreviated workflow is acceptable.

## Research

Research in sequential passes. Each pass has one focus. Don't skip ahead to code until you've completed the earlier passes.

**Pass 1: Read the issue.** If you were given a link, read it now. Do not explore code, do not pass go. Fetch the linked issue or document first. Summarize what it asks for. If you cannot restate the problem, you are not ready to proceed.

**Pass 2: Read linked references.** Follow every link in the issue: related issues, RFCs, external docs. Document what you learn. Code comments reference RFC 6455 (WebSocket), RFC 7692 (compression), and RFC 8441 (HTTP/2).

**Pass 3: Trace the code.** Start from public API inward: `Accept` (server) or `Dial` (client) → `Conn` → `Reader`/`Writer`. Read tests for intent; the autobahn-testsuite (`autobahn_test.go`) validates protocol compliance.

**Pass 4: Check both platforms.** Native Go files have `//go:build !js`. WASM lives in `ws_js.go`. Same API, different implementations. WASM wraps browser APIs and cannot control framing, compression, or masking.

**Pass 5: Search exhaustively.** If the change affects a pattern used in multiple places, grep for all instances. Missing one creates inconsistent behavior.

**Pass 6: Document unknowns.** List what you still don't know. Unknown unknowns become known unknowns when you ask "what am I still unsure about?"

**After all passes:** Can you restate the problem in your own words? If not, return to Pass 1. Gaps in earlier passes will cause problems later.

## Plan

When asked to plan, write to `PLAN.md`. Write for someone else, not yourself; don't skip context you already know.

**Pass 1: Document research.** Summarize what you learned in Research. Follow every reference; document findings so the implementer can verify. If the research section is empty, you haven't researched enough.

**Pass 2: Consider approaches.** For non-trivial problems, enumerate at least two approaches. For each, note: what would change, what could go wrong, what's the tradeoff.

**Pass 3: Detail the chosen approach.** Explain what and why, not step-by-step how. Point to specific files, functions, line numbers. Make claims verifiable. Leave room for the implementer to find a better solution.

**Pass 4: List open questions.** What's still unclear? What assumptions are you making? What would change your approach?

**After all passes:** Review from the implementer's perspective. Could they start work with only this document? If not, add what's missing.

## Implement

Implement in sequential passes. Don't write code until you've completed the verification passes.

**Pass 1: Verify understanding.** Did you do your research? Can you state your approach in one sentence? If requirements are ambiguous, stop and ask. A wrong assumption wastes more time than a quick question.

**Pass 2: Check scope.** Does this need to exist? Check if it already exists in the API. Is this the library's job or the user's job? The library handles protocol correctness; application concerns (reconnection, auth, routing) belong in user code.

**Pass 3: Check invariants.** Walk through Key Invariants before writing code:

- Reads: Will something still read from the connection?
- Pools: Will pooled objects be returned on all paths?
- Locks: Are you using context-aware `mu`, not `sync.Mutex`?
- Independence: Are you coupling reads and writes unnecessarily?

**Pass 4: Implement.** Make the change. Every change needs a reason; if you can't articulate why it improves things, don't make it. Preserve existing comments unless you can prove they're wrong.

**Pass 5: Verify examples.** Trace through usage examples as if writing real code. If an example wouldn't compile, the design is wrong. Check edge cases: what happens on error? On cancellation?

**After all passes:** If feedback identifies a problem, fix that specific problem. Don't pivot to a new approach without articulating what failed and why the new approach avoids it.

## Verify

Verify using external signals. Self-assessment is unreliable; these tools are the ground truth.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/websocket](https://github.com/coder/websocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
