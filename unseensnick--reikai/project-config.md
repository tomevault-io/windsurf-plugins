---
trigger: always_on
description: Android manga + light-novel reader. Personal fork built on [Mihon](https://github.com/mihonapp/mihon) (Tachiyomi lineage), adding light novels, multi-source grouping, manual merge/unmerge, and category sort order.
---

# Reikai

Android manga + light-novel reader. Personal fork built on [Mihon](https://github.com/mihonapp/mihon) (Tachiyomi lineage), adding light novels, multi-source grouping, manual merge/unmerge, and category sort order.

**Rebase onto Mihon (shipped 2026-06 as v0.1.0):** Reikai was previously a fork of [Yōkai](https://github.com/null2264/yokai); it has been rebased onto Mihon. The rebase has shipped: `main` is now the Mihon-based main (the old `design/mihon-rebase` branch is gone). The old Yōkai-based code (branch `design/library-compose`) is kept only as the porting reference. Full plan and feature list: [ROADMAP.md](ROADMAP.md) plus the per-feature records in [docs/dev/plans/](docs/dev/plans/); ongoing status: the `mihon-rebase` memory.

## Working approach

**Investigate before planning when context is thin.** If you aren't confident you understand the surrounding code, conventions, or constraints for a task (porting a Reikai feature onto Mihon, touching an unfamiliar module, changing cross-cutting infrastructure), investigate first: read the relevant files, trace the existing pattern. Two reference sources: `refs/mihon/` (the live upstream base) and the `design/library-compose` branch (Reikai's Yōkai-era features awaiting port). For non-trivial work, invoke `/scout` to produce a grounded findings report before forming a plan. Only present a plan once you're truly confident, then wait for approval before executing.

**Cite before you claim.** Every concrete claim about the codebase, framework, or upstream (a function name, a file path, a flag, "X calls Y") must come with a `file:line` citation from current code that you just read. If you can't cite it, you don't know it: read first, claim second. Memory and Handoff content are hypotheses, not facts; a memory that names a function or file is true only if it still exists in current code. When a stale memory is found, surface it for pruning instead of acting on it.

**Plan before acting.** Once you have enough context, think through what needs to change and why: which files are affected, what the failure modes are, whether the approach is sound. Use `EnterPlanMode` for non-trivial tasks to draft and get approval before touching code.

**Stop and replan when blocked.** If you hit an unexpected problem mid-task (a failing constraint, a broken assumption, an error you don't fully understand), stop all changes immediately and surface the blocker. Do not circumvent it (deleting a test, silencing a lint error, skipping a hook, or forcing past a tool denial). Replan from scratch with the new information.

**Offload long or hard tasks to subagents.** When a task requires deep codebase exploration, multi-file research, or extended multi-step work, spawn a subagent (`Agent` tool). This keeps the main context window clean.

**Explain in plain English, without dumbing down.** Default to clear everyday language: spell out what something does and why it matters before naming the construct, define jargon the first time, and prefer a concrete analogy over a term of art (the user is newer to Kotlin/Android). Plain English does NOT mean less substance: keep the real technical detail, the tradeoffs, the failure modes, and the `file:line` citations. The goal is that someone can follow the reasoning without already knowing the codebase, not that the content is thinner. When presenting findings or a plan, lead with the plain-English picture; the precise function/file names are support, not the headline.

## Architecture in brief

Mihon is **Compose + Voyager throughout**: there is no Conductor `*Controller` / RxJava `*Presenter` legacy layer to migrate from. Screens are Voyager `Screen` / `Tab` classes backed by a `ScreenModel`. DI is **Injekt**. Domain models are immutable (`tachiyomi.domain.*.model`). Preferences go through `PreferenceStore` and typed `*Preferences` classes. Persistence is SQLDelight. Full detail: [.claude/rules/architecture.md](.claude/rules/architecture.md).

## Screen conventions (match Mihon)

Every Reikai screen ported onto or added to Mihon follows Mihon's existing Voyager conventions. Index (full rationale + reference screen in [.claude/rules/screen-conventions.md](.claude/rules/screen-conventions.md)):

1. A Voyager `Screen` / `Tab` class, not a bare `@Composable fun FooScreen()`.
2. Business logic in a `ScreenModel` resolved via `rememberScreenModel { ... }`. Pure-UI screens may skip it and say so in a one-line comment.
3. No `Injekt.get<>()` / `injectLazy()` inside a `@Composable` body. Inject in the ScreenModel.
4. State exposed as `StateFlow` (typically `StateScreenModel<S>`). No RxJava on the screen path.
5. No `PreferenceStore` / `*Preferences` read inside a `@Composable`. Read in the ScreenModel, expose as state.
6. Coroutines via `screenModelScope.launchIO` / `launchUI` (ScreenModel) or `rememberCoroutineScope()` (composable). Never `GlobalScope`; use `WorkManager` for work that must outlive the screen.
7. Business logic out of `@Composable`. Side-effects in `LaunchedEffect` or the ScreenModel.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unseensnick/Reikai](https://github.com/unseensnick/Reikai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
