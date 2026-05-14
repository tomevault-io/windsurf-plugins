---
trigger: always_on
description: - Be brief. Don't throw walls of text. Break things down in steps.
---

## Interaction

 - Be brief. Don't throw walls of text. Break things down in steps.
 - Ask one question at a time. Don't bundle multiple questions.
 - Don't acknowledge corrections ("You're right", "Good point"). Just act on them.
 - Use Blade Runner names in examples, not the user's real name.

## Agent notes

 - Use quiet modes to avoid polluting your context window. E.g. `--quiet` on build/test commands.
 - By your nature you are overconfident in your knowledge. Don't trust, verify. Read man pages, check tool behavior where possible. You have a pattern of acknowledging rules in reflection but not following them during work -- knowing a rule exists is not the same as applying it. Slow down at decision points.
 - However, tool calls are not a substitute for thinking. Form a hypothesis before verifying.
 - You are running inside a VM which is different from the production environment.
 - Avoid cryptic Bash tool calls, they are hard to review for permission checks.
 - Always use `rcl` for json processing, it's better suited for this than `jq`.
 - Don't fabricate capabilities. Verify tool availability before claiming you can use something.
 - When the compiler warns, it's right. Investigate instead of suppressing.
 - Verify subagent work: check that agents committed on the right branch with actual changes.
 - Never use `find` or `grep` to navigate the repo, they waste context on untracked files. Use `git grep` and `git ls-files` instead.
 - When memory and CLAUDE.md disagree, treat it as a signal to verify before acting, not to pick a side on autopilot. Memory is observation and may drift; CLAUDE.md is also written and can be stale or wrong. For destructive or externally visible actions especially, ask rather than guess which one is right.

## Project details

 - This tool is for expert users (me) who can debug and understand the source. I will be watching the tool while it runs.
 - That means that a crash is _relatively_ not as bad as in a long-running daemon.
 - Therefore, optimize for code simplicity and readability over fancy tool output.
 - It needs to work for me on my laptop and cluster. This is not a generic tool that needs to work for every possible user on the planet.
 - We build incrementally with a short feedback loop. Make it work first, we'll make it fancy later. Do not prematurely complicate or generalize, make it easy to change later.

## Working through a task

 - Keep testability in mind from the start. Functional approaches (pure core, IO at the edge) are often more feasible to test than imperative code.
 - Tests must not call `sleep`, `Instant::now`, `SystemTime::now`, or perform real I/O -- even with zero durations, timing calls couple test speed and stability to the scheduler, and "fast" tests on a developer laptop become flaky on a slow CI box. When code measures time or does IO, push those calls up to the caller so tests can pass deterministic substitutes (or test the pure data-in/data-out layer separately).
 - Do not pull in external dependencies without permission. Permission will only be granted if there is a good justification.
 - The docs are not law. If we discover design flaws while implementing, we can stop and change the design.
 - For large tasks, run `git diff` at the end and review your own work. It is very unlikely that you got a perfect version on your very first iteration, usually there are substantial things to change.
 - Before presenting code to the user, run the post-generation checklist below and fix what you find. Do not skip this step. Do not say "the code looks good" without having run `git diff` and walked every checklist item. Walking the checklist means articulating *why* each item passes for this change. If your check is "looks fine", that's rubber-stamping; you haven't actually checked. The checklist exists because these items are easy to miss when reviewing your own work — running it as ritual defeats the purpose.
 - I will review your changes afterwards, so optimize for small reviewable diffs. Do not change comments or code without a good reason.
 - If it gets complex, typecheck at intermediate points with `cargo check --quiet`.
 - If the changes touch a test or code covered by tests, confirm with `cargo test --quiet`.
 - If the changes are not covered by a test, ask yourself, should they be? Not everything makes sense to test.
 - Run `cargo fmt --quiet` at the end on Rust code, `black --quiet` on Python code.
 - Write failing tests first, then implement (red-green).
 - New code paths need new tests before claiming done.
 - After a task is complete (I will tell you when it is, after you address my comments), reflect on the conversation. Are there any generic learnings? Update CLAUDE.md or your memory.

## Reviewing your own work

Review at multiple levels, from high to low:

 1. Does the new behavior actually solve the problem we set out to solve?
 2. Does the diff implement the proposed solution? Are there logic bugs?
 3. Do the structs, methods, functions make sense? Could the call graph be simpler? Is there duplication?
 4. Local code quality: complex chains that could be a match? Comments stating the obvious? Missing justifications?

At every level, ask: is this complexity inherent, or an artifact of how I implemented it?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruuda/deptool](https://github.com/ruuda/deptool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
