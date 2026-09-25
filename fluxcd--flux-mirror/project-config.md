---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/flux-mirror`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/flux-mirror`. Read this file before making changes.

## Contribution workflow for AI agents

These rules come from [`fluxcd/flux2/CONTRIBUTING.md`](https://github.com/fluxcd/flux2/blob/main/CONTRIBUTING.md) and apply to every Flux repository.

- **Do not add `Signed-off-by` or `Co-authored-by` trailers with your agent name.** Only a human can legally certify the DCO.
- **Disclose AI assistance** with an `Assisted-by` trailer naming your agent and model:
  ```sh
  git commit -s -m "Add feature X" --trailer "Assisted-by: <agent-name>/<model-id>"
  ```
  Use this only when explicitly asked to commit. The `-s` flag adds the human's `Signed-off-by` from their git config - do not remove it.
- **Commit message format:** Subject in imperative mood ("Add feature X" instead of "Adding feature X"), capitalized, no trailing period, <=50 characters.
- **Commit body:** Add a succinct explanation of what changed and why, wrap at 72 characters.
- **Trim verbiage:** in PR descriptions, commit messages, and code comments. No marketing prose, no restating the diff, no emojis.
- **Rebase, don't merge:** Never merge `main` into the feature branch; rebase onto the latest `main` and push with `--force-with-lease`. Squash before merge when asked.
- **Tests:** New features, improvements and fixes must have test coverage.

## Project

`flux-mirror` is a Flux CLI plugin for declaratively mirroring Helm charts, OCI artifacts, and container images between registries. It is a single Go binary, cobra-based.

Read the [README](README.md) for an overview of the project and its features.

### Code Structure

- `cmd/flux-mirror/` - the `main` package. One file per cobra command or command concern (`sync.go`, `login.go`, `create_secret.go`, `keygen.go`, `version.go`, `completion.go`, `progress.go`, `logging.go`). `main.VERSION` is overridden at build time by the Makefile.
- `cmd/flux-mirror/main_test.go` - hosts `TestMain`, shared `executeCommand(...)` helpers, and `resetCmdArgs()`, which restores global cobra flag state between tests. New commands or flags must update this reset path so tests do not leak state across subtests.
- `api/v1beta1/` - the `mirror.plugin.fluxcd.io/v1beta1` API types: the `Config` wire model (`hosts`, `artifacts`, `charts`, selector, verification) and the sync `Report` envelope, annotated with kubebuilder markers. `zz_generated.deepcopy.go` and the published JSON Schemas are generated from these types; the package stays dependency-light (apimachinery + stdlib) and carries only the wire structs, enums, consts, and `Effective*` helpers — no semver/OCI/SPIFFE deps.
- `internal/config/` - decoding (`Decode`), semantic validation (`Validate`, `ValidateNoEntriesOK`), and path resolution (`ResolvePaths`) for `api/v1beta1.Config` (`apiVersion: mirror.plugin.fluxcd.io/v1beta1`, `kind: Config`). Validation is free functions over the API types, not methods.
- `tools/schema-gen/` - generator that turns controller-gen CRD output into a standalone draft 2020-12 JSON Schema. Run via `make generate`; do not edit `docs/{config,report}/*-v1beta1.json` by hand.
- `internal/selector/` - tag selection pipeline for OCI artifacts: regex prefilter, semver filter, sort strategy (`semver`, `alphabetical`, `numerical`), then top-N limit.
- `internal/sync/` - sync runner, retry/timeout behavior, per-entry execution, summaries, outcomes, and exit-code aggregation. Report assembly (`NewReport`, `RenderReport`) builds the `api/v1beta1.Report` envelope; the wire report types live in `api/v1beta1`.
- `internal/artifacts/` - OCI artifact mirroring from source repository tags to destination repository tags, including drift handling, dry-run outcomes, referrers, verification, and concurrency fan-out.
- `internal/charts/` - Helm chart mirroring to OCI destinations, including version selection, deterministic Helm-OCI publication, drift handling, and dry-run outcomes.
- `internal/oci/` - OCI client wrapper, auth/keychain setup, transport customization, digest checks, blob copy, Helm artifact helpers, referrers, and cosign verification.
- `internal/helmrepo/` - HTTP/S Helm repository access, index/chart resolution, and ambient Helm credential handling.
- `internal/registryauth/`, `internal/jwkio/`, `internal/keygen/` - per-host registry auth, JWT/JWK loading and signing, and JWK key generation.
- `internal/flags/` - reusable `pflag.Value` implementations for CLI flags with constrained values, such as output format.
- `internal/testregistry/` - test helpers for registry-backed mirror tests.
- `actions/setup/` - composite GitHub Action for installing the CLI on CI runners.

### Build, Test, and Lint

All development goes through the Makefile - do not invoke `go build` directly, because the Makefile stamps `main.VERSION` via `-ldflags` and runs `tidy`/`fmt`/`vet` as prerequisites.

- `make build` - build `./bin/flux-mirror` with VERSION stamped from git
- `make test` - runs `tidy`, `fmt`, `generate`, `vet`, then `go test ./... -coverprofile cover.out`
  - Single test pattern: `make test GO_TEST_ARGS="-run TestVersionCmd"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/flux-mirror](https://github.com/fluxcd/flux-mirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
