---
trigger: always_on
description: We want to build a first-class open-source AI coding assistant: a terminal-native agent that edits code, runs commands, and helps carry multi-step development workflows. This repository is the standalone public source export of `code/` from Noumena's internal monorepo.
---

# Noumena Code: What We're Building

We want to build a first-class open-source AI coding assistant: a terminal-native agent that edits code, runs commands, and helps carry multi-step development workflows. This repository is the standalone public source export of `code/` from Noumena's internal monorepo.

Your goal is to keep the public CLI correct, buildable, and safe for external contributors and users who clone it and immediately ask their agents to work on it.

This means: no hidden dependencies, no "agents will figure it out" assumptions, no silent fallbacks, and no changes that break `git clone && bun install && bun run build`.

**Elegant minimalism** isn't just fewer lines; it's disciplined intent plus impeccable execution.

## Repository Contract

- This repo is **public / OSS**. Do not commit secrets, internal hosts, hardcoded IPs, or unreleased model identifiers.
- Do not add internal-only features without a clear public bypass or build-time gate.
- The default build path must remain `bun install && bun run build`. Any new required step must be explicit in `README.md`.
- Keep surfaces small: one supported path per use-case unless we are intentionally replacing the old one.

## Execution Environment

- This repo uses **Git**, not Sapling. Use `git` for history, branches, and pull requests.
- The supported build tooling is **Bun + Rust/Cargo**. Do not assume Python, Docker, Kubernetes, or Buck2 are available.
- Local builds produce a single-file native `ncode` binary under `.tmp/packages/`.
- Tests are run with `bun test <file>` or `bun run test` (isolated per-file).

## Agent Protocol (Failure Prevention)

A lot of people clone this repo and immediately have their agent install things. These rules prevent that from breaking the repo.

### Session Start (Always)

- Read `AGENTS.md`, then `CLAUDE.md` (if present).
- Confirm repo root with `git rev-parse --show-toplevel`.
- Confirm working copy state with `git status --short`.

### Mode Gates (Hard)

**No-Edits Mode**
- Trigger: user says "do not make edits/changes", "review only", or "planning/brainstorming".
- In this mode: do not modify tracked files, do not install deps, do not run destructive Git ops, and do not change state; only read/inspect/analyze.
- Exit only when the user explicitly authorizes execution ("proceed", "implement", "make the changes", "do it").

**Execution Mode**
- Default when the user asks to implement/fix/build.
- If the user says "just X", do X immediately with minimal narration.

### Scope Lock (Before Editing)

- Before editing: list the exact files you will modify.
- Do not touch out-of-scope files; stable/working code is read-only unless explicitly told otherwise.
- For ports/refactors: preserve semantics by default; call out intentional semantic deltas and get approval.
- After fixing a bug pattern: search for other occurrences (prefer `rg`) and fix them in-scope.

### Don't Guess (Ever)

- Never guess at environment state, config values, or file contents. Verify via files, diffs, logs, or commands.
- Don't assume hidden tools exist. The supported toolchain is Bun, Cargo, and `git`.

### Git Safety (High Severity)

- This is a Git repo. Do not run Sapling commands (`sl`, `sl status`, `sl diff`, etc.).
- Do not run `git reset --hard`, `git checkout -- .`, `git rebase -i`, or destructive history edits without explicit approval.
- Never `git push --force` to `main`.
- Create pull requests for non-trivial changes; avoid pushing directly to `main` unless explicitly authorized.

### Dependency / Install Discipline (Critical for OSS)

- **Do not install system packages, global npm/bun/cargo packages, or OS tooling as a fix.** If the repo genuinely needs a new dependency, add it to `package.json` or `Cargo.toml` and document why.
- **Do not modify the user's shell config** (`.bashrc`, `.zshrc`, etc.) to make the build work.
- **Do not rely on `sudo` or root privileges.** The build must work in a normal user clone.
- If a build fails because a tool is missing, check `README.md` and `OSS_BUILD.md` first. If it's not documented, treat it as a missing repo setup step and ask before adding it.
- If `bun install` does not provide a needed Node dependency, add it as a normal dependency/devDependency in `package.json`, not as a side-effect command.

### Build Discipline (High Severity)

- The canonical build is `bun run build` (alias for `bun run build:external`).
- Do not delete `.tmp/`, `dist/`, or build outputs "just to be safe".
- Do not introduce a second durable build path for the same supported workflow.
- Native Rust N-API modules live under `native/` and are built by the Bun build scripts. Do not manually invoke `cargo build` at the root as the supported path.

### Validation Discipline

- Start with the smallest relevant validation:
  - `bun test <changed-file>`
  - `bun test src/path/to/file.test.ts`
- Run the full suite only when the task touches broad cross-cutting concerns.
- For build/package changes, run `bun run build` end-to-end and verify the binary path printed in the manifest.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noumena-Network/code](https://github.com/Noumena-Network/code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
