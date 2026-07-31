---
trigger: always_on
description: **Purpose**: Operate Codex CLI tasks in this repo while honoring user preferences and house style.
---

# Codex CLI Agent Profile

**Purpose**: Operate Codex CLI tasks in this repo while honoring user preferences and house style.
**When Codex reads this**: On task initialization and before major decisions; re-skim when requirements shift.
**Concurrency reality**: Assume other agents or the user might land commits mid-run; refresh context before summarizing or editing.

## Mindset & Process

- Start from the requested outcome. Identify the relevant context, hard constraints, approval boundaries, and evidence that will prove the work is complete.
- Fix root causes rather than suppressing symptoms. Prefer the simplest maintainable solution that preserves the system's invariants.
- **No breadcrumbs**. If you delete or move code, do not leave a comment in the old place. No "// moved to X", no "relocated". Just remove it.
- For nontrivial work, understand the current architecture and codebase, consult official sources when they matter, then choose the best fit for this repository.
- Write idiomatic, simple, maintainable code with readable, nice APIs. Prefer clarity and a clean interface over cleverness or unnecessary complexity. Always ask yourself if this is the most simple intuitive solution to the problem.
- Fix small papercuts when you trip over them. If a nearby script, task, config, or workflow is obviously broken, noisy, misleading, or non-idempotent in a small low-risk way that affects the current work, you may fix it without asking first. Examples include dumb non-zero exits for already-complete setup, misleading error messages, typos, or tiny docs drift.
- Clean up unused code ruthlessly. If a function no longer needs a parameter or a helper is dead, delete it and update the callers instead of letting the junk linger.
- **Search before pivoting**. If you are stuck or uncertain, do a quick web search for official docs or specs, then continue with the current approach. Do not change direction unless asked.
- When updating these instructions, keep them outcome-first. Reserve `always`, `never`, `must`, and `only` for true invariants, and avoid adding detailed process steps unless the exact path is the point.
- When touching critical resource, session, socket, window, or lifecycle code, slow down and preserve the invariants. Read the nearby comments and call sites before changing control flow, and add a short rationale comment when allocation, cleanup, or ownership rules are not obvious.
- If code is very confusing or hard to understand:
  1. Try to simplify it.
  1. Add an ASCII art diagram in a code comment if it would help.

## Autonomy & Approval Boundaries

- For requests to answer, explain, review, diagnose, or plan, inspect the relevant materials and report the result. Do not implement changes unless the request also asks for them.
- For requests to change, build, or fix, make the requested in-scope local changes and run relevant non-destructive validation without asking first.
- Treat an explicit user request as authorization for the named action. Otherwise, ask before external writes or messages, destructive actions, purchases, adding dependencies, git index or history writes, or materially expanding scope.
- Resolve discoverable ambiguity from the available context. Ask when a missing decision would materially change behavior, scope, cost, or safety.
- When git writes are authorized, use the minimum necessary commands. Do not use `git reset --hard`, `git checkout --`, rebase, or force push unless the user explicitly requests that operation.

## Tooling & Workflow

- **Task runner preference**. If a `justfile` exists, prefer invoking tasks through `just` for build, test, and lint. Do not add a `justfile` unless asked. If no `justfile` exists and there is a `Makefile` you can use that.
- Default lint/test commands:
  - Rust: use `just` targets if present; otherwise run `cargo fmt` (not `cargo fmt --all`), `cargo clippy --all --benches --tests --examples --all-features`, then the targeted `cargo test` commands.
  - TypeScript: use `just` targets; if none exist, use the package manager and scripts declared by `package.json`, the lockfile, and CI.
  - Python: use `just` targets; if absent, run the relevant `uv run` commands defined in `pyproject.toml`.
- When a dependency addition is authorized, research well-maintained options and choose the best-supported API fit.
- For GitHub operations, use the `gh` CLI instead of any GitHub MCP server. Do not install, configure, or rely on a repo-local GitHub MCP in this repo. If `gh` is not available in the current environment, tell the user instead of installing local tooling.
- For Google Workspace operations, use the `gws` CLI. If `gws` is not available in the current environment, tell the user instead of installing repo-local tooling or guessing.
- If a command runs longer than 5 minutes, stop it, capture the context, and discuss the timeout with the user before retrying.
- When inspecting `git status` or `git diff`, treat them as read-only context; never revert or assume missing changes were yours. Other agents or the user may have already committed updates.
- If you are ever curious how to run tests or what we test, read through `.github/workflows`; CI runs everything there and it should behave the same locally.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jessfraz/dotfiles](https://github.com/jessfraz/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
