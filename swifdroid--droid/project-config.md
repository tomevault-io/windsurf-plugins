---
trigger: always_on
description: `droid` is an independent Git repository implementing the public Swift framework for native Android applications.
---

# Droid - Agent Governance

## Repository Identity

`droid` is an independent Git repository implementing the public Swift framework for native Android applications.

It owns app/lifecycle/manifest/Gradle DSLs, declarative view/state behavior, Android/AndroidX/Material wrappers, SwifDroid-owned listener bridges, Kotlin support code, and local wrapper/reference tooling.

It depends on JNIKit but does not own JNI reference/thread/signature semantics. It has a separate sibling documentation repository, but public documentation does not define Droid behavior.

This repository is designed to be opened directly by LLM/coding agents. Parent `../SwifDroid` governance is optional cross-repository coordination context, not a prerequisite for normal Droid work.

## Authority Hierarchy

When local documents conflict, higher authority wins:

1. `.agent/SYSTEM_RULES.md` - global repository invariants
2. `.agent/WORKFLOW.md`, `.agent/DEVELOPMENT_ORCHESTRATION.md`, `.agent/ARTIFACTS_WORKFLOW.md`, and `.agent/COMMIT_RULES.md` - development/orchestration/artifact/Git workflow
3. `.agent/ARCH_INDEX.md` and the owning `.agent/architecture/*.md` file - technical architecture authority
4. `.agent/CODE_STYLE.md` and applicable specialized guides - implementation policy
5. `.agent/MASTER_PLAN.md` - repository capability roadmap
6. `.agent/OPEN_DECISIONS.md` - unresolved choices only
7. `.agent/PROJECT_MEMORY.md` and `.agent/SOURCE_MAP.md` - durable current-state/navigation facts
8. `.agent/TASKS.md`, `.agent/TODO.md`, `.agent/TECH_DEBT.md`, `.agent/TASKS_ARCHIVE.md` - work state
9. `.agent/CONTEXT_LOADING_RULES.md`, `.agent/PUBLIC_CONTENT_IDEAS.md`, `.agent/SKILL_INDEX.md`, `.agent/skills/*` - progressive context, lazy public-content capture, and focused procedures

Specialized local guides:

- `.agent/ANDROID_CLASS_WRAPPERS.md` - detailed wrapper implementation patterns
- `.agent/MCP_WRAPPER_WORKFLOW.md` - operational MCP wrapper/reference procedure
- `.agent/WRAPPED_CLASSES.md` - navigation/evidence map, not semantic completeness authority
- `.agent/DEVCONTAINER_COMPILATION_CHECK.md` - exact current wrapper completion build gate

`.artifacts/**` is transient plans/evidence/working memory, never stable authority. Root `PLAN.md`/`NEW_WIDGET.md` are task/context documents, not higher architecture authority.

## Mandatory Workflow

Use **PLAN -> IMPLEMENT -> AUDIT** for non-trivial work.

- Define exact behavior/repository/path scope, architecture owner, public API effect, build/runtime evidence, and docs impact before mutation.
- Implement the smallest coherent behavior. Do not bulk-expand adjacent wrappers/API merely because they are nearby.
- Audit source semantics, JNI boundary, state/lifecycle ownership, generated-consumer behavior when relevant, mandatory build gates, documentation impact, and Git state.
- If implementation disproves a reviewed Android/JNI/API assumption, stop that path and re-plan.

### Mandatory Iterative-Development Routing

For non-trivial iterative LLM-assisted work, load `.agent/DEVELOPMENT_ORCHESTRATION.md` and `.agent/ARTIFACTS_WORKFLOW.md`.

- Use `.artifacts/**` as disposable Git-ignored external working memory for research, plans, numbered surgical tasks, execution evidence, verification, reviews, corrections, and chat handoff.
- Large implementation/correction work is decomposed into numbered task files; the executor receives one short generic coordinator prompt and runs approved tasks autonomously in order.
- If `.artifacts/**` is missing, reconstruct current context from stable docs + Git + actual source/build state instead of guessing lost transient state.
- Executor reports are evidence, never proof; independently audit actual source/diff/Git afterward.
- After meaningful research/design/implementation/validation/correction/audit, perform the lazy public-content capture check owned by `.agent/PUBLIC_CONTENT_IDEAS.md`. Decide from already-loaded context first; open the router plus one relevant shard only when the check is positive.

## Mandatory Context Routing

For every Droid production/test edit:

1. read this file;
2. read `.agent/CODE_STYLE.md`;
3. use `.agent/ARCH_INDEX.md` to select one primary architecture owner;
4. load at most two supporting architecture owners when genuinely needed;
5. use `.agent/SOURCE_MAP.md` before broad discovery;
6. inspect only the exact implementation/test/support files required.

Additional routing:

- Android/AndroidX/Material wrapper work -> `.agent/architecture/ANDROID_WRAPPERS.md` + `.agent/ANDROID_CLASS_WRAPPERS.md`.
- Use/change MCP helpers -> additionally `.agent/MCP_WRAPPER_WORKFLOW.md` and `.agent/architecture/BUILD_AND_TOOLING.md`.
- Wrapper completion -> `.agent/DEVCONTAINER_COMPILATION_CHECK.md` is mandatory exact build procedure.
- JNI lifetime/signature/class-loader changes -> `.agent/architecture/JNI_DEPENDENCY_BOUNDARY.md`; inspect JNIKit only when the contract actually requires it.
- Public user-facing API changes -> `.agent/skills/documentation_sync_skill.md` for the docs audit boundary.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swifdroid/droid](https://github.com/swifdroid/droid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
