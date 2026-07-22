---
trigger: always_on
description: This is the tool-neutral project instruction entry point for coding agents.
---

# AGENTS.md

This is the tool-neutral project instruction entry point for coding agents.
Tool-specific compatibility files should point here rather than duplicate
project guidance.

## Project Overview

AKernel provides cluster-backed remote sandbox environments for agents and
developer workflows. The current public user-facing surface is the Python
`akernel-sdk`, including the `akernel_sdk.Sandbox` API and the `ak` CLI.

Use AKernel when a task needs an isolated remote environment with command
execution, file operations, interactive PTYs, port forwarding, or reverse
tunnels. The project overview and deployment quick start are in
[`README.md`](./README.md), detailed SDK documentation is in
[`sdk/python/README.md`](./sdk/python/README.md), and runnable examples are in
[`sdk/python/examples/`](./sdk/python/examples/).

## Source Layout

- `sdk/python/` - AKernel Python SDK and CLI.
- `sdk/python/akernel_sdk/` - SDK implementation for `Sandbox`, commands,
  filesystem, PTY support, instance plumbing, and CLI helpers.
- `sdk/python/examples/` - maintained AKernel SDK examples.
- `sdk/python/tests/` - maintained AKernel SDK tests.
- `builder/` - Dockerfiles, service configs, runtime rootfs build, and image
  entrypoint scripts for the public all-in-one image.
- `deploy/` - Helm charts, standalone scripts, Terraform modules, and
  deployment helper scripts.
- `assets/` - static images used by the root README.

The open-source AKernel repository contains the SDK, deployment configuration,
build tooling, and examples. Node runtime components such as `sandboxd` and
`distill-fs` are maintained in their own upstream repositories. The all-in-one
Docker build copies and compiles their pinned Git submodules in dedicated
builder stages. It also downloads `runsc` from the official gVisor release
bucket and verifies its published SHA-512 checksum.

## Common Commands

All commands should be run from the repository root.

```bash
make help
make check VENDOR=aliyun
make config VENDOR=aliyun
make build
make push
make plan
make deploy
make token TTL=24h
make print-env
make e2e
```

This is a command reference, not an unconditional sequence. Skip `make build`
and `make push` when the deployment profile selects an existing image. The
image repository and tag used by `make build` and `make push` come from the
profile created by `make config`; set both during configuration rather than
overriding only the build command.

`make plan` is read-only with respect to cloud resources. `make deploy` applies
Terraform and Helm changes, while `make destroy` destroys cloud resources.
Agents must show the plan and obtain explicit user approval before running
either mutating command. Do not use `AUTO_APPROVE=1` without that approval.

## Local Deployment State

Interactive deployment helpers write local state under `.akernel/default/` by
default. Pass `ENV=<name>` when you need multiple independent deployment
profiles. These directories are intentionally ignored by Git. They may contain:

- generated Terraform variables
- kubeconfig files and paths
- IAM signing seeds
- generated JWT tokens
- SDK environment exports

Never commit `.akernel/`, Terraform state, kubeconfigs, tokens, signing seeds,
cloud credentials, private registry URLs, or local debug artifacts.

## Build

AKernel uses Docker for building. The public distribution ships one all-in-one
image that can run as master, frontend, node, or standalone depending on the
deployment entrypoint and environment.

```bash
make build
```

For a build that will be pushed and deployed, set `IMAGE_REPOSITORY` and
`IMAGE_TAG` when creating the deployment profile. A one-off `IMAGE_TAG`
override on `make build` does not update the profile consumed by `make push`.
The build creates only the selected image reference; it does not add a second
`akernel-all-in-one` alias. `make push` pushes that selected reference directly.

The build helper performs two Docker builds. `builder/runtime.Dockerfile`
assembles the Python runtimes and creates `yr-runtime-rootfs.img`.
`builder/node.Dockerfile` then compiles the node components and produces the
AKernel all-in-one image using that runtime image.

Initialize submodules with `git submodule update --init --recursive` before
building. The all-in-one image builds `sandboxd` and `sbox` from
`src/sandboxd`, builds `distill_fs` from `src/distill-fs`, and
downloads the official gVisor `runsc` release pinned by `GVISOR_RELEASE` in
the image build.

The submodule gitlinks are the single source of truth for the sandboxd and
distill-fs revisions included in a clean release. `make build` always compiles
the local submodule worktrees, so developers may check out a different commit
or edit either directory and rebuild without pushing first. Each component
maintains and embeds its own semantic version: sandboxd uses
`version/VERSION`, while distill-fs uses the package version in `Cargo.toml`.
AKernel does not inject parent-repository version metadata into component
compilation.

Inspect the selected local versions without building an image:

```bash
make versions
```

The final image uses standard OCI labels for the AKernel version and revision.
Component semantic versions are reported by their binaries, and their exact

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inclusionAI/AKernel](https://github.com/inclusionAI/AKernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
