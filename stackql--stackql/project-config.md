---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

A scripted post-release step (formerly the standalone `stackql-mcpb-packaging` repo, imported here with history) that packages the StackQL MCP server into per-platform [MCPB](https://github.com/anthropics/mcpb) bundles (`.mcpb`) for distribution and listing on the official MCP Registry. CI is the root workflow [.github/workflows/mcp-packaging.yml](/.github/workflows/mcp-packaging.yml): PR validation on `packaging/**` paths builds + smoke-tests against the latest published release; publishing is a MANUAL `workflow_dispatch` with the target version, because the consumed release assets (Authenticode-signed Windows exe, Apple-notarised darwin `.pkg`) are signed outside CI and attached to the release before dispatch. Upload is same-repo via the default `GITHUB_TOKEN`. The `release.yaml` version pin and the old in-directory workflows are retired; local `make` targets require an explicit `VERSION=X.Y.Z`.

This repo does NOT build or sign the stackql binaries - that happens upstream in the normal stackql build/signing process. Here you drop the already-signed release artefacts (per-arch zips and the notarised macOS `.pkg`) into `bin/`, run one script, and get signed `.mcpb` bundles plus checksums in `dist/` to attach to the matching GitHub release.

The server packed into each bundle is the `stackql` binary itself, launched as `stackql mcp --mcp.server.type=stdio` (see [manifest/manifest.template.json](manifest/manifest.template.json)). The `--mcp.server.type=stdio` flag is required - without it the MCP server does not produce JSON-RPC on stdout. The separate `stackql_mcp_client` binary is a test client and is NOT packaged.

## Common commands

A [Makefile](Makefile) wraps `scripts/package.sh` for the common flows. The script is still the source of truth; `make` is convenience.

`VERSION` defaults to the `stackql_release` value pinned in [release.yaml](release.yaml) (leading `v` stripped), so plain `make all` builds the pinned release. Passing `VERSION=X.Y.Z` overrides it. `release.yaml` is also what CI builds on PRs and what a pushed tag must match to publish (see "Release flow" below).

One-shot from a clean checkout - downloads the release artefacts from `https://github.com/stackql/stackql/releases/download/v<VERSION>/...` into `bin/`, then builds every available bundle:

```bash
make all VERSION=X.Y.Z
# 'make VERSION=X.Y.Z' is equivalent ('all' is the default target)
# 'make' alone uses the version pinned in release.yaml
```

Just download (skip packaging):

```bash
make download VERSION=X.Y.Z
```

Just package whatever is already in `bin/` (skip downloading):

```bash
make package VERSION=X.Y.Z
# or call the script directly:
./scripts/package.sh --version X.Y.Z
```

Build a single target. Two variants:

```bash
# Download just that target's source artefact and build only that bundle.
# Use this on a Mac to do the darwin slice in the two-machine release flow.
make one TARGET=darwin-universal VERSION=X.Y.Z
make one TARGET=linux-x64        VERSION=X.Y.Z

# Build from already-present artefacts in bin/ (temporarily hides the
# others under bin/.hidden/ and restores them after).
make linux-x64        VERSION=X.Y.Z
make linux-arm64      VERSION=X.Y.Z
make windows-x64      VERSION=X.Y.Z
make darwin-universal VERSION=X.Y.Z
```

Self-signed bundles (testing only - production envelope signing is not currently wired up; see "Trust model" below):

```bash
make signed VERSION=X.Y.Z
```

Build the OCI image (linux amd64 locally for testing; multi-arch push needs `docker login` with push rights on `docker.io/stackql/stackql-mcp`):

```bash
make oci VERSION=X.Y.Z          # local amd64 build from the release zips
python scripts/smoke-test.py --docker stackql/stackql-mcp:X.Y.Z
make oci-push VERSION=X.Y.Z     # multi-arch (amd64+arm64) build + push
```

Build the npm wrapper package (`@stackql/mcp-server`, an npx-able launcher that downloads the platform's published `.mcpb`, verifies the sha256 pins baked into the package, caches the binary under `~/.stackql/mcp-server-bin/`, and spawns it). ORDERING RULE: `make npm-manifest` fetches the canonical `.sha256` files from the published release - it must run AFTER the `.mcpb` assets for the version are published, same as `make server-json`. Publishing to npmjs is manual (2FA):

```bash
make npm-manifest VERSION=X.Y.Z   # render npm/platforms.json + stamp version
python scripts/smoke-test.py --cmd "node npm/bin/stackql-mcp.js"
make npm-pack VERSION=X.Y.Z       # build the tarball
cd npm && npm publish --access public
```

Build the PyPI wrapper (`stackql-mcp-server`, the same launcher in stdlib-only Python for uvx/pip; shares the npm wrapper's binary cache at `~/.stackql/mcp-server-bin/`). Same ordering rule; publish is manual (2FA):

```bash
make pypi-manifest VERSION=X.Y.Z
make pypi-build VERSION=X.Y.Z     # sdist + wheel (needs 'pip install build')
python -m twine upload pypi/dist/*
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stackql/stackql](https://github.com/stackql/stackql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
