---
trigger: always_on
description: - Must complete tasks end-to-end without asking the user for confirmation on small or medium-sized implementation decisions.
---

## Execution Autonomy

- Must complete tasks end-to-end without asking the user for confirmation on small or medium-sized implementation decisions.
- Must make local naming, refactoring, implementation, and routine workflow decisions independently when they are reversible and do not change security, architecture, or product scope.
- Must create atomic commits during multi-step work as each independent step is completed and verified. Must not bundle unrelated steps into one final commit.
- Must apply existing knowledge, tacit engineering knowledge, and established best practices directly. Must not ask about standard implementation details that a competent engineer should already know.
- Must not interrupt execution for trivial questions, obvious choices, or routine best-practice decisions. Ask only when the choice is irreversible or materially affects security, architecture, or product behavior.

## Product Stage and Development Bias

- This project is in a high-velocity product iteration stage. Optimize for quickly shipping coherent user-facing behavior, not for preserving existing implementation details by default.
- Product intent from the user is the primary source of truth. Existing behavior is evidence, not authority; if the current behavior conflicts with the intended user experience, change the behavior directly.
- Default to developing directly on the current `main` branch in this repository. Do not create a new branch, worktree, or detached work area unless the user explicitly asks for one, the user asks for parallel isolated work, or a concrete safety reason requires isolation.
- Work in small, atomic steps on `main`: each independent behavior change should be implemented, verified, and committed separately. Do not accumulate unrelated changes for one large final commit.
- Prefer decisive product fixes over narrow patches that only silence the immediate symptom. If the root problem is a mismatched product model, fix the model rather than adding local guardrails around the symptom.
- Avoid unrelated changes, not necessary product changes. If the intended behavior requires changing a broader product model, do that directly and keep unrelated refactors out of the commit.
- Keep engineering discipline proportional to risk: inspect the relevant code first, preserve data safety, avoid avoidable regressions, and verify the actual running product path before claiming completion.
- When validation matters, verify against the real app or runtime the user is using, not only an isolated worktree, stale build, or inferred code path.

## Core and Shell Architecture

- The **Go core** (`internal/`, `cmd/wuu/`) is the reusable foundation: agent runtime, providers, tool loop, sessions, config, and the `wuu app-server` subprocess. The core is not coupled to any specific shell.
- The **current shell** is the Electron desktop in `desktop/`. It spawns the core as a subprocess and owns the UI, native integrations, and the IPC bridge.
- **Future shells** (VS Code extension, JetBrains plugin, etc.) consume the core by spawning `wuu app-server`. They do not need to import or fork the Go core; they reuse it as a process.
- A change is **shell-level** when it touches `desktop/` only (UI, native APIs, packaging). A change is **core-level** when it touches `internal/` or `cmd/wuu/`. Keep this boundary clean: do not let the shell leak into the core, and do not let the core depend on Electron APIs.

## Agent-Friendly Text Entrypoint

- Wuu has no TUI. Use Electron for human interaction and `wuu exec` for agents, scripts, CI, and automation.
- When an agent modifies Wuu's agent-facing runtime, verify the product path with `wuu exec --json` or the `wuu debug app-server ...` protocol probes when practical.
- Preserve automation-safe output: default `wuu exec` stdout is only the final answer, JSON mode stdout is only JSONL, and diagnostics belong on stderr.

## Intent First

- Must start from the user's real goal before optimizing local implementation details. The current codebase is context, not the primary definition of what should be built.
- When a request affects product behavior, must reason first about interaction design, visual design, and the broader project vision before choosing detailed code changes.
- Must not assume the existing implementation is correct just because it already exists. Evaluate whether it actually serves the intended user experience and product direction.
- Must not get trapped in minor technical details when they do not materially affect the user's outcome. Prioritize the highest-leverage product and UX decisions first.
- Must still inspect and understand the relevant code before changing it, but that inspection must support the intended outcome rather than let the current implementation define the goal.

## Third-Party Reference Code

- The `thirdparty/` directory contains reference implementations from related agent, CLI, and product codebases. Treat it as a local research library when the user asks for "industry best practices", "how others do this", "reference implementations", or similar guidance.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueberrycongee/wuu](https://github.com/blueberrycongee/wuu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
