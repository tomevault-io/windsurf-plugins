---
trigger: always_on
description: This is a step by step tutorial for learning how to use Workflow.
---

# Workflow Tutorial Introduction

This is a step by step tutorial for learning how to use Workflow.

## Layout

The project has both a starting point, as well as an example of the completed tutorial.

To help with the setup, we have created a few helper modules:

- `tutorial-views`: A set of 3 views for the 3 screens we will be building, `Welcome`, `TodoList`,
  and `TodoEdit`.
- `tutorial-base`: This is the starting point to build out the tutorial.
- `tutorial-final`: This is an example of the completed tutorial - could be used as a reference if
  you get stuck.

## Tutorial Steps

- [Tutorial 1](Tutorial1.md) - Single view backed by a workflow
- [Tutorial 2](Tutorial2.md) - Multiple views and navigation
- [Tutorial 3](Tutorial3.md) - State throughout a tree of workflows
- [Tutorial 4](Tutorial4.md) - Refactoring
- [Tutorial 5](Tutorial5.md) - Testing

<!-- workflow-kotlin-AGENTS-injection:START -->
# Workflow Kotlin - AI Agent Guide

This is **Square's workflow-kotlin** — a Kotlin multiplatform application framework that provides
unidirectional data flow and state machine primitives for building composable, testable apps.

- Written in Kotlin, targeting JVM and JS
- Used primarily for Android app development (but not Android-specific at the core)
- Repository: https://github.com/square/workflow-kotlin
- Documentation: https://square.github.io/workflow/

## AI Skills Index

This repository includes detailed AI skills with step-by-step guidance for common tasks.
**Read the relevant skill before starting work** — they contain templates, critical rules, and
correct API usage.

| Task | Skill | Location |
|---|---|---|
| Create a new Workflow | `create-workflow` | `workflow-core/.agents/skills/create-workflow/SKILL.md` |
| Write unit tests (testRender) | `workflow-testing` | `workflow-testing/.agents/skills/workflow-testing/SKILL.md` |
| Write integration tests (renderForTest) | `workflow-integration-testing` | `workflow-testing/.agents/skills/workflow-integration-testing/SKILL.md` |

### When to use which skill

- **Creating a workflow?** Read `create-workflow` first — it covers `StatefulWorkflow` vs
  `StatelessWorkflow`, `eventHandler`, Workers, child workflows, and naming conventions.
- **Writing tests for render logic?** Read `workflow-testing` — it covers `testRender`,
  `expectWorkflow`, `expectWorker`, `verifyAction`, and `verifyActionResult`.
- **Writing tests for multi-step flows or async behavior?** Read `workflow-integration-testing` —
  it covers `renderForTest`, `WorkflowTurbine`, `awaitNextRendering`, and `awaitNextOutput`.

## Architecture

| Module | Purpose |
|---|---|
| `workflow-core` | Core types: `StatefulWorkflow`, `StatelessWorkflow`, `Worker`, `WorkflowAction`, `Snapshot` |
| `workflow-runtime` | Runtime engine: `renderWorkflowIn` starts and manages workflow trees |
| `workflow-testing` | Test harnesses: `testRender` (unit), `renderForTest` / `WorkflowTurbine` (integration) |
| `workflow-ui/` | Android UI bindings: `Screen`, `AndroidScreen`, `ComposeScreen`, `BackStackScreen` |
| `workflow-rx2` | RxJava2 interop |
| `workflow-tracing` | Runtime tracing and diagnostics |

## Core Concepts

A **Workflow** is a composable state machine with four type parameters:

- `PropsT` — input from parent (like function arguments). Use `Unit` if none needed.
- `StateT` — internal state (only for `StatefulWorkflow`). Use `Unit` or omit via `StatelessWorkflow`.
- `OutputT` — events emitted to parent. Use `Nothing` if the workflow never emits output.
- `RenderingT` — the "view model" returned each render pass. Typically a `Screen` data class.

Workflows form a **tree**: parents render children via `context.renderChild()`. Props flow down,
outputs flow up, and renderings bubble up to the UI layer.

## Key Patterns

### Workflow Types

**StatefulWorkflow** — has internal state that persists across renders:

```kotlin
object MyWorkflow : StatefulWorkflow<MyProps, MyState, MyOutput, MyScreen>() {
  override fun initialState(props: MyProps, snapshot: Snapshot?): MyState = MyState.Initial
  override fun render(renderProps: MyProps, renderState: MyState, context: RenderContext): MyScreen { ... }
  override fun snapshotState(state: MyState): Snapshot? = null
}
```

**StatelessWorkflow** — rendering is a pure function of props:

```kotlin
object MyWorkflow : StatelessWorkflow<MyProps, MyOutput, MyScreen>() {
  override fun render(renderProps: MyProps, context: RenderContext): MyScreen { ... }
}
```

**Factory functions** for inline definitions:

```kotlin
val myWorkflow = Workflow.stateful<MyProps, MyState, MyOutput, MyScreen>(
  initialState = { props -> MyState.Initial },
  render = { props, state -> MyScreen(...) }
)

val myWorkflow = Workflow.stateless<MyProps, MyOutput, MyScreen> { props -> MyScreen(...) }
```

### Event Handling

**Prefer `eventHandler`** over raw `actionSink.send()` — it provides Compose stability via the
required `name` parameter:

```kotlin
override fun render(renderProps: MyProps, renderState: MyState, context: RenderContext): MyScreen {
  return MyScreen(
    onClicked = context.eventHandler("onClicked") {
      state = state.copy(loading = true)
    },
    onItemSelected = context.eventHandler("onItemSelected") { item: Item ->
      setOutput(MyOutput.Selected(item))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [square/workflow-kotlin](https://github.com/square/workflow-kotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
