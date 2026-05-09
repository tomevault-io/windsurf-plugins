---
trigger: always_on
description: Mobile remote editor for iOS and Android. Primary platform is iOS (`gpui_ios` + Metal). Secondary platform is Android (`gpui_android` + `gpui_wgpu` + Vulkan).
---

# Zedra

Mobile remote editor for iOS and Android. Primary platform is iOS (`gpui_ios` + Metal). Secondary platform is Android (`gpui_android` + `gpui_wgpu` + Vulkan).

## Agent Workflow

- Inspect the relevant code paths first and infer local patterns before proposing or making changes.
- Ask before making any meaningful product or architectural decision. Tiny details may follow existing patterns without approval.
- For normal feature work, prefer the smallest diff that fits the current design.
- If the current structure is blocking quality, propose the refactor and wait for approval before doing broader cleanup.
- Keep code concise, readable, and modular. Prefer clarifying code over adding comments.
- When fixing an edge case or an important regression-prone path, add a minimal code comment at the relevant block explaining the invariant or reason for the guard.
- Prioritize correctness and clarity over cleverness or speed unless performance is the explicit problem.
- Avoid panic-prone shortcuts such as unchecked indexing or `unwrap()` in normal code paths. Propagate or handle errors instead of discarding fallible results.
- Surface blockers quickly with a recommendation. Keep progress updates short and include reasoning or tradeoffs.

## Debugging Workflow

- Read the relevant code path deeply before changing behavior.
- On mobile issues, prefer adding targeted `tracing` logs with a clear searchable prefix so the developer can run the app, reproduce, and return logs.
- After the first failed debugging attempt, stop and ask for more information instead of arguing from hypotheses.
- Prefer root-cause fixes once the issue is confirmed.

## Repo Invariants

- `WorkspaceState` is the single source of truth for display state. Views read `WorkspaceState`, never `SessionHandle`, during render.
- `render()` must stay pure. Side effects belong in event handlers, subscriptions, or async tasks.
- Use `platform_bridge::bridge()` for platform integration. Do not call platform APIs directly from UI code.
- Use `tracing` for logging. Never add `log::` calls.
- Read `docs/DESIGN.md` before creating or redesigning UI.
- GPUI tasks are cancelled when their `Task` handle is dropped. Await, detach, or store tasks according to the intended lifetime.
- Inside GPUI entity `update`, `read_with`, and related closures, use the inner `cx` passed to the closure and avoid reentrant updates of the same entity.

## Protocol And Telemetry

- `docs/PROTOCOL_SPECS.md` is canonical. Any protocol change must update `zedra-rpc/src/proto.rs`, the relevant host and client handlers, and `docs/PROTOCOL_SPECS.md` in the same change.
- `crates/zedra-telemetry/src/lib.rs` defines the canonical telemetry `Event` enum.
- Telemetry must not include personal data. Use opaque IDs, durations, counts, enum labels, and booleans only.

## Documentation

- Write docs in a practical, direct style: start with the goal, put common actions first, and avoid promotional or apologetic wording.
- Use complete working examples. Use `sh` fences for terminal command blocks and backticks for inline commands, paths, settings, and keybindings.
- Keep repo guidance high-signal. New rules should be non-obvious, repeatedly encountered, and specific enough to act on; crate-specific rules belong in that crate's `AGENTS.md`.

## Validation

- Prefer targeted checks over broad suites.
- Add or update tests when there is an obvious existing place for them.
- For UI, platform, and device-driven changes, add or update manual verification steps in `docs/MANUAL_TEST.md`.
- Common checks:
  - `cargo fmt`
  - `cargo check -p zedra-rpc -p zedra-session -p zedra-terminal -p zedra-host`
  - `cargo check --manifest-path vendor/zed/Cargo.toml -p gpui_ios -p gpui` for vendored GPUI/iOS framework patches
  - `bun run format`
  - `bun run check`

## Git Commits

- Use commit subjects in the form `feat|fix|chore|docs: <description>`.
- When the change is scoped to a platform, feature, or crate, use `type(scope): <description>`, such as `feat(ios): ...`, `fix(host): ...`, or `chore(rpc): ...`.
- Commits must include only changes related to the current feature or fix. Never stage or commit unrelated work, even when the worktree has multiple concurrent edits.
- Commits made by Codex must include `Co-authored-by: Codex <codex@openai.com>`.
- `vendor/zed` is a separate submodule with its own git style: clear, capitalized, imperative subjects, no conventional prefixes, optional crate scope such as `git_ui: Add history view`, and no trailing punctuation.

## Platform Scope

- iOS is the primary development path. See `docs/IOS_WORKFLOW.md` for build, install, launch, and logging commands.
- Native iOS presentations should keep UIKit responsible for alerts, sheets, and keyboard accessories.
- `UIGlassEffect` is public UIKit on iOS 26+. Use `if #available(iOS 26.0, *)`, not runtime probing.
- In Swift bridge code, keep access control consistent with helper type visibility.

## Vendor Zed

- `vendor/zed` is a git submodule and an intentional part of the architecture, not just third-party reference code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanlethanh/zedra](https://github.com/tanlethanh/zedra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
