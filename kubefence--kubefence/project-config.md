---
trigger: always_on
description: Authoritative technical reference for AI agents working on this codebase.
---

# Agent Reference — nono-nri

Authoritative technical reference for AI agents working on this codebase.
Read this before reading any source file. See CLAUDE.md for commit conventions.

---

## What this project is

`nono-nri` is a Kubernetes NRI (Node Resource Interface) plugin that intercepts
container creation and wraps the container process with the `nono` Landlock
sandbox binary. Opt-in is via RuntimeClass: only pods whose RuntimeClass handler
matches the configured list are wrapped; all others are skipped with zero overhead.

The `nono` binary (built from source by `scripts/build-nono.sh`, glibc by default)
is copied from the container image to the host by a DaemonSet init container,
then bind-mounted read-only into each sandboxed container at `/nono/nono`.

---

## File map

```
cmd/nono-nri/main.go              entrypoint: flags (-log-level, -json, -config), kernel check, config, stub.Run
internal/nri/plugin.go            Plugin struct; CreateContainer, StopContainer, RemoveContainer
internal/nri/filter.go            ShouldSandbox, SkipReason — pure, no I/O
internal/nri/profile.go           ResolveProfile — annotation → validated profile name
internal/nri/adjustments.go       BuildAdjustment — args wrapping + bind mount + seccomp policy
internal/nri/seccomp.go           BuildSeccompPolicy — RuntimeDefault and restricted allowlists
internal/nri/state.go             WriteMetadata, RemoveMetadata — /var/run/nono-nri/…
internal/nri/config.go            Config struct, LoadConfig (TOML)
internal/nri/kernel.go            CheckKernel — Linux 5.13+ required for Landlock
internal/nri/export_test.go       test-only helpers (SetStateBaseDir, SetKernelVersionFunc, …)
internal/log/log.go               slog JSON (prod) / text (dev) logger factory; accepts slog.Level

deploy/kind/deploy.sh             Kind cluster + plugin deploy automation
deploy/kind/e2e.sh                E2E test suite (17 checks)
deploy/kind/cluster-containerd.yaml   Kind cluster config with NRI enabled
deploy/daemonset.yaml             DaemonSet manifest (init + main containers)
deploy/runtimeclass-kata.yaml     kata-nono-sandbox RuntimeClass (handler: kata-qemu)
deploy/10-nono-nri.toml.example   Annotated TOML config reference
Dockerfile                        Multi-stage: golang:1.24-alpine builder → alpine:3.20
.github/workflows/release.yaml    CI: builds static nono from source, builds + pushes to ghcr.io
scripts/build-nono.sh             builds nono from always-further/nono source; glibc by default
                                  (BUILD_TARGET=musl for fully static); patches keyring to drop
                                  libdbus (sync-secret-service disabled)
```

---

## Data flow: CreateContainer

```
NRI event
  └─ Plugin.CreateContainer(ctx, pod, ctr)
       ├─ ShouldSandbox(pod, cfg)          check pod.RuntimeHandler ∈ cfg.RuntimeClasses
       │    false → Log "skip" + return nil, nil, nil   (no adjustment, no state)
       │    true  ↓
       ├─ ResolveProfile(pod, cfg)         pod annotation "nono.sh/profile" or cfg.DefaultProfile
       ├─ BuildSeccompPolicy(cfg.SeccompProfile) → *LinuxSeccomp (nil when disabled)
       ├─ BuildAdjustment(ctr, profile, cfg.NonoBinPath, isVMRootfs, seccomp)
       │    SetArgs: [/nono/nono, wrap, --profile, <profile>, --, <original args...>]
       │    AddMount: host dir of NonoBinPath → /nono  (bind, ro, rprivate)
       │    SetLinuxSeccompPolicy: applied when seccomp != nil
       │    NOTE: isVMRootfs is currently a no-op — the bind-mount is always
       │    added regardless of vm_rootfs_classes membership (reserved for
       │    future use; for Kata the mount works via virtiofs either way)
       ├─ WriteMetadata(pod.UID, ctr.ID, …)
       │    creates /var/run/nono-nri/<podUID>/<ctrID>/metadata.json
       └─ Log "injected" + return adjustment
```

## Data flow: state cleanup

```
Container stops (kubelet → containerd CRI StopContainer)
  └─ Plugin.StopContainer(ctx, pod, ctr)       ← RELIABLE (direct gRPC RPC)
       └─ RemoveMetadata(pod.UID, ctr.ID)
            os.RemoveAll /var/run/nono-nri/<podUID>/<ctrID>/
            os.Remove    /var/run/nono-nri/<podUID>/        (if now empty)

Container removed (containerd NRI StateChange REMOVE_CONTAINER)
  └─ Plugin.RemoveContainer(ctx, pod, ctr)     ← belt-and-suspenders only
       └─ RemoveMetadata(pod.UID, ctr.ID)
```

---

## Critical NRI event delivery constraint

In containerd 2.x, **StateChange notifications are not delivered to external
(socket-connected) NRI plugins**. This affects:

| Event | Delivery mechanism | External plugin receives it? |
|---|---|---|
| `CreateContainer` | direct gRPC RPC | yes |
| `StopContainer` | direct gRPC RPC | yes |
| `RemoveContainer` | StateChange notification | **NO** |
| `RemovePodSandbox` | StateChange notification | **NO** |
| `StopPodSandbox` | StateChange notification | **NO** |

**Consequence:** `StopContainer` is the primary cleanup hook. `RemoveContainer`
is kept as a fallback for runtimes that do deliver StateChange events, but must
never be the only cleanup path.

When adding new lifecycle hooks, check whether the event is a direct RPC or a
StateChange. Only direct RPCs are reliable for external plugins.

---

## Key invariants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubefence/kubefence](https://github.com/kubefence/kubefence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
