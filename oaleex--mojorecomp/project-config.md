---
trigger: always_on
description: This file defines the operating rules for AI coding agents working in this
---

# MojoRecomp Agent Guide

This file defines the operating rules for AI coding agents working in this
repository. It is intentionally focused on decisions and guardrails. User-facing
setup and product information belongs in `README.md`.

## Project identity and scope

- The project name is **MojoRecomp**. The desktop application is
  **MojoRecomp Launcher**.
- Do not introduce or restore the former project name in code, paths, UI,
  metadata, package names, documentation, or generated output.
- MojoRecomp is an independent native recompilation project for Xbox 360 Crash
  titles. It is not affiliated with or endorsed by the games' rights holders.
- **Crash of the Titans (COT)** is the current playable target.
- **Crash: Mind over Mutant (MOM)** has a separate launcher profile and runtime
  identity, but is not yet a playable target.
- Keep maintained source code, project-controlled identifiers, comments, UI
  text, configuration, tests, logs, and documentation in English.
- English is mandatory for class, function, method, variable, constant, field,
  enum, test, file, and directory names created by the project; comments;
  diagnostics; console/log output; test fixtures and synthetic path names; build
  scripts; and developer-facing messages. Do not introduce Portuguese or any
  other non-English prose into maintained code merely because the project owner
  speaks that language. Non-English text is allowed only when it is intentional
  localization content or a proper name/title that must retain its original
  spelling.
- Do not use emoji or decorative Unicode glyphs in maintained source code,
  identifiers, comments, tests, logs, diagnostics, scripts, configuration, or
  developer-facing runtime/build output. Prefer plain ASCII punctuation there
  unless a non-ASCII character is required by an intentional localization or
  character-encoding test. Markdown documentation may use emoji or Unicode
  symbols when they improve readability, such as status indicators or checkmarks.

## Non-negotiable Git and collaboration rules

Read-only Git commands such as `git status`, `git diff`, `git log`, and
`git ls-files` are allowed. Every state-changing or remote Git action requires
an explicit instruction from the project owner for that exact action.

- Never run `git add`, `git commit`, `git push`, `git tag`, or publish a release
  unless explicitly requested.
- Never create an issue or pull request. If the owner later wants one, prepare a
  local summary or patch for human review instead of publishing it. Do not
  contact upstream projects or their maintainers on your own.
- Never force-push or change Git remotes.
- Do not create or switch branches/worktrees, stash changes, merge, rebase,
  amend, reset, restore, checkout files, or clean the worktree unless the owner
  explicitly requests that operation.
- A request to "finish", "fix", "update", or "clean up" is not authorization
  to stage, commit, push, open a PR, or modify the index.
- Preserve unrelated user changes. Inspect the relevant diff before editing and
  never discard work merely to obtain a clean status.
- The historical Git index is protected and may retain references to removed
  private or diagnostic material. Do not publish, commit, rewrite, unstage, or
  clean that state without a specific owner-approved plan.
- If a future request explicitly authorizes a commit, first inspect the exact
  staged set and ensure the protected historical index and private material are
  excluded. Do not assume the existing index is safe to commit.

## Legal and data boundary

The repository must remain useful without distributing copyrighted game data.
Users provide their own legally obtained supported game input.

Never add, stage, package, upload, quote, or expose:

- Xbox 360 disc images, archives, executables, title updates, keys, decrypted
  intermediates, movies, audio, textures, or other original game assets;
- extracted or managed game directories, including `games/`;
- saves, settings, user content, caches, or any `userdata/` contents;
- generated PPC translation units or generated guest images;
- build directories, packaged applications, executables, DLLs, symbols, maps,
  crash dumps, shader dumps, logs, screenshots, captures, or support bundles;
- local toolchain/dependency trees; or
- `.private/` contents, agent handoffs, operational notes, or other private
  development state.

Do not use real user saves or installed game data in automated tests. Use
isolated temporary fixtures and synthetic inputs. Do not weaken ignore rules or
release validation to make a local build artifact appear publishable.

Third-party license and notice files are required compliance material, not
documentation clutter. Preserve them and their provenance. Original MojoRecomp
code is licensed under the ISC License in the repository root unless an
individual file states different terms. Do not apply that ISC grant to
third-party code/tools, generated guest/PPC or original game code, game data,
launcher game artwork, names, characters, logos, trademarks, or any material
owned by another rights holder.

## Repository map and ownership

- `launcher/`: MojoRecomp Launcher, built with Tauri, Rust, Svelte, TypeScript,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OAleex/MojoRecomp](https://github.com/OAleex/MojoRecomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
