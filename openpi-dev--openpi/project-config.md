---
trigger: always_on
description: OpenPI is a Pi-native capability layer, not a second agent operating system. Extend Pi at its existing seams and preserve its lifecycle, vocabulary, and sources of truth.
---

# OpenPI agent contract

OpenPI is a Pi-native capability layer, not a second agent operating system. Extend Pi at its existing seams and preserve its lifecycle, vocabulary, and sources of truth.

## Design stance

### Build for model leverage

- Prefer capabilities whose usefulness increases as models improve.
- Give the model small, orthogonal mechanisms and high-fidelity feedback. Let the model decide strategy, decomposition, delegation, and synthesis.
- Encode safety and execution invariants in the runtime. Do not encode a preferred reasoning process, keyword router, or fixed orchestration workflow unless correctness requires it.
- Before adding a framework abstraction, ask whether a stronger model could use the underlying Pi primitive directly. If yes, expose or strengthen that primitive instead.

### Keep ownership clear

The model owns judgment:

- understanding intent and choosing an approach;
- deciding whether and how to delegate;
- adapting to evidence and synthesizing results;
- deciding when user input is needed.

The runtime owns enforceable facts:

- permissions, trust, isolation, and capability boundaries;
- concurrency, call, time, and resource limits;
- lifecycle, cancellation, cleanup, persistence, and recovery;
- atomicity, idempotency, replay safety, and fail-closed outcomes;
- observable execution state and exact terminal evidence.

Do not rely on prompts for runtime invariants. Do not move model judgment into a rigid state machine merely because it is easier to test.

### Preserve Pi-native composition

- Pi remains the source of truth for Sessions, provider/model selection, Skills, project Trust, and ordinary tools.
- Reuse Pi events, messages, tools, extension hooks, and session lifecycle before introducing parallel storage or control planes.
- Keep model-facing schemas compact and progressively disclose specialized capabilities and detailed instructions only when needed.
- Treat canonical execution facts, model-visible context, and operator-facing UI as distinct projections. Never infer completion from labels or presentation state.
- Prefer one general composable capability over several workflow-specific commands.

### Keep subagents simple

- Subagents are in-process Pi sessions and inherit the parent model and thinking level unless an explicit, reviewed override applies.
- The parent model owns delegation and synthesis. Child capabilities are a fail-closed intersection and do not silently widen parent authority.
- Background work should report completion through events without keeping an otherwise idle interactive parent turn alive. A synchronous wait is for explicit synchronization, not the default orchestration pattern.
- Do not add recursive teams, an external CLI harness, or a second provider stack without a concrete requirement and a complete identity, authority, lifecycle, and cleanup design.

## Feature decision test

Before implementing a new OpenPI feature, answer:

1. Which Pi primitive or extension seam already owns this lifecycle?
2. Is this model judgment or a runtime invariant?
3. Can the feature be an orthogonal mechanism instead of a prescribed workflow?
4. Will better models use it better without a framework rewrite?
5. What exact evidence distinguishes success, failure, cancellation, and uncertainty?
6. How is authority bounded, inspected, stopped, and cleaned up?

If those answers are unclear, investigate before adding surface area.

## Repository work

- Preserve unrelated and uncommitted user work. Make the smallest scoped change that satisfies the request.
- Follow existing patterns and tests before inventing a new abstraction.
- Run `bun run check` and `bun run test` after making a change. If a relevant validation command does not exist, call that out and propose one.
- When opening or updating a pull request, follow `.github/PULL_REQUEST_TEMPLATE.md` and complete its Problem, Value, Approach, Validation, and Impact sections.
- Avoid explicit return types unless they are necessary. Prefer inference over repeatedly declaring types.
- Treat `as any` as an absolute last resort; use real type safety.
- Keep user-visible behavior, model-visible context, and persisted/runtime state tests separate when the distinction matters.
- Runtime provenance: before diagnosing installed behavior, provider compatibility, a manual Pi smoke, or any UI result, read README section “开发运行时：区分 npm 与当前源码”. Prove both the checkout revision and the single OpenPI source reported by `pi list` before reasoning from source code.
- Preserve ignored local evidence as user work. Never use `git clean -fdx`; ignored benchmark runs, logs, and harnesses may be the only local copy.

### Knowledge and evidence

- Use GitHub Issues for discussion and work tracking. Preserve a reusable conclusion in the appropriate `docs/` category and link the Issue and record both ways.
- Keep facts, inferences, recommendations, unknowns, protocols, and validated results distinct. A research recommendation or complete-looking design is not a project constraint without an accepted Decision.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openpi-dev/openpi](https://github.com/openpi-dev/openpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
