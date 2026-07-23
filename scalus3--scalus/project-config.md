---
trigger: always_on
description: Scalus is a platform for developing decentralized applications (DApps) on Cardano.
---

# Claude Code Guidelines for Scalus Project

## Project Overview

Scalus is a platform for developing decentralized applications (DApps) on Cardano.
It compiles a subset of Scala 3 to Scalus Intermediate Representation (SIR) and then
lowers to Untyped Plutus Core (UPLC), the language of Cardano smart contracts.

**For smart contract development:** Use `/contract` skill
**For smart contract testing:** Use `/contract-test` skill

## Commands

Prefer `sbtn` over `sbt`, when `sbtn` is aviable and not hangs. 

| Command          | Purpose                                              |
|------------------|------------------------------------------------------|
| `sbtn quick`     | Format, compile, jvm/testQuick (fast iteration)      |
| `sbtn testQuick` | Run only tests affected by recent changes            |
| `sbtn ci`        | Full CI: format check, all platforms, docs, mima     |
| `sbtn it`        | Integration tests (requires Docker)                  |

**Compilation:**

- `sbtn jvm/Test/compile` - all JVM projects with tests
- `sbtn scalusJVM/Test/compile` - scalus-core JVM with tests
- `sbtn scalusCardanoLedgerJVM/compile` - cardano-ledger JVM without tests
- `sbtn "clean; Test/compile"` - full clean compilation, useful for checking for warnings. Otherwise
  incremental compilation could not show warnings in already compiled files

**Testing:**

- `sbtn jvm/test` - all JVM tests
- `sbtn jvm/testQuick` - run only tests affected by recent changes (incremental)
- `sbtn scalusJVM/test` - core tests
- `sbtn scalusExamplesJVM/test` - example tests
- `sbtn scalusCardanoLedgerIt/test` - integration tests

**Note:** sbt uses incremental compilation - only changed files and their dependents are recompiled.
Use `sbtn clean` to force full recompilation if you encounter stale class issues.

### Working in a git worktree with sbt

When the repo is checked out in a **git worktree** (`git worktree add ...`, e.g. `.claude/worktrees/<name>`),
sbt fails to load out of the box. Two one-time fixes are needed inside the worktree:

1. **`sbt-git` / JGit cannot read the worktree's `.git` file.** Loading the build throws
   `org.eclipse.jgit.errors.NoWorkTreeException: Bare Repository has neither a working tree, nor an index`
   (the bundled JGit 5.13 does not understand a worktree's `gitdir:`-pointer `.git`). Work around it by
   adding a local, **untracked** `.sbt` file at the worktree root that short-circuits the eager git
   settings with constants (do NOT commit it):

   ```scala
   // zz-worktree-git-override.sbt  — local worktree workaround, do not commit
   import com.github.sbt.git.SbtGit.git
   ThisBuild / git.gitUncommittedChanges := false
   ThisBuild / git.gitHeadCommit := Some("0000000000000000000000000000000000000000")
   ThisBuild / git.gitCurrentTags := Nil
   ThisBuild / git.gitCurrentBranch := "<your-branch>"
   ThisBuild / git.gitHeadMessage := None
   ```

   Add `zz-worktree-git-override.sbt` to your worktree's `.git/info/exclude` (or just leave it unstaged)
   so it never lands in a commit.

2. **The `plutus-conformance` corpus is not present in the worktree.** `scalusJVM/Test/compile` fails with
   `Plutus conformance corpus not found at .../plutus-conformance/test-cases/uplc/evaluation` (it is a
   symlink into the nix store that only exists in the primary checkout). Recreate the symlink in the
   worktree, pointing at the same target as the primary checkout:

   ```bash
   ln -s "$(readlink /path/to/primary/checkout/plutus-conformance)" <worktree>/plutus-conformance
   ```

   It is git-ignored, so it won't show up in `git status`.

## Architecture

### Compilation Pipeline

Scala Source → SIR (via compiler plugin) → UPLC (via SIR compiler) → Plutus Script (V1/V2/V3)

### Module Structure

**Cross-platform modules (JVM/JS, some with Native):**

| Module                         | Purpose                           | sbt Project                                       |
|--------------------------------|-----------------------------------|---------------------------------------------------|
| `scalus-core/`                 | Plutus VM, UPLC, standard library | `scalusJVM`, `scalusJS`, `scalusNative`           |
| `scalus-cardano-ledger/`       | Transaction building, ledger      | `scalusCardanoLedgerJVM`, `scalusCardanoLedgerJS` |
| `scalus-testkit/`              | Testing utilities                 | `scalusTestkitJVM`, `scalusTestkitJS`             |
| `scalus-examples/`             | Smart contract examples           | `scalusExamplesJVM`, `scalusExamplesJS`           |

**JVM-only modules:**

| Module                         | Purpose                           | sbt Project                         |
|--------------------------------|-----------------------------------|-------------------------------------|
| `scalus-plugin/`               | Scala 3 compiler plugin           | `scalusPlugin`                      |
| `scalus-plugin-tests/`         | Plugin test suite                 | `scalusPluginTests`                 |
| `scalus-uplc-jit-compiler/`    | Experimental UPLC JIT compiler    | `scalusUplcJitCompiler`             |
| `scalus-design-patterns/`      | Design pattern examples           | `scalusDesignPatterns`              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scalus3/scalus](https://github.com/scalus3/scalus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
