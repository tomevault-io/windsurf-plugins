---
trigger: always_on
description: For longer tasks that can be divided into independent, bounded workstreams,
---

# Repository Agent Guidance

## Subagents

For longer tasks that can be divided into independent, bounded workstreams,
proactively use subagents without waiting for the user to request delegation.
Good candidates include codebase exploration, separate component or risk
reviews, test-gap analysis, running independent test suites, and investigating
unrelated failures or logs.

Use the project-scoped agents under `.codex/agents/` when their role matches:

- `dakia_explorer` for fast, read-only execution-path and dependency mapping.
- `dakia_implementer` for a clearly owned implementation slice.
- `dakia_test_engineer` for adversarial test design and independent test runs.
- `dakia_reviewer` for correctness, security, concurrency, and regression review.
- `dakia_native_release_verifier` for Tauri, macOS, signing, notarization,
  updater, and release verification.

Give each subagent a concrete scope and expected result, run independent
workstreams in parallel, and wait for all relevant results before synthesizing
the outcome. Keep requirements, cross-cutting decisions, integration, and final
verification with the main agent.

Prefer parallel read-heavy work. For write-heavy work, assign non-overlapping
files or components with clear ownership; otherwise keep implementation
sequential to avoid edit conflicts and coordination overhead. Do not use
subagents for small tasks where delegation would add more overhead than value.

## Fresh worktrees

Run `npm run setup:worktree` before development or native checks in a fresh
worktree. The command is idempotent and prepares locked JavaScript dependencies,
Git LFS classifier assets, the macOS ONNX Runtime, and the debug CLI sidecar.
`npm run dev` performs the required subset automatically.

## Testing

Always rely on unit and integration tests as the primary form of verification.
Do UI testing only after exhausting those layers, at the very end.

## Optimistic mutations

All user-facing mutations must use optimistic updates. Reflect the intended
result in local UI state immediately, without blocking unrelated or subsequent
mutations on the network request. Reconcile with the authoritative result in
the background, and restore the affected state with clear error feedback when
the mutation fails.

Write regression tests from the actual failing input and user-visible outcome,
not merely from the current helper implementation. When a bug comes from email
markup, protocol data, database state, or another structured external input:

- Capture the smallest faithful, redacted fixture from the real input. Preserve
  provider-specific nesting, attributes, whitespace, and sibling relationships;
  do not replace it with cleaner invented markup that only exercises the
  expected code path.
- Assert semantics at the component or integration boundary. For rendering
  regressions, verify what remains visible, what moves behind a disclosure, the
  number and state of controls, and the complete expand-collapse-expand round
  trip. A helper return value alone is not sufficient.
- Include adversarial variants: empty and malformed structures, nested and
  adjacent markers, multiple candidates, whitespace and `<br>` differences,
  uncertain lookalikes, and content that must remain untouched.
- Exercise the production selection and event path. Avoid tests that call an
  exported helper directly when the failure can occur in candidate discovery,
  early-return behavior, event wiring, iframe messaging, or state cleanup.
- For iframe or size-dependent behavior, use a controllable `ResizeObserver`,
  mocked measurements, real toggle events, and flushed animation frames. Assert
  both growth and shrinkage. Remember that jsdom has no layout engine, so a
  jsdom-only height assertion is not evidence of WebKit layout behavior.
- Prefer a small checked-in corpus of redacted real provider fixtures over many
  simplified inline strings. Add the fixture that exposed each regression to
  that corpus.

Before declaring a regression fixed, explain which test would have failed on
the broken implementation. Temporarily reverting or otherwise perturbing the
fix is encouraged when practical to prove the test is sensitive to the defect.

## Native interaction design

When a desktop interaction is expected to behave like a system context menu,
use Tauri's native menu APIs and the platform menu implementation. Do not
recreate it with a web popover, an invisible or synthetic anchor element, or
manually translated browser pointer coordinates merely to approximate native
behavior. Webview, window, screen, logical, and physical coordinates can diverge
under display scaling, title bars, portals, and multi-monitor layouts.

Before adding a new interaction, search for an existing native implementation
in the repository and reuse its command, event-routing, validation, and error
feedback patterns where appropriate. If the product intentionally needs a web
menu, document that constraint and test positioning under scaling and near all
viewport edges.

For native context-menu actions:

- Validate action data in Rust before constructing a menu. Treat menu item IDs
  as untrusted event input, encode arbitrary values without delimiter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DakiaMail/dakia-desktop](https://github.com/DakiaMail/dakia-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
