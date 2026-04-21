---
trigger: always_on
description: provides only infrastructure. The core principle: **ZERO hardcoded roles.**
---

# Gas City

Gas City is an orchestration-builder SDK — a Go toolkit for composing
multi-agent coding workflows. It extracts the battle-tested subsystems from
Steve Yegge's Gas Town (github.com/steveyegge/gastown) into a configurable
SDK where **all role behavior is user-supplied configuration** and the SDK
provides only infrastructure. The core principle: **ZERO hardcoded roles.**
The SDK has no built-in Mayor, Deacon, Polecat, or any other role. If a
line of Go references a specific role name, it's a bug.

You can build Gas Town in Gas City, or Ralph, or Claude Code Agent Teams,
or any other orchestration pack — via specific configurations.

**Why Gas City exists:** Gas Town proved multi-agent orchestration works,
but all its roles are hardwired in Go code. Steve realized the MEOW stack
(Molecular Expression of Work) was powerful enough to abstract roles into
configuration. Gas City extracts that insight into an SDK where Gas Town
becomes one configuration among many.

## Development approach

**TDD.** Write the test first, watch it fail, make it pass. Every package
has `*_test.go` files next to the code. Integration tests that need real
infrastructure (tmux, filesystem) go in `test/` with build tags.

**The spec is a reference, not a blueprint.** When the DX conflicts
with the spec, DX wins. We update the spec to match.

## Architecture

**Work is the primitive, not orchestration.** Gas City's orchestration
is a thin layer atop the MEOW stack (beads → molecules → formulas).
The work definition and tracking infrastructure is what matters; the
orchestration shape is configurable on top.

### The nine concepts

Gas City has five irreducible primitives and four derived mechanisms.
Removing any primitive makes it impossible to rebuild Gas Town. Every
mechanism is provably composable from the primitives.

**Five primitives (Layer 0-1):**

1. **Agent Protocol** — start/stop/prompt/observe agents regardless of
   provider. Identity, pools, sandboxes, resume, crash adoption.
2. **Task Store (Beads)** — CRUD + Hook + Dependencies + Labels + Query
   over work units. Everything is a bead: tasks, mail, molecules, convoys.
3. **Event Bus** — append-only pub/sub log of all system activity. Two
   tiers: critical (bounded queue) and optional (fire-and-forget).
4. **Config** — TOML parsing with progressive activation (Levels 0-8 from
   section presence) and multi-layer override resolution.
5. **Prompt Templates** — Go `text/template` in Markdown defining what
   each role does. The behavioral specification.

**Four derived mechanisms (Layer 2-4):**

6. **Messaging** — Mail = `TaskStore.Create(bead{type:"message"})`.
   Nudge = `AgentProtocol.SendPrompt()`. No new primitive needed.
7. **Formulas & Molecules** — Formula = TOML parsed by Config. Molecule =
   root bead + child step beads in Task Store. Wisps = ephemeral molecules.
   Orders = formulas with gate conditions on Event Bus.
8. **Dispatch (Sling)** — composed: find/spawn agent → select formula →
   create molecule → hook to agent → nudge → create convoy → log event.
9. **Health Patrol** — ping agents (Agent Protocol), compare thresholds
   (Config), publish stalls (Event Bus), restart with backoff.

### Layering invariants

1. **No upward dependencies.** Layer N never imports Layer N+1.
2. **Beads is the universal persistence substrate** for domain state.
3. **Event Bus is the universal observation substrate.**
4. **Config is the universal activation mechanism.**
5. **Side effects (I/O, process spawning) are confined to Layer 0.**
6. **The controller drives all SDK infrastructure operations.**
   No SDK mechanism may require a specific user-configured agent role.

### Progressive capability model

Capabilities activate progressively via config presence.

| Level | Adds                    |
|-------|-------------------------|
| 0-1   | Agent + tasks           |
| 2     | Task loop               |
| 3     | Multiple agents + pool  |
| 4     | Messaging               |
| 5     | Formulas & molecules    |
| 6     | Health monitoring       |
| 7     | Orders             |
| 8     | Full orchestration      |

## Architecture specs

Read **`specs/architecture.md`** before touching:

- `internal/api/` (HTTP + SSE API layer)
- `cmd/gc/` (CLI) — especially anything that constructs events,
  calls `apiroute.go:apiClient()`, or uses
  `internal/api/genclient`
- `internal/events/` (event bus, registry)
- `internal/extmsg/` (external-messaging emitters)
- Anything that affects `internal/api/openapi.json`,
  `docs/schema/openapi.json`, or the generated TS types under
  `cmd/gc/dashboard/web/src/generated/`

Load-bearing invariants enforced by CI (violating any fails the
build; full rationale is in the spec):

- **Object model at the center.** `internal/{beads, mail, convoy,
  formula, agent, events, session, sling, ...}` is the canonical
  domain. The CLI (`cmd/gc/`) and the HTTP+SSE API
  (`internal/api/`) are projections over it. Neither re-implements
  domain logic.
- **Typed wire.** No hand-written JSON on any HTTP or SSE wire
  path; no `map[string]any` or `json.RawMessage` on wire types
  (two documented exceptions live in the spec). All endpoints are
  Huma-registered; the OpenAPI spec is generated, never
  hand-written (`TestOpenAPISpecInSync`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gastownhall/gascity](https://github.com/gastownhall/gascity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
