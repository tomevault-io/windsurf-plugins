---
trigger: always_on
description: Repo-specific instructions for Codex working on `truST`.
---

# AGENTS

Repo-specific instructions for Codex working on `truST`.

This is the canonical shared rulebook for repo agents. Claude imports this file
from `CLAUDE.md`; Codex reads it directly. Keep shared rules here or in
repo-local `.codex/skills/**`, not in tool-private memory.

## Branch and Worktree Bootstrap Rule (non-negotiable)

- As part of creating or starting work on any branch, worktree, clone,
  validation copy, temporary checkout, or remote-builder checkout, identify the
  primary checkout that holds the canonical agent files.
- Manually copy the root `AGENTS.md` and the complete `.codex/skills/**`
  directory from that primary checkout into the destination checkout. Never
  assume ignored files were transferred by Git, branch creation, worktree
  creation, cloning, rsync, or another synchronization step.
- Before editing, building, testing, committing, pushing, merging, or releasing,
  verify that the destination copies exist and match the canonical source, then
  read the destination `AGENTS.md` and every skill required for the task.
- Report the canonical source path, destination path, branch, HEAD, and copy
  verification result. If the source files are missing, stale, or cannot be
  verified, stop and restore them before continuing.
- Every sub-agent must independently verify its own checkout and must not edit
  until this bootstrap has passed.
- When switching branches inside an existing checkout, verify the files again.
  When creating a branch in a new worktree or checkout, always copy them
  manually.

## Test-First Development Rule (non-negotiable)

- Apply test-first development to every behavior-changing code task: new
  features, bug fixes, and intentional behavior changes. Do not write or change
  production code for a behavior before establishing the test for that
  behavior.
- Work one observable behavior slice at a time:
  1. Write the smallest focused automated test that expresses the requested
     behavior, and register it in the real test runner when registration is
     required.
  2. Run that test before implementation and confirm it fails for the expected
     reason: the behavior is missing or incorrect.
  3. Implement only the minimum production change needed for that behavior.
  4. Run the same test again. If it is red, fix the implementation and repeat;
     only a green result completes that behavior slice.
- A compile error, missing dependency, unregistered test, broken harness,
  timeout, or unrelated failure does not count as the required red result. Fix
  the test environment and rerun until the test reaches the expected behavior
  assertion.
- Do not add post-hoc tests to justify code that was already implemented. If
  production code exists before the required red result, establish an honest
  pre-fix baseline in an isolated checkout or revert only the unapproved change
  with the user's permission, then perform the red-green loop.
- For behavior-preserving refactors, first add or identify behavior-lock tests
  and confirm they are green before editing; keep them green throughout. If the
  refactor introduces any new behavior, use the red-green workflow for that
  behavior.
- Frontend and VS Code work follows the same rule. Use a rendered interaction,
  state, or layout test at the closest practical boundary; static source-text
  checks alone do not prove user-visible behavior. After the focused test is
  green, complete the required real rendered/browser verification separately.
- Record both commands and outcomes in the handoff: the expected red failure
  before implementation and the green result from the same focused test after
  implementation. Broad validation and release gates run after the focused
  red-green loop; they do not replace it.


## Codebase Orientation

- `crates/trust-syntax`: lexer/parser, rowan CST.
- `crates/trust-hir`: semantic model, type checking, and IEC rules.
- `crates/trust-ide`: diagnostics, completion, hover, references, and editor
  feature logic.
- `crates/trust-lsp`: LSP protocol boundary and command wiring.
- `crates/trust-dev`: developer/workbench CLI package.
- `crates/trust-plcopen`: PLCopen XML import/export helpers.
- `crates/trust-runtime-core`: portable runtime core/value/bytecode pieces.
- `crates/trust-runtime`: Linux host runtime, product CLI, IO/web/control
  surfaces.
- `editors/vscode`: VS Code extension.

Common conventions:

- Use `smol_str::SmolStr` for interned strings when the surrounding crate does.
- Use `rustc_hash::FxHashMap/FxHashSet` for hot internal maps/sets when already
  established locally.
- Use `thiserror` for library errors and `anyhow` for application/CLI errors.
- `unsafe_code = "forbid"` is expected for language/IDE/dev-tooling crates such
  as `trust-syntax`, `trust-hir`, `trust-ide`, `trust-lsp`, and `trust-dev`.
  Runtime/host and vendored low-level code may contain registered unsafe sites;
  those are governed by the unsafe register and board evidence, not by a
  workspace-wide unsafe ban.

When adding or changing tokens in `trust-syntax`, update the `TokenKind` enum,
the matching `SyntaxKind` conversion/table, docs/spec token coverage when
applicable, and lexer/parser tests or snapshots.

## Release Hygiene Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johannesPettersson80/trust-platform](https://github.com/johannesPettersson80/trust-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
