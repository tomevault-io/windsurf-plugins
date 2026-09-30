---
trigger: always_on
description: UIKitUltra is the canonical repository/package/framework brand; the canonical public Swift library/module is `Ultra`. The repository migrated from the legacy `UIKitPlus` name; that rename is historical migration context rather than an active identity transition. The existing public `U*` prefix is retained and means **Ultra**. Naming authority: `.agent/PRODUCT_IDENTITY.md`.
---

# UIKitUltra - Agent Governance

## Repository Identity

UIKitUltra is the canonical repository/package/framework brand; the canonical public Swift library/module is `Ultra`. The repository migrated from the legacy `UIKitPlus` name; that rename is historical migration context rather than an active identity transition. The existing public `U*` prefix is retained and means **Ultra**. Naming authority: `.agent/PRODUCT_IDENTITY.md`.

UIKitUltra is a declarative, protocol-oriented UI framework whose Apple implementation remains built directly on UIKit/AppKit and whose accepted expansion architecture targets additional real-native backends plus an owned TUI runtime. Backend naming/architecture is routed through `ARCH_INDEX.md`; one-time rename mechanics stay in `.artifacts/**`.

Core characteristics:
- fluent `Self`-returning API composition;
- extension-driven feature growth;
- reference-semantic reactive state (`State`, `InnerState`, mapped/bound states);
- deferred/state-backed layout constraints;
- explicit UIKit/AppKit platform bridges;
- native-platform behavior as the preferred implementation substrate.

Architecture is frozen by default. Contract-changing architecture work requires explicit maintainer approval.

## Authority Hierarchy

When stable documents conflict, higher authority wins. **This hierarchy is for conflict resolution, not a preload list; do not read every owner by default.**

1. `.agent/SYSTEM_RULES.md` - global operational/engineering invariants.
2. `.agent/WORKFLOW.md`, `.agent/DEVELOPMENT_ORCHESTRATION.md`, `.agent/PARALLEL_DEVELOPMENT.md` when linked worktrees are actually in scope, `.agent/ARTIFACTS_WORKFLOW.md`, and `.agent/COMMIT_RULES.md` - development/orchestration/worktree/artifact/Git workflow.
3. `.agent/PRODUCT_IDENTITY.md` for canonical brand/module naming, then `.agent/ARCH_INDEX.md` and the owning `.agent/architecture/*.md` files for technical architecture and architecture-ID authority.
4. `.agent/STYLE_GUIDELINES.md`, `.agent/DSL_SAFETY_RULES.md`, `.agent/EXTENSION_RULES.md`, and applicable focused policy/skill docs - implementation conventions inside architecture boundaries.
5. `.agent/PROJECT_MEMORY.md` and `.agent/SOURCE_MAP.md` - durable current-state/navigation facts.
6. `.agent/TASKS.md`, `.agent/TODO.md`, `.agent/TECH_DEBT.md`, `.agent/TASKS_ARCHIVE.md`, and `.agent/STATE_VNEXT_PLAN.md` - active work, future work, debt, history, and State planning.
7. `.agent/CONTEXT_LOADING_RULES.md`, `.agent/CONTEXT_BUDGET.md`, `.agent/PUBLIC_CONTENT_IDEAS.md`, `.agent/SKILL_INDEX.md`, `.agent/skills/*`, `.agent/TEMPLATE_INDEX.md`, and `.agent/templates/*` - progressive routing and focused operational guidance.

`.artifacts/**` is disposable Git-ignored working memory and never stable authority.

Architecture owners win on UIKitUltra semantics. Link to owners instead of duplicating full contracts in routing/workflow docs.

## Mandatory Development Workflow

**PLAN -> IMPLEMENT -> AUDIT**

- Non-trivial work requires current-repository research and a reviewed plan before production mutation.
- Use `.agent/DEVELOPMENT_PHASES.md` for UIKitUltra-specific phase mechanics.
- For non-trivial iterative LLM-assisted work, load `.agent/DEVELOPMENT_ORCHESTRATION.md` and `.agent/ARTIFACTS_WORKFLOW.md`.
- Large/cognitively dense implementation or correction work is decomposed into numbered surgical task files; detailed mechanics stay in those files and the executor receives one short coordinator prompt.
- Executor reports are evidence, never proof. Independently inspect actual source/diff/Git and relevant architecture owners.
- If a materially reviewed assumption fails during implementation, stop that path and re-plan rather than silently widening scope.
- Commit and push are separate explicit gates; passing AUDIT does not authorize either.

## UIKitUltra Engineering Routing

For source/API work:

1. Use `.agent/ARCH_INDEX.md` to select **one primary architecture owner**.
2. Add at most two supporting architecture docs only when the task actually crosses those boundaries; `LAYER_MODEL.md` is not a ceremonial mandatory read.
3. Inspect analogous existing UIKitUltra source before designing a new approach; consult legacy UIKitPlus history only when it is actually relevant.
4. Preserve the native-first two-stage pattern where applicable: thin native/backend wrapper first, UIKitUltra convenience layer second.
5. Preserve fluent/reference/state/extension/platform invariants owned by the selected architecture docs.

Application state-placement work additionally routes through `.agent/architecture/APPLICATION_STATE_OWNERSHIP.md` and its `AO*` invariants.

## Mandatory Context Budget

Normal work:
- starts here;
- uses `ARCH_INDEX.md` to select one primary architecture owner;
- keeps at most 3 active architecture docs by default: that primary owner plus at most 2 supporting docs only when genuinely needed;
- loads at most one operational skill by default;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swiftstream/UIKitUltra](https://github.com/swiftstream/UIKitUltra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
