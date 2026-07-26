---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/flux2`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/flux2`. Read this file before making changes.

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
- **Pre-PR gate:** `make tidy fmt vet && make test` must pass and the working tree must be clean.
- **Flux is GA:** Backward compatibility is mandatory. Breaking changes to CLI flags, output format, or behavior will be rejected. Design additive changes.
- **Copyright:** All new `.go` files must begin with the header from `cmd/flux/main.go` (Apache 2.0). Update the year to the current year when copying.
- **Tests:** New features, improvements and fixes must have test coverage. Add unit tests in `cmd/flux/*_test.go` tagged `//go:build unit`. Follow the existing `cmdTestCase` + golden file patterns. Run tests locally before pushing.

## Code quality

Before submitting code, review your changes for the following:

- **No secrets in logs or output.** Never surface auth tokens, passwords, deploy keys, or credential URLs in error messages, log lines, or CLI output. Bootstrap and source-secret commands handle sensitive material — take extra care.
- **No unchecked I/O.** Close HTTP response bodies, file handles, and tar readers in `defer` statements. Check and propagate errors from I/O operations.
- **No path traversal.** Validate and sanitize file paths extracted from archives or user input. Never `filepath.Join` with untrusted components without validation.
- **No command injection.** Do not shell out via `os/exec` for git, helm, or kustomize operations. Use the Go libraries already in use (`fluxcd/pkg/git`, `fluxcd/pkg/kustomize`, `fluxcd/pkg/ssa`).
- **No hardcoded defaults for security settings.** TLS verification must remain enabled by default. Git auth settings come from user-provided secrets.
- **Error handling.** Wrap errors with `%w` for chain inspection. Do not swallow errors silently. CLI errors must be actionable — tell the user what went wrong and how to fix it without leaking internal state.
- **Resource cleanup.** Ensure temporary files and directories (manifest staging, downloaded tarballs) are cleaned up on all code paths (success and error). Use `defer` and `t.TempDir()` in tests.
- **No panics.** Never use `panic` in runtime code paths. Return errors and let the CLI handle them gracefully.
- **Output discipline.** Machine-readable data (tables, YAML, JSON) goes to stdout via `rootCmd.OutOrStdout()`. Human-readable status messages go to stderr via the `stderrLogger`.
- **Minimal surface.** Keep new exported APIs in `pkg/` to the minimum needed. Every export is a backward-compatibility commitment.

## Project overview

flux2 is the Flux CLI (`flux` command) and distribution repository. It is **not** a controller — it consumes CRD APIs from six independent controller repos (source-controller, kustomize-controller, helm-controller, notification-controller, image-reflector-controller, image-automation-controller). It serves two purposes:

1. **CLI tool** — a Cobra-based binary that installs Flux onto Kubernetes clusters, bootstraps GitOps pipelines, and manages all Flux CRD objects (create, get, export, reconcile, suspend, resume, delete, diff, build, etc.).
2. **Distribution hub** — it bundles the Kustomize manifests for all Flux controllers and releases them as `manifests.tar.gz` on GitHub. Those manifests are also compiled into the binary itself via `//go:embed`.

## Repository layout

- `cmd/flux/` — all CLI source. Single `main` package with one file per command or resource type. `main.go` defines the root cobra command with global flags. `manifests.embed.go` embeds the generated controller manifests via `//go:embed`.
- `internal/build/` — `flux build kustomization` logic (kustomize-based diff/build, SOPS secret masking).
- `internal/flags/` — custom `pflag.Value` types providing enum validation (e.g. `LogLevel`, `ECDSACurve`, `RSAKeyBits`, `PublicKeyAlgorithm`, `DecryptionProvider`).
- `internal/tree/` — tree-printing helper for `flux tree kustomization`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/flux2](https://github.com/fluxcd/flux2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
