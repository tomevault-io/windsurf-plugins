---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/image-reflector-controller`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/image-reflector-controller`. Read this file before making changes.

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
- **Pre-PR gate:** `make tidy fmt vet && make test` must pass and the working tree must be clean after codegen. Commit regenerated files in the same PR.
- **Flux is GA:** Backward compatibility is mandatory. Breaking changes to CRD fields, status, CLI flags, metrics, or observable behavior will be rejected. Design additive changes and keep older API versions round-tripping.
- **Copyright:** All new `.go` files must begin with the boilerplate from `hack/boilerplate.go.txt` (Apache 2.0). Update the year to the current year when copying.
- **Spec docs:** New features and API changes must be documented in `docs/spec/v1/` — `imagepolicies.md` and `imagerepositories.md`. Update the relevant file in the same PR that introduces the change.
- **Tests:** New features, improvements and fixes must have test coverage. Add unit tests in `internal/controller/*_test.go` and other `internal/*` packages as appropriate. Follow the existing patterns for test organization, fixtures, and assertions. Run tests locally before pushing.

## Code quality

Before submitting code, review your changes for the following:

- **No secrets in logs or events.** Never surface registry credentials, cloud provider tokens, or pull-secret contents in error messages, conditions, events, or log lines.
- **No unchecked I/O.** Close HTTP response bodies, file handles, and registry connections in `defer` statements. Check and propagate errors from I/O operations.
- **No unbounded reads.** Use `io.LimitReader` when reading from network sources. Repositories with very large tag sets (tens of thousands) consume significant memory; respect existing limits.
- **No direct Badger imports from reconcilers.** Database access goes through the `DatabaseReader`/`DatabaseWriter` interfaces in `internal/controller/database.go`. That boundary is load-bearing for testability.
- **Registry rate limits.** New scan paths must go through the existing reconcile rate limiter and the `TokenCache`. Do not call `remote.List` in tight loops — public registries (Docker Hub in particular) rate-limit aggressively.
- **Auth through `AuthOptionsGetter`.** Registry auth resolution lives in `internal/registry/options.go`. Add new auth knobs there, not inline in the reconciler. Cloud provider tokens must use the shared `TokenCache`.
- **Error handling.** Wrap errors with `%w` for chain inspection. Do not swallow errors silently. Return actionable error messages that help users diagnose the issue without leaking internal state.
- **Resource cleanup.** Ensure temporary files and directories are cleaned up on all code paths (success and error). Use `defer` and `t.TempDir()` in tests.
- **Concurrency safety.** Do not introduce shared mutable state without synchronization. Reconcilers run concurrently; per-object work must be isolated. Leader election is required because BadgerDB does not support concurrent access from multiple replicas.
- **No panics.** Never use `panic` in runtime code paths. Return errors and let the reconciler handle them gracefully.
- **Minimal surface.** Keep new exported APIs, flags, and environment variables to the minimum needed. The `api/` module is consumed by image-automation-controller — every exported change is a cross-repo contract change.

## Project overview

image-reflector-controller is a component of the [Flux GitOps Toolkit](https://fluxcd.io/flux/components/). It reconciles two CRDs under `image.toolkit.fluxcd.io/v1`:

- `ImageRepository` — scans an OCI image repository at a fixed interval, lists the tags via the registry API, and persists them.
- `ImagePolicy` — references an `ImageRepository`, filters its tag set (optional regex), and elects a "latest" tag using one of three policies: `semver` (Masterminds/semver ranges), `alphabetical`, or `numerical`. It can also reflect the image digest for the elected tag (`digestReflectionPolicy: Never | IfNotPresent | Always`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/image-reflector-controller](https://github.com/fluxcd/image-reflector-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
