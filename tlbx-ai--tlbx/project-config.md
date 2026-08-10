---
trigger: always_on
description: This terminal runs inside MidTerm (web terminal multiplexer).
---

This terminal runs inside MidTerm (web terminal multiplexer).

If `.tlbx/AGENTS.md` exists, follow it for browser control and tmux workflows.
If it does not exist, do not assume extra MidTerm-specific workflow permissions.

## Release Authority

Release requests authorize the complete matching script workflow in the current turn. Do not run unrelated release, tag, publish, promote, or merge-to-main workflows outside the requested path.

Development happens on `dev`. `main` is only for stable integration/promotion. If a task requires switching branches, do it automatically, complete the requested work, and return to `dev` after main/stable integration is finished.

- Midterm is programmed in c# and typescript -> all major data processing/protocol logic/business logic shall be handled in c#, the typescript frontend shall be held as lean as possible.
- Use best practices for maintainable memory efficient code that uses the newest available .net features >= .net 10
- We do not have a big team continuously revisiting code quality, and we cannot afford to come back later to clean up avoidable leftovers. If a feature change or refactor supersedes logic, helpers, types, config, branches, or APIs, remove that dead code in the same change. Do not leave cleanup debt behind on the assumption that someone will revisit it later.
- always search if somthing exist first before implementing new features/api surfaces 
- Midterm is in production, it is used by large teams and needs to be stable and performant 
- For keyed UI lists/trees, use `src/Ai.Tlbx.MidTerm/src/ts/utils/domReconcile.ts`; content-only hot updates must preserve DOM node identity and full rebuilds are only for structural add/remove/move/filter changes.

Rules:

- "Cut a dev/prerelease" authorizes the full `scripts/release-dev.ps1` cycle: version bump, verification, commit, annotated prerelease tag, push to `dev`, push tag, and the CI/artifact publishing triggered by that push.
- If the user says "patch release", "minor release", or "major release" without specifying stable/main/promote, default to the full dev/prerelease path with `scripts/release-dev.ps1`.
- "Stable release", "main release", and "promote" all mean the stable promotion path from `dev` to `main`. Use `scripts/promote.ps1`, not `scripts/release.ps1`, unless the user explicitly asks for a direct main-branch release script.
- "Promote current dev to stable" authorizes the complete `scripts/promote.ps1` workflow: create/find the PR, merge `dev` to `main`, update the stable version, commit, tag, push `main`, push tag, merge `main` back into `dev`, and push `dev`.
- `scripts/release.ps1` is the direct main-branch stable release script. Treat it as exceptional and only use it when the user explicitly asks for a direct stable release from `main`.
- Dev/prerelease path: use `scripts/release-dev.ps1`.
- Stable promotion path: use `scripts/promote.ps1`.
- Direct stable release path: use `scripts/release.ps1` only when explicitly requested.
- Never promote to stable, merge to `main`, or run `scripts/promote.ps1` unless the user explicitly asks for stable/main/promote. A dev/prerelease request never implies stable promotion.
- Release scripts are allowed to create and push release tags and release commits as part of their authorized full cycle.
- For MidTerm fixes, implement and verify the change, then cut a dev patch release with `scripts/release-dev.ps1` unless no good solution was found, verification is uncertain, or Johannes explicitly says not to release yet.
- Before running a release script, inspect `git status`. Fix normal worktree issues yourself, including committing intended changes or removing accidental generated leftovers. If the tree contains outlandish or unrelated changes whose ownership or intent is unclear, stop and ask before releasing.
- Choose `-mthostUpdate yes` when the protocol between `mt`, `mthost`, or `mtagenthost` changes, or when `mthost`/`mtagenthost` internals changed in a way that must ship to running installs. Choose `-mthostUpdate no` for web/frontend-only changes. If uncertain, ask before releasing.
- For release tasks, the final stop message must end by showing the release contents. Prefer dumping the same changelog entries that were passed to the release script.

## Terminal Design Constraints

- Do not suggest hiding, virtualizing, or lazily deactivating visible terminal sessions as a latency optimization.
- In MidTerm, sessions that are shown are intentionally kept as genuinely active terminals; latency work must preserve that UX model.

## Terminal Size Ownership

- Terminal row/column size ownership is server-authoritative and scoped per terminal session, never global to the whole browser.
- Only the current owner may send authoritative `cols`/`rows`; every resize must carry the server-issued ownership epoch. Followers render the canonical PTY size and CSS-scale locally.
- A user can explicitly take control at any time. The takeover must apply immediately and clearly explain that the terminal will be optimized for this browser.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tlbx-ai/tlbx](https://github.com/tlbx-ai/tlbx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
