---
trigger: always_on
description: Preserve correctness, security, data integrity, and explicit requirements. Within those constraints, optimize
---

# AGENTS.md

## Priorities

Preserve correctness, security, data integrity, and explicit requirements. Within those constraints, optimize
for maintainability, readability, and testability. Add extensibility, flexibility, and reuse only when current
needs justify them.

Use design principles as judgment aids, not a checklist of patterns to implement. Resolve trade-offs in favor
of clear behavior and lower overall complexity.

## Working approach

- Read applicable instructions and relevant implementation, callers, and tests. Expand context as dependencies
  or uncertainty require; load documentation and skills only when their scope matches the task.
- Use commands verified in repository scripts, configuration, or CI. Follow sound local conventions; introduce
  a different pattern to address a concrete limitation, not a stylistic preference.
- For cross-cutting or risky work, identify intended behavior, affected contracts, and verification before
  editing. Make straightforward changes directly.
- Resolve ambiguity from contracts and repository evidence. Ask only when remaining uncertainty materially
  affects behavior, scope, or data safety; otherwise use the simplest consistent interpretation.
- Within the authorized scope, implement and verify the change. Run checks and fix introduced failures without
  repeated approval in confirmed isolated environments. Before unfamiliar or potentially state-changing
  commands, confirm the target environment and expected side effects are within the authorized scope. Changes
  to shared or external state require explicit authorization; a command named `test` is not proof of isolation.
- Preserve unrelated work. Production actions, destructive data operations, and destructive Git operations
  require explicit authorization beyond permission to edit code.

## Design and implementation

### Simplicity and abstraction

- **Occam's razor / KISS:** Choose the least complex sufficient solution: fewer assumptions, concepts, states,
  dependencies, and indirections. Reduce understanding and change costs, not line count.
- **YAGNI:** Add only capabilities required now. Do not prebuild configuration, extension points, or
  frameworks. Necessary safety checks and tests are not speculative work.
- **DRY:** Give each business rule one authoritative representation. Share stable knowledge, not merely
  similar syntax; keep independently changing concepts separate.
- An abstraction must reduce complexity for its callers, consolidate stable knowledge, or isolate an actual
  variation. Moving code behind another name is not enough.
- Prefer standard-library and existing project capabilities. Add dependencies only when their benefits justify
  their maintenance cost; use established implementations for security-sensitive primitives.

### Boundaries and contracts (SOLID)

- **SRP:** Group code by its reason to change; split independent responsibilities, not cohesive logic to
  satisfy arbitrary size limits.
- **OCP:** Extend behavior at demonstrated variation points; repair flawed abstractions instead of preserving
  them behind extra layers.
- **LSP:** Preserve behavioral contracts, including invariants and failure semantics. Do not strengthen
  preconditions or weaken postconditions.
- **ISP:** Shape small, cohesive interfaces around consumer needs, not every capability of an implementation.
- **DIP:** Separate business policy from volatile infrastructure through explicit boundaries; do not create an
  interface for every type.
- **LoD:** Depend on direct collaborators' public contracts, not their internal object graphs. Avoid
  forwarding layers that merely disguise coupling.

### One fact, one owner

- Every fact has exactly one representation that may advance it. Every other representation may encode,
  cache, project, or render that fact, and none of them may create a competing transition.
- When a fact appears in several places, name the owner before changing any of them. Storage records, wire
  values, caches, read models, and user-interface state are projections; a projection that can also originate
  a change is a second owner, whatever it is called.
- A rule about which representation wins when two disagree is evidence that both can advance independently.
  Repair the ownership instead of adding the arbitration rule.
- Ownership is a property of the fact, not of the layer. A single implementation still deserves a boundary
  when it owns a necessary guarantee; a boundary that owns nothing is a forwarding layer.

### Readability and state (Zen of Python)

- Use the host language's idioms. Prefer explicit dependencies, flat control flow, readable spacing, and
  coherent namespaces over implicit magic or clever compression.
- Represent necessary complexity behind clear boundaries. Keep justified exceptions local and prefer practical
  clarity over rigid uniformity.
- Keep mutable state minimal and separate business decisions from external I/O.
- Prefer one clear path per behavior. Simplify hard-to-explain logic without fragmenting cohesive code into
  tiny helpers.
- Make failures explicit; suppress only specific expected errors allowed by the contract. Never turn
  unexpected failure into apparent success.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tangerg/typesafe-sdk-go](https://github.com/Tangerg/typesafe-sdk-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
