---
trigger: always_on
description: This repository is `helm-secrets`, a Helm plugin implemented mostly in POSIX shell. It decrypts Helm value files on demand, supports secret references through multiple backends, and integrates with Helm downloader and post-renderer plugin mechanisms.
---

# AGENTS.md

This repository is `helm-secrets`, a Helm plugin implemented mostly in POSIX shell. It decrypts Helm value files on demand, supports secret references through multiple backends, and integrates with Helm downloader and post-renderer plugin mechanisms.

## Project Map

- `plugin.yaml` is the Helm 3 plugin manifest. It registers the `secrets` command and downloader protocols.
- `plugins/helm-secrets-cli`, `plugins/helm-secrets-getter`, and `plugins/helm-secrets-post-renderer` contain Helm 4 plugin manifests. Helm 4 currently treats CLI, getter, and post-renderer plugins as separate plugin types, so these manifests are split even though Helm 3 can declare the command and downloader protocols together in `plugin.yaml`. Keep versions and user-facing help synchronized with `plugin.yaml` and `scripts/commands/help.sh`.
- `scripts/run.sh` is the main plugin entrypoint. It initializes globals, loads libraries/backends, parses top-level `helm secrets` options, then dispatches to command scripts.
- `scripts/commands/` contains subcommands and Helm integration:
  - `encrypt.sh`, `decrypt.sh`, `edit.sh` implement direct file operations.
  - `helm.sh` wraps arbitrary Helm commands, decrypts `-f`, `--values`, `--set-file`, and decrypts literals in `--set`, `--set-string`, and `--set-json`.
  - `downloader.sh` implements `secrets://`, `secrets+gpg-import://`, `secrets+gpg-import-kubernetes://`, `secrets+age-import://`, `secrets+age-import-kubernetes://`, and `secrets+literal://`.
  - `post-renderer.sh` evaluates `vals` references in rendered manifests when `--evaluate-templates` is enabled.
  - `help.sh` and `version.sh` provide user-facing CLI output.
- `scripts/lib/` contains shared shell functions for logging, traps, path handling, backend dispatch, file retrieval, HTTP downloads, and strict variable expansion.
- `scripts/lib/backends/` contains in-tree backends:
  - `sops.sh` is the default backend. It can encrypt, decrypt, edit, and detect SOPS-encrypted content.
  - `vals.sh` resolves `ref+...` references. It does not support encrypt or edit.
  - `noop.sh` passes files through for tests and non-encrypted workflows.
  - `_custom.sh` is a helper contract for out-of-tree backends.
- `scripts/lib/file/` abstracts value source retrieval:
  - `local.sh` handles normal files.
  - `http.sh` downloads `http://` and `https://` values with `curl` or `wget`.
  - `custom.sh` delegates arbitrary `*://` sources to Helm through a tiny chart in `helm-values-getter`.
- `scripts/wrapper/` contains wrapper scripts for Windows and optional automatic `helm secrets` forwarding.
- `docs/` is the wiki-style documentation source. Update docs when CLI flags, environment variables, security behavior, or integration behavior changes.
- `examples/` contains sample charts and backend scripts for SOPS, vals, Argo CD, Terraform, and custom backends.
- `tests/` contains first-party Bats tests and assets. `tests/bats/` is vendored/submodule test tooling; do not edit it unless intentionally updating submodules.

## Runtime Flow

1. Helm invokes `scripts/run.sh` through a plugin manifest.
2. `run.sh` sets `HELM_BIN`, `SCRIPT_DIR`, `TMPDIR`, default backend (`sops`), quiet mode, decrypted file naming settings, and feature flags from `HELM_SECRETS_*` environment variables.
3. It loads `common.sh`, `expand_vars_strict.sh`, `file.sh`, `backend.sh`, and `http.sh`, then calls `load_secret_backend`.
4. Top-level arguments are parsed before dispatch. Global flags such as `--backend`, `--backend-args`, `--quiet`, `--ignore-missing-values`, `--evaluate-templates`, and `--decrypt-secrets-in-tmp-dir` affect later command behavior.
5. Direct commands call backend helpers through `backend.sh`. Wrapped Helm commands source `commands/helm.sh`.
6. `helm_wrapper` rewrites Helm arguments:
   - For `-f`/`--values`, it fetches the source, decrypts encrypted files into `.dec` files or temp files, and passes decrypted paths to Helm.
   - For `secrets://...` values, prefer passing the protocol URL through to Helm so Helm's downloader plugin handles it directly. Avoid resolving `secrets://` through `_file_get` in the wrapper unless there is a specific compatibility reason and regression coverage for trailing newlines.
   - For `--set-file`, it decrypts file contents and preserves Helm key prefixes.
   - For `--set`, `--set-string`, and `--set-json`, it resolves encrypted literal values and preserves escaped commas, lists, and trailing newlines.
   - It records generated decrypted files and removes them in `_trap_hook`.
7. Downloader protocol handling in `downloader.sh` prints decrypted content to stdout for Helm downloader usage. Key-import protocols initialize temporary GPG homes or `SOPS_AGE_KEY_FILE` before decrypting.
8. If template evaluation is enabled, `helm.sh` injects a Helm post-renderer. Helm 3 invokes `helm secrets post-renderer`; Helm 4 uses the separate `secrets-post-renderer` plugin because Helm 4 plugin types are split.

## Backend Contract

Backends are dispatched by name through functions in `scripts/lib/backend.sh`. A backend named `foo` must provide:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkroepke/helm-secrets](https://github.com/jkroepke/helm-secrets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
