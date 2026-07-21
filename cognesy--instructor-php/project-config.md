---
trigger: always_on
description: - **CONTRIBUTING.md** - Development workflows, package management, and contribution guidelines
---

# Key Reference Files

## Monorepo Root

- **CONTRIBUTING.md** - Development workflows, package management, and contribution guidelines
- **CONTENTS.md** - Project structure and package overview
- **README.md** - Main project documentation
- **HARD REFERENCE** - `CONTRIBUTING.md` is the source of truth for QA execution (`composer qa`, `qa:*`) and contribution workflow details.

## Individual Subpackages

Each package in `packages/` may contain:
- **README.md** - Package-specific documentation
- **OVERVIEW.md** - Package overview and architecture
- **INTERNALS.md** - Implementation details and internal structure
- **CHEATSHEET.md** - Quick reference for package usage

# Code Style

- Use strict types and type hints for arguments and return values everywhere
- Do not nest control structures (if, loops) beyond 1 level, exceptionally 2 levels
- Use `match` for complex conditionals - avoid `if`/`else if` chains or `switch` statements; avoid ternary operators
- Prefer immutable data structures and functional programming paradigms
- Avoid using arrays as collections - use dedicated collection classes
- Avoid exceptions for control flow - do not wrap everything in try/catch; either let exceptions bubble up or use monadic Cognesy\Utils\Result\Result for error handling if error needs to be handled on the same level
- Use namespaces and PSR-4 autoloading

## Agents Package Style

- Data classes are `readonly final class` — all properties readonly, all mutations return new instances via `with*()` methods
- Interfaces use `Can-` prefix (`CanUseTools`, `CanEmitAgentEvents`, `CanInterceptAgentLifecycle`)
- Hooks use `Hook` suffix (`StepsLimitHook`, `ErrorPolicyHook`)
- Accessors have no prefix: `state()`, `execution()`, `currentStep()` — not `getState()`
- Mutators use `with` prefix and return `self`: `withState()`, `withCurrentStep()`
- Named constructors for defaults: `::empty()`, `::fresh()`; for hydration: `::fromArray()`
- Minimal docblocks — only on class/interface declarations; methods are self-documenting via types and naming
- Method order: constructors → lifecycle/transitions → accessors → mutators → private helpers → serialization (`toArray`/`fromArray`)
- Enums are string-backed for serialization; put logic (priority, comparison) as methods on the enum itself
- Use `match(true) { ... }` for multi-condition branching
- Use `$this->field ?? Default::empty()` pattern ("ensure pattern") to provide defaults instead of null-checking

# Design Principles

- Always start with extremely simple code, refactor when it is required - do not over-engineer solutions (YAGNI)
- Use DDD (Domain Driven Design) principles - aggregate roots, value objects, entities, repositories, services
- Use Clean Code and SOLID principles
- Use interfaces for contracts, avoid concrete class dependencies
- Prefer using monadic designs for complex fragments of code - e.g. to avoid null checks and make the code cleaner and simpler.

## Agents Package Architecture

### Mental Model

- **AgentLoop** is a step-based iterator: each step = one LLM inference + tool execution cycle
- Loop lifecycle: `beforeExecution` → [`beforeStep` → `handleToolUse` → `afterStep` → check `shouldStop()`]* → `afterExecution`
- The loop yields intermediate states via `iterate()` (generator pattern) — callers can observe/persist between steps

### State Layering

- **AgentState** = session-level (persistent across executions): agentId, messages, metadata, optional `ExecutionState`
- **ExecutionState** = transient per-run: executionId, status, steps, continuation signals — null between executions
- **AgentStep** = immutable snapshot of one step: input/output messages, inference response, tool executions, errors
- **StepExecution** = wraps AgentStep with timing and continuation metadata — keeps step data immutable while tracking execution details
- State updates are always atomic — `withCurrentStepCompleted()` chains through the layers to prevent partial/inconsistent states
- `AgentLoop` auto-resets terminal executions (completed/failed) on entry — no manual `forNextExecution()` needed

### Stop/Continuation

- **StopSignal** = value object with reason (enum) + message + context + source
- **StopReason** enum has priority ordering for conflict resolution (ErrorForbade > StopRequested > StepsLimitReached > ...)
- **ExecutionContinuation** aggregates stop signals; `shouldStop()` = has signals AND no continuation override
- **AgentStopException** is a control-flow exception (not an error) — caught in loop, converted to StopSignal

### Invariant vs Variant Behavior

The boundary between `\Core` and `\Hook` is **whether the behavior is optional**:

- **Core state transitions** = invariant. Always happens, can't be removed, not configurable. Folded into `with*()` methods on state objects. Examples: `withCurrentStep()` routes step output to the correct message section; `withCurrentStepCompleted()` archives the step; `AgentLoop.ensureNextExecution()` auto-resets terminal executions on entry.
- **Hooks** = variant. Configurable, optional, composable via builder. Examples: step limits, token limits, summarization, finish reason stopping. If you could imagine an agent that doesn't need it, it's a hook.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cognesy/instructor-php](https://github.com/cognesy/instructor-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
