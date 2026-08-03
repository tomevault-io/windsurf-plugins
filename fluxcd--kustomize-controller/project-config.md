---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/kustomize-controller`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/kustomize-controller`. Read this file before making changes.

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
- **Spec docs:** New features and API changes must be documented in `docs/spec/v1/kustomizations.md`. Update it in the same PR that introduces the change.
- **Tests:** New features, improvements and fixes must have test coverage. Add unit tests in `internal/controller/*_test.go` and other `internal/*` packages as appropriate. Follow the existing patterns for test organization, fixtures, and assertions. Run tests locally before pushing.

## Code quality

Before submitting code, review your changes for the following:

- **No secrets in logs or events.** Never surface decryption keys, auth tokens, passwords, or credential URLs in error messages, conditions, events, or log lines. SOPS key material (PGP, age, KMS credentials) must never be logged or returned in errors.
- **No unchecked I/O.** Close HTTP response bodies, file handles, and archive readers in `defer` statements. Check and propagate errors from `io.Copy`, `os.Remove`, `os.Rename`.
- **No path traversal.** Validate and sanitize file paths extracted from source artifacts. Kustomize builds must stay within the source root; respect `--no-remote-bases` to block remote base fetches.
- **No unbounded reads.** Use `io.LimitReader` when reading from network or archive sources. Respect existing size limits; do not introduce new reads without bounds.
- **No command injection.** Do not shell out via `os/exec`. Kustomize builds go through `fluxcd/pkg/kustomize`, not the `kustomize` CLI.
- **No hardcoded defaults for security settings.** TLS verification must remain enabled by default; impersonation and cross-namespace ref restrictions must be honored.
- **SSA field manager integrity.** Always use the controller's field manager via `fluxcd/pkg/ssa`. Never hard-code a different manager or strip fields out of band — changing it orphans fields on every managed object.
- **Error handling.** Wrap errors with `%w` for chain inspection. Do not swallow errors silently. Return actionable error messages that help users diagnose the issue without leaking internal state.
- **Resource cleanup.** Ensure temporary files, directories, and fetched artifacts are cleaned up on all code paths (success and error). Use `defer` and `t.TempDir()` in tests.
- **Concurrency safety.** Do not introduce shared mutable state without synchronization. Reconcilers run concurrently; per-object work must be isolated.
- **No panics.** Never use `panic` in runtime code paths. Return errors and let the reconciler handle them gracefully.
- **Minimal surface.** Keep new exported APIs, flags, and environment variables to the minimum needed. Every export is a backward-compatibility commitment.

## Project overview

kustomize-controller is a core component of the [Flux GitOps Toolkit](https://fluxcd.io/flux/components/). It reconciles the `Kustomization` CRD (`kustomize.toolkit.fluxcd.io`). For each object it:

1. Fetches a source artifact produced by source-controller from a `GitRepository`, `Bucket`, `OCIRepository`, or (feature-gated) generic `ExternalArtifact`.
2. Runs a Kustomize build against the referenced path, with optional `postBuild` variable substitution (envsubst).
3. Decrypts SOPS-encrypted Kubernetes secrets inline using keys from PGP, age, OpenBao/Vault, AWS KMS, GCP KMS, or Azure Key Vault.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/kustomize-controller](https://github.com/fluxcd/kustomize-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
