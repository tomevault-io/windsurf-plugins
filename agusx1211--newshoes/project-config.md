---
trigger: always_on
description: Project New Shoes runs the original Command & Conquer: Generals / Zero Hour C++
---

# AGENTS.md

## Project state

Project New Shoes runs the original Command & Conquer: Generals / Zero Hour C++
engine in the browser through WebAssembly. The foundational port is substantially
complete: the real engine boots, renders the shell, and runs playable skirmishes.
Current work is product development—features, fidelity, bug fixes, compatibility,
performance, hardening, and cleanup.

Zero Hour in `GeneralsMD/Code/` is the primary target. The main source areas are:

- `GameEngine/` — simulation, AI, data loading, UI logic, objects, weapons, and
  networking protocol;
- `GameEngineDevice/` — rendering, audio, video, input, and OS-facing device
  implementations;
- `Libraries/Source/` — WW3D and the original third-party integration surface;
- `WebAssembly/` — the Emscripten build, browser platform layer, launcher, asset
  import, runtime bridge, and verification harness.

Read `PROJECT.md` for the current architecture before making broad or
cross-cutting changes.

## Private agent instructions

At the start of every task, read the repository-root `AGENTS_PRIVATE.md` after
this file when it exists. It is the place for user- or machine-specific agent
instructions that should remain local. Follow it as supplemental project
guidance, subject to higher-priority instructions and the user's current
request.

`AGENTS_PRIVATE.md` is gitignored. Never stage, commit, quote, or publish its
contents. Its absence is normal and does not block work.

## Agent codename

At the start of every task, after reading the private instructions, use the
repository's `agent-identity` skill at `.claude/skills/agent-identity/SKILL.md`.
Run its generator exactly once for the current parent agent identifier, remember
the resulting codename for the entire task, and do not rerun it unless the
parent identifier changes. The codename distinguishes concurrent agents that
have the same GitHub App and model identity without exposing the parent
identifier itself.

## Engineering stance

The original engine is the product, but it is no longer an untouchable artifact.
Core engine files may be changed when that is the cleanest correct way to add a
feature or fix a problem.

Treat core changes with care:

- understand the real call path, ownership, and invariants before editing it;
- keep changes scoped and reviewable; avoid broad rewrites when a focused change
  will do;
- preserve simulation behavior, data compatibility, save/network determinism,
  and native behavior unless the task intentionally changes them;
- prefer existing engine abstractions and data-driven behavior over parallel
  browser-only implementations;
- use target-specific conditionals only for genuine platform differences, not
  to avoid integrating the real code;
- add or update verification at the level where the behavior is owned.

Platform adapters are still normal and necessary. They must implement the
semantics the engine relies on, with explicit unsupported/error behavior where a
browser cannot provide them.

## No new stubs or fake compatibility

Do not introduce, preserve as the solution, or extend stubs, no-op
implementations, canned-success paths, dummy data, weak fallback ownership, or
“just enough to link” shims unless the user explicitly approves that tradeoff
for the specific task.

In particular:

- do not silently claim success for behavior that did not happen;
- do not shadow a real engine implementation with a simplified copy;
- do not make a harness-only implementation stand in for product behavior;
- do not hide unsupported behavior behind empty methods or invented defaults;
- when existing legacy stubs are encountered, avoid expanding their role and
  prefer retiring them through the real implementation.

An existing legacy stub may remain when it is outside the requested scope, but
it cannot be the implementation or completion evidence for new work.

Test doubles that are scoped to tests and clearly identified as such are fine.
Temporary diagnostic hooks are fine when they observe or drive the real runtime
without replacing product behavior.

## Implementation workflow

- Start from the requested feature, bug, or measured product problem. Reproduce
  it when practical and trace the real runtime path before changing code.
- Build fixes and features into the actual `cnc-port` runtime. Focused tests are
  useful, but they do not replace integration through the shipping path.
- Prefer `npm run build:port` for the normal iteration loop. Use broader builds
  and regression suites in proportion to the change.
- Preserve unrelated user changes and keep commits narrowly scoped.
- When a task exposes separate follow-up work, report it clearly instead of
  quietly widening the current change.

## Public project information

`WebAssembly/pages/project-content.json` is the canonical source for public and
agent-facing claims on newshoes.gg. Update it whenever a change affects a public
capability, status, requirement, setup step, privacy behavior, limitation,
troubleshooting answer, or stable resource. Every capability needs a current
review date and repository evidence paths.

Do not hand-edit `llms.txt`, `project.md`, `project-info.json`, `robots.txt`, the
sitemap, or the discovery metadata in built HTML. The Pages build generates all

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agusx1211/NewShoes](https://github.com/Agusx1211/NewShoes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
