---
trigger: always_on
description: Collet is a modular Clojure workflow engine for declarative ETL and ELT
---

# Repository Guidelines

## Project Snapshot

Collet is a modular Clojure workflow engine for declarative ETL and ELT
pipelines. The repository is a Clojure CLI workspace resolved by Kmono: each
runtime library or executable has its own `deps.edn`, tests, artifact contract,
and independent release tag.

This is not a single-classpath monolith. Keep implementation, dependencies,
tests, resources, and artifact metadata in the package that owns them.

## Start Here

- Read `README.md` for the pipeline and action model.
- Read `docs/development.md` before changing packages, dependencies, builds, or
  releases.
- Read `docs/repl-workflow.md` before editing Clojure.
- Read only the task-relevant action, syntax, migration, or release docs.
- Treat this file as the routing layer, not the complete manual.

## Repository Map

| Area | Paths |
| --- | --- |
| Pipeline engine | `collet-core/src/`, `collet-core/test/` |
| Individual actions | `collet-action-*/src/`, `collet-action-*/test/` |
| Compatibility aggregate | `collet-actions/` |
| JVM application | `collet-app/` |
| Babashka pod and CLI | `collet-cli/` |
| Shared test fixtures | `test-fixtures/` |
| Build and release tooling | `build/src/`, `build/test/`, `bb.edn` |
| Examples and docs | `examples/`, `docs/`, `README.md` |
| Agent tooling | `dev/user.clj`, `scripts/agent/` |

## Golden Commands

Run repository-level commands from the repository root.

| Task | Command |
| --- | --- |
| Start the worktree nREPL | `clojure -M:dev:nrepl` |
| Discover nREPL ports | `clj-nrepl-eval --discover-ports` |
| Discover ports with the fallback client | `bb scripts/agent/nrepl-eval.bb --discover` |
| Reload changed project code | `clj-nrepl-eval -p <port> "(user/reload)"` |
| Inspect semantic code index | `jbcontext status` |
| Semantically search repository code | `jbcontext search "<query>"` |
| Repair Clojure delimiters | `clj-paren-repair path/to/file.clj` |
| Check Clojure formatting | `bb fmt:check --file path/to/file.clj` |
| Format Clojure files | `bb fmt:fix --file path/to/file.clj` |
| Inspect the package graph | `bb kmono query` |
| Test one package | `bb test:module <module>` |
| Run non-integration tests | `bb test:unit` |
| Run Docker integration tests | `bb test:integration` |
| Run every test | `bb test` |
| Build packages | `bb build [module]` |
| Verify artifact contracts | `bb verify` |
| Inspect the release plan | `bb release:plan` |

Package selectors are directory names such as `collet-core`,
`collet-action-http`, `collet-actions`, `collet-app`, and `collet-cli`.

## Operating Rules

- Preserve user-owned changes. Inspect a dirty file before editing it, and do
  not revert, overwrite, or clean unrelated work.
- Do not run destructive file or Git operations unless the user explicitly
  requested the exact operation.
- Keep one logical change per branch or worktree. Use the active worktree for
  every command so REPL state, source paths, and Git state agree.
- Keep runtime changes and package-specific tests/resources in the owning
  package. Put reusable unpublished test helpers in `test-fixtures`.
- For Datalevin reads of related entities, use one Datalog query with an inline
  `pull` (or another set-based read); do not query IDs and then pull each
  entity individually.
- Internal source dependencies stay as top-level `:local/root` entries. Do not
  replace them with Maven versions or edit generated POMs.
- Root build, CI, documentation, agent, and development files do not select a
  package release. If a packaging change alters an artifact, change the owning
  package metadata in the same release-producing commit.
- Never run `bb release` unless the user explicitly requests publication.

## Clojure and REPL Rules

- Clojure work is REPL-first. Before editing any Clojure source, test, fixture,
  or build namespace, attach to or start an nREPL from the active worktree.
- Discover servers with `clj-nrepl-eval --discover-ports`. If the global client
  is unavailable, run
  `bb scripts/agent/nrepl-eval.bb --discover`.
- If no suitable worktree REPL is running, start
  `clojure -M:dev:nrepl` in a long-running terminal, then rediscover its port.
- If a sandbox denies a loopback socket with `Operation not permitted`, rerun
  the exact start or eval command through the environment's approval mechanism.
  The repository fallback client cannot bypass OS or sandbox permissions.
- Apply runtime and shared-fixture edits with
  `clj-nrepl-eval -p <port> "(user/reload)"`. The fallback form is:

  ```shell
  bb scripts/agent/nrepl-eval.bb \
    --port <port> \
    --code "(user/reload)"
  ```

- The root reload scope is every existing `collet-*/src` directory plus
  `test-fixtures/src`. Package tests and `build/src` remain authoritative
  through their isolated CLI test commands.
- Never use `:reload-all` or `tools.namespace/refresh-all`. They can re-evaluate
  library protocols and orphan live instances.
- Recreate pipelines, records, deftypes, protocol implementations, publishers,
  and other stateful values after reload. Restart the JVM if a value is bound
  to a stale class or protocol identity.
- Keep evals small, deterministic, and bounded. Do not run destructive file,
  Git, publication, or network actions through nREPL without explicit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [velio-io/collet](https://github.com/velio-io/collet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
