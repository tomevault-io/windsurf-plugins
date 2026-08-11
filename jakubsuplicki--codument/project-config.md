---
trigger: always_on
description: This is the canonical cross-agent guide for working on Codument. `CLAUDE.md` may exist for Claude-specific tooling, but this file is the shared contract.
---

# Codument Agent Guide

This is the canonical cross-agent guide for working on Codument. `CLAUDE.md` may exist for Claude-specific tooling, but this file is the shared contract.

## Product Direction

Codument is a docs-backed delivery workflow for AI coding agents. The core loop is:

```text
grill -> plan -> approve -> implement -> verify -> document -> review -> commit -> repeat
```

Do not treat docs as an afterthought. The docs are the control plane that lets an agent resume work without relying on chat history.

## Build & Test

```bash
npm run typecheck
npm run build
npm test
```

## Project Structure

- `src/commands/` — CLI commands (`init`, `scan`, `update`)
- `src/lib/` — Core libraries for profiles, registry, scaffolding, detection, codemods, markers, and versioning
- `src/hooks/` — Claude profile hook script
- `skills/` — Workflow skills shipped with the package
- `agents/` — Claude profile subagent definitions
- `rules/` — Claude profile path-scoped rule templates
- `templates/` — Documentation templates copied on init
- `tests/` — Node test runner tests

## Working Rules

- Use the approved feature plan before source edits.
- Keep implementation slices small enough to review and commit independently.
- Ask which doc owns a file with `codument context --file <path> --owner` before and after touching it — one line, from the same resolver the gate uses. Read `docs/.registry.json` when you need the whole map, not to answer one question about one file.
- Update mapped docs as part of the same change.
- Keep docs compact and durable; do not preserve working chatter.
- Use conventional commit prefixes.

<!-- codument:start -->
## Codument Delivery Workflow

### Core loop
Use Codument as the durable control plane for agent-led engineering work:

1. Grill the request against existing docs, code, ADRs, and project language.
2. Plan the feature in docs before changing source code.
3. Wait for explicit user approval before implementation.
4. Implement one planned step at a time.
5. Build the strongest practical feedback loop, preferring red-green-refactor when it fits.
6. Update the mapped docs + `docs/.registry.json` as part of the same step: materialize each new source file with `codument map materialize`; when a symbol moved, update its doc at intent altitude if a contract changed, or `codument ack` a pure-internal refactor (never a junk mirror edit to clear the gate).
7. Review the diff against the approved plan, tests, docs, and architecture.
8. Commit focused work with a conventional commit, authored as the user with no AI `Co-Authored-By` trailer.
9. Move to the next unchecked step.

If a codument command's quoted argument comes back refused as several arguments — `--reason "one two three"` rejected as three — the launcher split it before codument saw argv, so no quoting fixes it. Run the CLI a different way for the rest of the session: `npx codument …`, or `node node_modules/codument/dist/cli.js …`. Seen with `bunx` on Windows.

### Quality bar
Aim for the best-effort, durable solution, not the first plausible one. Before calling a plan or a step done, zoom out and check it adversarially — where is this half-baked, what did I assume, what would break it. Resolve issues yourself; pull the user in only for a genuinely load-bearing, unconfirmed call (the assumption gate below), not for work that should just happen.

### Implementation discipline
Write the least code that solves the understood problem — the over-engineering guard that complements the quality bar above. Before adding code, check whether it needs to exist at all: the plan's non-goals may rule it out; the codebase may already have the helper or pattern to reuse; the language, runtime, or an installed dependency may already do it. Only then write new code, and add no dependency or abstraction the plan did not ask for. This runs after you understand the change, never instead of it: the smallest diff in the wrong place is a second bug, not a lazy win.

Fix bugs at the root, not the symptom. A report names one broken path; find the shared function it runs through, guard it once, and check the sibling callers that path implies. A per-caller patch that leaves a sibling caller broken is not a fix.

### Response altitude
Docs have a fixed altitude and so do replies. Lead with the answer: the recommendation, the finding, the verdict goes in the first line or two — never the reasoning that produced it. Then stop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakubsuplicki/codument](https://github.com/jakubsuplicki/codument) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
