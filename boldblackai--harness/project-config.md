---
trigger: always_on
description: **Harness** is a portable containerized environment for running coding agents. See README.md for more project details.
---

# harness

## Project Overview

**Harness** is a portable containerized environment for running coding agents. See README.md for more project details.

**Documentation website:** Built with [Zensical](https://zensical.org) from `docs/`. Config in `zensical.toml`. Deploys to GitHub Pages via `.github/workflows/docs.yml`, which also runs a build-only check on PRs that touch `docs/`, `zensical.toml`, or the workflow itself (so doc build failures are caught before merge). Build locally with `pip install zensical && zensical build --clean` (CI uses `uv tool install zensical==0.0.43`).

## Commands

```bash
pnpm build            # Compile TypeScript → bin/harness.js
make build            # Same via Makefile
make image            # Build all Docker images (base + opencode + hermes variants)
make image-base       # Build base image only
make image-opencode   # Build opencode variant
make image-hermes     # Build hermes variant
pnpm link --global    # Make `harness` CLI available globally for local testing
pnpm lint             # Run all linters (biome, markdownlint, shellcheck, hadolint, actionlint)
pnpm format           # Auto-format with Biome
pnpm test:e2e         # Run e2e CLI tests (uses a docker shim, no real docker needed)
```

System linters (`shellcheck`, `hadolint`, `actionlint`) must be installed separately (`brew install shellcheck hadolint actionlint`).

## Architecture

All CLI logic lives in `src/harness.ts` (compiles to `bin/harness.js`). It:

1. Parses CLI args via `minimist`
2. Selects an adapter (`PiAdapter`, `OpenCodeAdapter`, or `HermesAdapter`) based on `--agent` flag
3. Constructs and spawns a `<runtime> run` command (`docker` by default, or Apple's `container` when `HARNESS_CONTAINER_RUNTIME=apple`) that mounts `$PWD` and passes the prompt via stdin or `-e`

**Adapter pattern:** Each adapter implements how to invoke the agent binary inside the container (command, flags, env vars). Adding a new agent means adding a new adapter class and registering it in the `ADAPTERS` map.

### Image structure

The project uses a **multi-image architecture** with a shared base and agent-specific variants:

| Image | Dockerfile | Tag pattern | Contents |
|-------|-----------|-------------|----------|
| Base (pi) | `Dockerfile` | `<version>` | Debian stable-slim, Node.js v24, pnpm, `git`, `pi-coding-agent`, `gh`, `mise`, `tini`, `fd`, `ripgrep`, `jq` |
| OpenCode | `Dockerfile.opencode` | `opencode-<version>` | Base + `opencode-ai` |
| Hermes | `Dockerfile.hermes` | `hermes-<version>` | Base + `uv`, `cosign`, `tirith`, Python venv with `hermes-agent` (incl. MCP SDK), `python-telegram-bot`, `croniter`, `faster-whisper` |

The image tag is selected at runtime based on `--agent`: pi uses `<version>`, others use `<agent>-<version>`.

### Key subsystems

**Cosign image verification (`verifyImage`):** On every run (unless `--no-verify` or `HARNESS_IMAGE_TAG` is set), harness verifies the container image was signed by the official CI workflow and carries a valid SLSA provenance attestation. Verified digests are cached at `~/.cache/harness/cosign-verified.json`. Requires `cosign` installed on the host.

**Container runtime selection (`HARNESS_CONTAINER_RUNTIME`):** The host container runtime is abstracted behind a `ContainerRuntime` interface in `src/harness.ts` with two implementations: `DockerRuntime` (default, reproduces the original argv byte-for-byte) and `AppleContainerRuntime` (Apple's native `container` CLI for macOS 26 / Apple Silicon). Selection is case-insensitive over named values — `docker` (default/unset) or `apple`; any other value is a hard error. The runtime owns the binary name, the pull subcommand (`docker pull` vs `container image pull`), local digest lookup (`docker image inspect --format` Go-template vs `container image inspect` JSON → `data[0].configuration.descriptor.digest`, rebuilt into `repo@sha256:<digest>`), and the final `run` argv. The apple path emits `-i`/`-t` separately (no clustered `-it`), space-separated capability flags (`--cap-drop ALL` not `--cap-drop=ALL`), and **omits** `--security-opt` entirely — `apple/container` has no such option, and its microVM isolation (per-workload guest kernel under Apple's Virtualization framework) subsumes the host-kernel role of the `block-af-alg.json` seccomp profile; `--cap-drop=ALL --cap-add=NET_RAW` stays (capabilities are supported). A `container --version` prerequisite probe runs before any spawn and prints an install hint if the binary is missing. Everything else (adapters, `persistMounts`, skills/context-file mounts, ephemeral logic, cloud-mode env, image-tag selection, entrypoints, Dockerfiles, CI) is runtime-agnostic and unchanged. The cosign verified-digest cache is keyed by digest, not runtime, so a digest verified under one runtime is a cache hit under the other.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boldblackai/harness](https://github.com/boldblackai/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
