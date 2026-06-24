---
trigger: always_on
description: Conventions for any AI agent (Codex, Claude Code, Cursor, Aider, …) editing this repository.
---

# AGENTS.md

Conventions for any AI agent (Codex, Claude Code, Cursor, Aider, …) editing this repository.
This file is the single source of truth; `CLAUDE.md` points here.

## The non-negotiable rule

**Spec-Driven Development.** The specification in [`spec/spec.md`](spec/spec.md) is the contract.
Code that diverges from the spec is a bug.

The workflow for any non-trivial change is:

1. Read the relevant spec file(s).
2. **Update the spec first.** Get human approval if the change is substantive.
3. Only then modify code to match.
4. Run `npm run typecheck && npm test && npm run lint` before declaring done.

Trivial changes (typo, obvious one-line fix, dependency bump) can skip step 2. When in doubt,
change the spec first.

## Project layout

```
veche/
├── spec/
│   └── spec.md            ← single monolithic contract; sections 1–19 +
│                            `## Partition: <name>` per feature
│                            (persistence, agent-integration,
│                            committee-protocol, meeting, web-viewer, install)
├── src/                   ← implementation, hexagonal + vertical slices
│   ├── features/<slice>/  ← one folder per Partition
│   ├── adapters/inbound/mcp/     ← MCP server (stdio)
│   ├── adapters/inbound/cli/     ← human-operator CLI (list, show, watch, install)
│   │                               + renderers + lib helpers
│   ├── adapters/inbound/web/     ← `veche watch` HTTP/SSE server
│   ├── infra/             ← composition root (DI wiring)
│   ├── bin/veche-server.ts  ← stdio MCP entrypoint
│   └── bin/veche.ts         ← CLI entrypoint
├── skills/veche/          ← canonical SKILL.md + optional UI metadata
├── examples/              ← sample .mcp.json, config.json
├── scripts/               ← repo helpers (e.g. sdd-approve-all.sh)
├── .sdd/                  ← SDD config + finalised approval plans
├── dist/                  ← build output (not committed locally)
└── node_modules/
```

Navigation: open `spec/partitions/<name>.md` for the slice you're touching.
Each partition file is self-contained (Context, Glossary, Surfaces,
Behaviours, Contracts, Invariants, Policies, Constraints, Migrations,
Deltas, Open questions, Assumptions, Out-of-scope). The thin
[`spec/spec.md`](spec/spec.md) is just an index.

## Architecture

**Hexagonal + Vertical Slices.**

- Each feature under `src/features/<feature>/` owns its domain, ports, application code, and
  (if it owns ports) adapters.
- Domain code imports only from `shared/`. Never from `adapters/` or `infra/`.
- Application code imports from own `domain/` and `ports/`. Never from another feature's
  domain directly — cross-slice imports go through the barrel `index.ts`.
- Adapters implement a port. They never import another adapter.
- `infra/` is the composition root; it may import anything.

Dependency table lives in [`spec/partitions/`](spec/partitions/) — one file per feature; the [`spec/spec.md`](spec/spec.md) index lists them.

## Naming conventions

- **Entities** are pure classes — no ORM/framework decorators. File name = entity name
  (`Meeting.ts`, `Participant.ts`).
- **Ports** are interfaces or abstract classes with suffix `Port` (`AgentAdapterPort`,
  `MeetingStorePort`). File name = port name.
- **Adapters** are named `<Technology><Port>` (`CodexCliAgentAdapter`, `InMemoryMeetingStore`,
  `FileMeetingStore`).
- **Use cases** are classes with suffix `UseCase` (`StartMeetingUseCase`). File name = class
  name.
- **Errors** are domain-level classes ending in `Error` (`MeetingNotFound`,
  `AdapterTurnTimeout`). Framework error types stay at the adapter boundary.
- **Variables**
  - Booleans as questions: `isActive`, `hasDropped`, `canAcceptTurn`.
  - Query methods return values, zero side effects — noun-named (`Meeting.transcript()`).
  - Command methods return `void`, have side effects — verb-named (`Meeting.addMessage(...)`).
  - Predicates — question-named (`Round.isSettled()`).

One file = one public class/type. Private helpers may share a file only if used nowhere else.

## Coding rules

- **Do not add code comments by default.** Well-named identifiers document intent. Add a
  comment only when the *why* is non-obvious (hidden constraint, subtle invariant, workaround
  for a specific CLI bug). Never explain *what* the code does.
- **Do not add features, refactors, or abstractions beyond what the task requires.** Three
  similar lines is better than a premature abstraction.
- **Do not add validation for impossible states.** Validate at system boundaries (MCP input,
  CLI output), trust internal code.
- **Do not add backwards-compat shims.** Change the code instead.
- **Do not delete unfamiliar state.** Investigate before overwriting.
- **Do not bypass hooks (`--no-verify`) or sign-off** unless the user explicitly asks.

## Tests

- Unit tests live next to the code: `Foo.test.ts` next to `Foo.ts`.
- Integration tests for a slice live under `src/features/<slice>/.../__tests__/`.
- E2E tests that spawn real CLI subprocesses live under `src/e2e/` and MUST be gated on
  `process.env.VECHE_E2E === '1'` (use `const d = runE2e ? describe : describe.skip;`).
  They are skipped by default because they consume tokens.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyberash-dev/veche](https://github.com/cyberash-dev/veche) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
