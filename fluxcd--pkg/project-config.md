---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/pkg`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/pkg`. Read this file before making changes.

## Contribution workflow for AI agents

These rules come from [`fluxcd/flux2/CONTRIBUTING.md`](https://github.com/fluxcd/flux2/blob/main/CONTRIBUTING.md) and apply to every Flux repository.

- **Do not add `Signed-off-by` or `Co-authored-by` trailers with your agent name.** Only a human can legally certify the DCO.
- **Disclose AI assistance** with an `Assisted-by` trailer naming your agent and model:
  ```sh
  git commit -s -m "Add support for X" --trailer "Assisted-by: <agent-name>/<model-id>"
  ```
  The `-s` flag adds the human's `Signed-off-by` from their git config — do not remove it.
- **Commit message format:** Subject in imperative mood ("Add feature X" instead of "Adding feature X"), capitalized, no trailing period, ≤50 characters. Body wrapped at 72 columns, explaining what and why. No `@mentions` or `#123` issue references in the commit — put those in the PR description.
- **Trim verbiage:** in PR descriptions, commit messages, and code comments. No marketing prose, no restating the diff, no emojis.
- **Rebase, don't merge:** Never merge `main` into the feature branch; rebase onto the latest `main` and push with `--force-with-lease`. Squash before merge when asked.
- **Pre-PR gate:** `make test-<module>` must pass for every module you touched. Run `make tidy` to tidy all affected modules.
- **Flux is GA:** Backward compatibility is mandatory. These modules are consumed by all Flux controllers — breaking changes to exported APIs, function signatures, or behavior will be rejected. Design additive changes.
- **Copyright:** All new `.go` files must begin with the Apache 2.0 boilerplate header. Update the year to the current year when copying.
- **Tests:** New features, improvements and fixes must have test coverage. Follow existing patterns in the module you're modifying. Run tests locally before pushing.

## Code quality

Before submitting code, review your changes for the following:

- **No unchecked I/O.** Close HTTP response bodies, file handles, and archive readers in `defer` statements. Check and propagate errors from I/O operations.
- **No path traversal.** The `tar` module uses `cyphar/filepath-securejoin` — always extract archives through it. Never `filepath.Join` with untrusted components without validation.
- **No command injection.** Do not shell out via `os/exec`. Use Go libraries for git, OCI, and cloud operations.
- **No hardcoded defaults for security settings.** TLS verification must remain enabled by default.
- **Error handling.** Wrap errors with `%w` for chain inspection. Do not swallow errors silently. Return errors that help callers diagnose the issue without leaking internal state.
- **Resource cleanup.** Ensure temporary files and directories are cleaned up on all code paths. Use `defer` and `t.TempDir()` in tests.
- **No panics.** Never use `panic` in library code. Return errors and let callers decide how to handle them.
- **Thread safety.** These packages are used in concurrent reconcilers. Do not introduce shared mutable state without synchronization.
- **Minimal surface.** Every exported type, function, and method is a backward-compatibility commitment consumed by multiple controllers. Minimize new exports.

## Project overview

`fluxcd/pkg` is the shared Go SDK for the Flux GitOps Toolkit. It is a **multi-module monorepo** — there is no top-level `go.mod`. Each subdirectory is its own independently versioned Go module, tagged separately (e.g. `runtime/v0.103.0`, `ssa/v0.23.0`, `apis/meta/v1.26.0`). All Flux controllers import specific modules from this repo.

The repository provides: controller runtime helpers, server-side apply engine, git operations, cloud auth/workload identity, OCI operations, kustomize building, artifact storage, and shared API types.

## Repository layout

There is **no top-level `go.mod`**. Each directory is its own module:

- `apis/meta/` — foundational API types: standard conditions (`Ready`, `Stalled`, `Reconciling`), reasons, annotations (`ReconcileRequestAnnotation`), artifact spec, dependency references.
- `apis/event/` — Flux event schema dispatched to notification-controller.
- `apis/acl/` — cross-namespace access control types.
- `apis/kustomize/` — Kustomize-related API types (e.g. `HealthCheckExpressions`).
- `runtime/` — largest module. Sub-packages: `conditions`, `patch`, `reconcile`, `events`, `metrics`, `features`, `cel`, `acl`, `controller`, `dependency`, `errors`, `jitter`, `leaderelection`, `logger`, `object`, `predicates`, `probes`, `pprof`, `secrets`, `statusreaders`, `testenv`, `transform`, `client`.
- `ssa/` — server-side apply engine (`ResourceManager`): apply, diff, wait, delete, change sets. Sub-packages: `jsondiff`, `normalize`, `errors`, `utils`.
- `git/` — git operations. `gogit/` sub-package is the concrete go-git implementation. `repository/` defines `Reader`/`Writer` interfaces.
- `auth/` — cloud workload identity: `aws/`, `azure/`, `gcp/`, `generic/`, `githubapp/`, `utils/`. Central `GetAccessToken()` with caching.
- `artifact/` — artifact storage: `config/`, `digest/`, `server/`, `storage/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/pkg](https://github.com/fluxcd/pkg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
