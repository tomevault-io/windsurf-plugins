---
trigger: always_on
description: AgentENV is a Rust workspace for running AI agents inside isolated, snapshot-capable Firecracker-based environments. It exposes an E2B-compatible HTTP API so agents can create, pause, resume, and reuse sandboxes. Requires a Linux host with `/dev/kvm` access.
---

# CLAUDE.md

## What is AgentENV

AgentENV is a Rust workspace for running AI agents inside isolated, snapshot-capable Firecracker-based environments. It exposes an E2B-compatible HTTP API so agents can create, pause, resume, and reuse sandboxes. Requires a Linux host with `/dev/kvm` access.

## Build, Lint, Test Commands

```bash
make                          # build the workspace
make fmt                      # rustfmt check (agentenv, envd, uvm-ublk, uvm-ublk-daemon)
make clippy                   # clippy with -D warnings
make test                     # full test suite (agent + envd + ublk)
make test-unit                # unit tests only
make test-agent-integration   # integration tests (tests/integration/*.rs)
make bench                    # snapshot benchmarks
make start-server             # build and run the API server (auto-provisions dependencies)
```

Dev/CI tooling via `cargo adev` (delegated from Makefile):
```bash
cargo adev codegen            # regenerate all OpenAPI clients/server
cargo adev mutants            # run mutation tests
cargo adev coverage           # run code coverage
make firecracker-client       # shorthand for cargo adev codegen firecracker
make envd-http-client         # shorthand for cargo adev codegen envd
make agentenv-server          # shorthand for cargo adev codegen server
make custom-extension-client  # shorthand for cargo adev codegen custom-extension
```

Dependency downloads, generated OverlayBD runtime configs, and OverlayBD packaging are provisioned automatically during server startup. Machine-wide KVM group access, ublk device permissions, OverlayBD system config, and network sysctls require a one-time root setup via `server --setup-host --runtime-user <user> --runtime-group <group>`; normal startup validates those prerequisites and fails with actionable errors when they are missing.

All registry access goes through `regctl`: userImage manifest fetch, config blob fetch, layer download, tools drive image download (`src/setup/deps.rs::extract_ext4_from_ghcr`, unpacked with `umoci`), and OCI referrers lookup when `[image_resolver].try_referrers_overlaybd_prefixes` is non-empty (referrers lookup failures fall back to the source image). Server setup provisions both automatically: `regctl` is downloaded from the `[regclient]` entry in `config/deps_manifest.toml` to `/usr/local/bin/regctl`, and `umoci` is installed as a `[packages.runtime]` system package. `src/image/oci_image.rs` fetches the manifest via `regctl manifest get` and classifies it — standard OCI tar images trigger a full `regctl image copy` + per-layer conversion into local `.commit` files, while overlaybd-native images skip blob download entirely and emit a remote-ref `image.json` that the overlaybd runtime's `registryfs_v2` backend reads directly from the registry. User-facing image references are normalized by `ImageResolver` from template API `userImage` fields and CLI image arguments. For private registries referenced by `userImage`, run `docker login <registry>` before starting the server; `write_generated_overlaybd_global_config` auto-detects `~/.docker/config.json` (or `$DOCKER_CONFIG/config.json`) and wires the overlaybd runtime's `credentialConfig.mode=file` so the runtime can authenticate too.

P2P artifact transport (`src/p2p/`) is a project-wide, optional node-to-node artifact layer. Consumers depend on `P2pTransport` rather than a concrete backend. `DisabledP2pTransport` is the default no-op implementation; `IrohBlobsP2pTransport` embeds an `iroh` endpoint and `iroh-blobs` `FsStore`, serving bytes, byte ranges, and a small AgentENV catalog protocol from the AgentENV server process. Configure it with `[p2p]`. When enabled for overlaybd, the server also starts a localhost HTTP facade: `/p2p-http/{*origin}` is patched into overlaybd registryfs for foreground range reads, while `/p2p-control/publish-layer` lets the ublk daemon publish completed background-downloaded layers as full-layer artifacts by reference. Overlaybd layer artifact identity is owned by `src/overlaybd/p2p/artifact.rs` (`overlaybd-layer/v1/sha256:<digest>` plus `LayerMetadata`); snapshot publishing must reuse that helper instead of inventing snapshot-specific layer keys. Snapshot publishing also advertises fixed artifacts under `snapshot/v1/artifacts/{snapshot_id}/...` after repository commit; OSS runtime resolution tries P2P before object storage for those fixed artifacts, while POSIX resolution does not consume P2P. Scheduler integration covers endpoint discovery and a lightweight in-memory artifact-to-node index: heartbeats advertise the local `P2pEndpoint`, `ListP2pPeers` returns ready peers for a backend, `RecordP2pArtifact`/`ForgetP2pArtifact`/`LookupP2pArtifact` maintain a key-to-node hint index that accelerates artifact lookup before falling back to broad peer polling. Scheduler stores only key-to-node mappings and never stores artifact locators, metadata, or proxies bytes. Node unregister removes all artifact mappings for that node. See `docs/src/internals/p2p-design.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kvcache-ai/AgentENV](https://github.com/kvcache-ai/AgentENV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
