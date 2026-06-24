---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

**File-size rule:** this document must stay under 250 lines. When you add a new rule or section, tighten or remove existing content first — never just append. A bloated guide is read less, not more. Keep prose terse, examples one-line, and prefer references to inline duplication.

## Product vision

Voleeo is an **AI-native local toolkit with MCP bridge**. Every feature serves one goal: an AI agent (via MCP) and a developer share the same view of the system. UI, data models, and IPC are designed for both human and machine consumers.

## Model routing (delegated sub-work only)

Applies to subagents/workflows spawned via the Agent/Workflow tools — pick `model` per task. The **main session** model is user-controlled (`/model`) and never auto-switches; this table only routes delegated work. When unsure, step up a tier.

- **Haiku** — mechanical, fully-specified: keyword/file searches, fan-out reads (`Explore`), renames, formatting, log/grep scans, applying a known edit across files. Also commit messages and PR descriptions (use the cheapest tier).
- **Sonnet** — default for real work: feature implementation, code review, writing tests, focused single-file/single-bug debugging, doc edits.
- **Opus** — hard reasoning: architecture/design, cross-crate refactors, render-loop/concurrency/IPC-shape bugs (see numbered rules), security review, ambiguous multi-step tasks, adversarial verification of findings.

## Tool discipline

- **Locate before reading.** Use grep/find to confirm a file holds what you need before `Read`; never read whole directories.
- **Budget the work.** Aim to finish a task in under 20 tool calls. After ~10 calls with no visible progress, stop and explain the blocker rather than pushing on.

## Code quality

**KISS · DRY · YAGNI.** When planning and writing code, default to the simplest thing that works: stdlib or an already-installed dep before new code, one line before fifty, a native/CSS/DB feature before app code. Build nothing for a need that isn't here yet. Factor *real* duplication — never abstract speculatively (no interface with one impl, no config for a constant). Shortest working diff wins; if you can't name why a piece of code must exist, cut it.

**Single Responsibility.** Each file/component/hook does one thing. If you describe it with "and," split it.

**Readability over cleverness.** Functions short enough to read without scrolling. When a file grows, split into a directory: `index.tsx` orchestrates, hooks/sub-components own one concern each. Canonical examples: `components/ApiRequestTree/`, `views/ApiWorkspace/RequestPane/`, `views/ApiWorkspace/ResponsePane/TimelineTab/`.

**File limit: 250 lines** for `.tsx`/`.ts` in `src-web/src/`. **500 lines** for `.rs` files, excluding `#[cfg(test)]` blocks. Stop and split before adding more code.

**Rust test layout.** Public-API / fixture-driven tests live in `crates/<crate>/tests/<topic>.rs` with data under `tests/fixtures/<group>/`, `include_str!`'d instead of inline blobs — `voleeo-import` is the reference (`tests/openapi.rs` + `tests/fixtures/openapi/petstore.yaml`). Tests that exercise **private** items stay inline in `#[cfg(test)] mod tests` (the external `tests/` dir is a separate crate and can't see private/`pub(crate)` items) — never widen visibility just to relocate a test.

**Comments document the non-obvious.** Only comment important or hard logic — if the code already explains itself, skip it. Skip `// Foo` above a `<Foo/>`, section labels for self-evident JSX, JSDoc that restates the signature. Reserve comments for load-bearing context, surprising trade-offs, "why this and not the natural alternative," and footguns that cost someone an afternoon. Useful: `// Drain hops BEFORE pushing the error event — the policy callback fires after the awaiting task throws.` Useless: `// Set the active tab to params` above `setActiveTab("params")`.

**Terse, and audited every edit.** When a comment earns its place, say it in the fewest words — the *why* over the *what*; prune words that restate the code, collapse multi-line `///` blocks a sentence covers. This applies to *existing* comments in any file you touch, not just new ones: tighten or delete padding as part of the same change — editing is the only reliable moment to pay comment debt, not a future "cleanup pass" that never comes.

## Numbered rules

Several below are distilled from confirmed 100%+ CPU bugs in this codebase. Treat violations as bugs.

### React — render-loop prevention

**1. Inline callbacks as props that a child uses in `useEffect` deps cause infinite loops.** Wrap handlers in `useCallback` with the right deps. Real bug: passing `onParamCountChange={(e,t) => setParamCounts({e,t})}` → new fn ref each render → child effect fires → `setState` → re-render → 100% CPU.

**2. `setState` with object literals causes spurious re-renders.** `{ a, b }` is a new reference. Use functional form returning `prev` when values match: `setX(p => p?.a===a && p?.b===b ? p : { a, b })`.

**3. Memoize computed arrays/objects used as effect deps.** `[...a, ...b]` in render body is new every render. Wrap in `useMemo`.

**4. Always pass explicit `useEffect` dep arrays.** Never omit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voleeo/voleeo-api](https://github.com/voleeo/voleeo-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
