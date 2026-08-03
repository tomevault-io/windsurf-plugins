---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build / Test / Format

The project is built via `make`, not directly with `swift build`. Two Swift packages live in this repo: the root package (Containerization libraries + `cctl` + macOS-only integration binary) and `vminitd/` (the Linux guest init system, compiled as a static musl binary inside the Linux dev container via the apple/`container` CLI — see `make vminitd`).

- `make all` — build everything (`containerization` + `vminitd` + `init.ext4` rootfs in `bin/`). Default `BUILD_CONFIGURATION=debug`; pass `release` (or use `make release`) for optimized builds.
- `make containerization` — build just the host-side Swift package (skips vminitd).
- `make vminitd` — build vminitd / vmexec only. On macOS this runs `swift build --swift-sdk …-swift-linux-musl` *inside the Linux dev container* via the `container` CLI (the cloud-hypervisor build model), producing static musl binaries at `vminitd/bin/`; no host Swiftly/SDK needed. `make linux-build LIBC=glibc` builds via a Linux dev container.
- `make test` — unit tests with code coverage. `make coverage` regenerates the coverage report.
- `make integration` — runs `bin/containerization-integration`. Requires an in-repo kernel under `bin/` (`bin/vmlinux-arm64` on arm64, `bin/vmlinuz-x86_64` or `bin/vmlinux-x86_64` on x86_64); if absent, run `make fetch-default-kernel` to download the Kata-provided kernel for the host arch.
- Single test: `swift test --filter ContainerizationOCITests.ReferenceTests/testParsing` (Swift Testing / XCTest filter syntax). Targets are listed in `Package.swift`.
- `make linux-test` — runs `swift test` inside the Linux dev container (requires the `container` CLI from apple/container).
- `make linux-build` — builds the host-side Swift package (incl. `cctl`, `Containerization`, and `CloudHypervisor`) inside the same Linux dev container. Use this to validate Linux portability of host-side code; the resulting `cctl` is what the cloud-hypervisor backend ships behind.
- `make linux-integration` — runs the cross-platform integration suite against a real cloud-hypervisor VM inside the dev container (nested virt via apple/container's `--virtualization`). Requires a KVM-capable kernel at `kernel/vmlinux-arm64` (or `kernel/vmlinuz-x86_64` on x86_64 hosts) — build via `make -C kernel`; the kata-fetched kernel doesn't include KVM. Also requires `make fetch-cloud-hypervisor` and `make linux-build` to have been run first. Linux runs only the cross-platform subset (`process true`/`false`/`echo hi`); the macOS suite is unchanged.
- `make fetch-cloud-hypervisor` — downloads the static `cloud-hypervisor` v52.0 (aarch64) binary into `bin/cloud-hypervisor` for the Linux integration tests.
- `make build-cloud-hypervisor` / `make build-virtiofsd` — build patched `cloud-hypervisor` / `virtiofsd` from sources you have cloned into `.local/cloud-hypervisor` and `.local/virtiofsd` respectively. There is no fetch target — clone the upstream repos at the revision you want pinned. `build-virtiofsd` applies `scripts/patches/virtiofsd-skip-cap-drop-with-sandbox-none.patch` and is idempotent. Both run inside the same Linux dev container as `linux-integration` so the resulting binaries are aarch64-linux-gnu.
- `make dist-x86_64` — assembles `bin/containerization-x86_64-<sha>.tar.gz` (cctl + cloud-hypervisor + virtiofsd + initfs.ext4 + kernel) for x86_64 Linux deployment, cross-compiled inside the aarch64 dev container via the Static Linux SDK (Swift) and `cargo zigbuild` (Rust). Prereqs: `.local/cloud-hypervisor` and `.local/virtiofsd` source checkouts (clone deliberately — no fetch target), and an x86_64 kernel built via `make -C kernel TARGET_ARCH=x86_64`. Per-stage rebuild env vars: `REBUILD_VMINITD=1`, `REBUILD_INITFS=1`, `REBUILD_CH=1`, `REBUILD_VIRTIOFSD=1`; cctl x86 always rebuilds. **Full pipeline, toolchain rationale, and troubleshooting in `docs/x86_64-build.md`.** The orchestrator is `scripts/build-dist-x86_64.sh`.
- `make fmt` — applies `.swift-format` and refreshes license headers via hawkeye.
- `make check` — formatting + license-header lint (this is what the pre-commit hook runs). Uses `.swift-format-nolint` for stricter linting.
- `make pre-commit` — installs `scripts/pre-commit.fmt` as a git pre-commit hook.
- `make protos` — regenerates `Sources/Containerization/SandboxContext/SandboxContext.{pb,grpc}.swift` from the `.proto`. Touch this whenever the proto changes; never hand-edit the generated files.
- `make init` / `make init-image` — `init` compiles the guest and builds `bin/initfs.ext4` (+ a rootfs tar) inside the dev container via `scripts/build-initfs.sh` (mkfs + loop mount, with a `mke2fs -d` fallback), then `init-image` creates the `vminit:latest` OCI image from the tar with the native `cctl` (`cctl rootfs create --rootfs <tar> --image vminit:latest`). CI splits these: a Linux container job builds the initfs artifact, the macOS job runs `init-image`. Building the guest on macOS requires the apple/`container` CLI — there is no host Swiftly / Static Linux SDK setup step anymore.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apple/containerization](https://github.com/apple/containerization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
