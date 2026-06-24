---
trigger: always_on
description: Guidance for AI coding assistants working in `fluxcd/flux-schema`. Read this file before making changes.
---

# AGENTS.md

Guidance for AI coding assistants working in `fluxcd/flux-schema`. Read this file before making changes.

## Contribution workflow for AI agents

These rules come from [`fluxcd/flux2/CONTRIBUTING.md`](https://github.com/fluxcd/flux2/blob/main/CONTRIBUTING.md) and apply to every Flux repository.

- **Do not add `Signed-off-by` or `Co-authored-by` trailers with your agent name.** Only a human can legally certify the DCO.
- **Disclose AI assistance** with an `Assisted-by` trailer naming your agent and model:
  ```sh
  git commit -s -m "Add feature X" --trailer "Assisted-by: <agent-name>/<model-id>"
  ```
  The `-s` flag adds the human's `Signed-off-by` from their git config — do not remove it.
- **Commit message format:** Subject in imperative mood ("Add feature X" instead of "Adding feature X"), capitalized, no trailing period, ≤50 characters.
- **Commit body:** Add a succinct explanation explaining what and why, wrap at 72 characters.
- **Trim verbiage:** in PR descriptions, commit messages, and code comments. No marketing prose, no restating the diff, no emojis.
- **Rebase, don't merge:** Never merge `main` into the feature branch; rebase onto the latest `main` and push with `--force-with-lease`. Squash before merge when asked.
- **Tests:** New features, improvements and fixes must have test coverage.

## Project

`flux-schema` is a Flux CLI plugin for Kubernetes schema extraction, manifest validation, and GitOps repository discovery. Single Go binary, cobra-based.

Read the [README](README.md) for an overview of the project and its features.

### Code Structure

- `cmd/flux-schema/` — the `main` package. One file per cobra subcommand (`version.go`, etc.), each registering itself in `init()` via `rootCmd.AddCommand(...)`. `main.VERSION` is overridden at build time by the Makefile.
- `internal/extractor/` — OpenAPI v2 swagger and CRD → standalone-strict JSON Schema extraction. `ExtractKubernetes`/`ExtractOpenShift`/`ExtractCRDs` are the entry points; `transformers.go` holds the pipeline steps (`inlineRefs`, `injectGVK`, `replaceIntOrString`, `nullableOptional`, `closeAdditionalProperties`, `stripVendorExtensions`) plus the exported `StripDescriptions` post-process.
- `internal/validator/` — JSON Schema validation of Kubernetes YAML manifests. `loader.go` compiles schemas via `santhosh-tekuri/jsonschema/v6` with `Draft2020` as the default draft; `formats.go` registers the Kubernetes string formats (`duration`, `date`, etc.) that the library doesn't assert by default.
- `internal/inventory/` — GitOps repository discovery for the `discover` command. `Scan` walks a directory via `os.Root` (reads are OS-confined to the scanned root, symlinks are not followed), classifies directories (`kustomize-overlay`, `helm-chart`, `terraform-module`; chart and Terraform subtrees are pruned), and lists every resource with its defining file; files referenced as kustomize patches are excluded to avoid double counting. `NewInventory` converts the `Result` into the versioned `Inventory` envelope from `api/v1beta1/inventory_types.go`.
- `internal/tmpl/` — Go `text/template` renderer for the output-path and `--schema-location` templates. `SchemaVars` (`Group`, `GroupPrefix`, `Kind`, `Version`) is the shared variable set; values are lowercased at render time and `Group: ""` is normalized to `core`.
- `internal/yamldoc/` — line-oriented `bufio.SplitFunc` that splits a byte stream on `\n---` boundaries. Matches kubectl's `splitYAMLDocument` behavior.
- `internal/flags/` — reusable `pflag.Value` implementations for CLI flags shared across commands.
- `cmd/flux-schema/main_test.go` — hosts `TestMain`, the shared `executeCommand(args)` test helper, and `resetCmdArgs()` which restores every command's flag defaults between tests. New commands must add their flag reset here or tests will leak state across subtests.

### Build, Test, and Lint

All development goes through the Makefile — do not invoke `go build` directly, because the Makefile stamps `main.VERSION` via `-ldflags` and runs `tidy`/`fmt`/`vet` as prerequisites.

- `make build` — build `./bin/flux-schema` with VERSION stamped from git
- `make test` — runs `tidy`, `fmt`, `vet`, then `go test ./... -coverprofile cover.out`
    - Single test: `make test GO_TEST_ARGS="-run TestVersionCmd ./cmd/flux-schema/"`
- `make lint` — runs golangci-lint with revive, staticcheck, and goimports
- `make run GO_RUN_ARGS="version -o json"` — build then run the CLI with args

CI (`.github/workflows/test.yaml`) runs `make test` + `make lint` and fails if the working tree becomes dirty, so always run `make test` before committing.

### Code Conventions

- File header: every `.go` file must start with the two-line Apache-2.0 header — enforced by golangci-lint's `revive.file-header` rule.
- Struct tags: only `json` and `inline` are permitted on struct fields (revive `struct-tag` rule).
- Flag wiring: for any flag with a fixed set of accepted values, add a type under `internal/flags/` and register it with `cmd.Flags().VarP(&args.x, "name", "n", args.x.Description())` rather than `StringVarP` — this gets validation, the `a|b|c` type hint in `--help`, and consistent error messages for free.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fluxcd/flux-schema](https://github.com/fluxcd/flux-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
