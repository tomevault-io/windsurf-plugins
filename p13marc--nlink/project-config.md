---
trigger: always_on
description: Guidance for Claude Code working on this repository. Conventions
---

# CLAUDE.md

Guidance for Claude Code working on this repository. Conventions
and invariants only — for API tutorials, follow the pointers to
source doc-strings, `docs/recipes/`, and `crates/nlink/examples/`.

## Project Overview

`nlink` is a Rust library for Linux network configuration via
netlink. The library is the deliverable; the `bins/{ip,tc,ss,nft,
wifi,devlink}` binaries exist as proof-of-concept demonstrations.
Every produced binary is `nlink-` prefixed (`nlink-ip`, `nlink-tc`,
…) so it never shadows the real system tool — the `[[bin]] name` in
each `bins/*/Cargo.toml` matches the package name. CLI integration
tests reference binaries via `env!("CARGO_BIN_EXE_nlink-<tool>")`.

Key design invariants:
- **Custom netlink** — no `rtnetlink` / `netlink-packet-*`
  dependency. We own the wire format end-to-end.
- **Async/tokio native** via `AsyncFd`.
- **Library-first**: binaries are thin wrappers over typed APIs.
- **Single publishable crate** (`nlink`) with feature flags. All
  binaries are `publish = false`.
- Rust edition 2024.

## Build & Test

```bash
cargo build                               # all crates + bins
cargo build -p nlink                      # library only
cargo test -p nlink --lib                 # library unit tests

# Lint and dep hygiene before any commit
cargo clippy --workspace --all-targets --all-features -- --deny warnings
cargo machete                             # no unused deps
```

## Integration tests

Live under `crates/nlink/tests/integration/` and require root +
network namespaces. Maintainer runs `cargo test` as a regular
user, so root-gated tests would **bit-rot silently** if they
weren't both (a) gated with `nlink::require_root!()` (so they
skip cleanly as non-root) and (b) run under the privileged-CI
gate that landed in 0.15.0 (Plan 140 — see
`.github/workflows/integration-tests.yml`; runs on every push/PR
to master under a container with `CAP_NET_ADMIN` + `CAP_SYS_ADMIN`
+ `seccomp=unconfined`). For local validation as a non-root user,
the `--apply` example runners stay the canonical channel (e.g.,
`examples/netfilter/conntrack.rs --apply`).

```bash
cargo test --test integration --features lab --no-run
sudo ./target/debug/deps/integration-* --test-threads=1
```

For new tests that need root, gate with `nlink::require_root!()`
(early-returns `Ok(())` when `euid != 0`). For tests that depend
on a specific kernel module, also gate with
`nlink::require_module!("nf_conntrack")` — `has_module()` checks
`/sys/module/<name>` so it works for both loadable and built-in
features. For new examples, prefer the `--apply` runner pattern
over assertions.

## Architecture

| Layer | Path | Role |
|---|---|---|
| Library | `crates/nlink/` | Single publishable crate with feature flags |
| Binaries | `bins/{ip,tc,ss,nft,wifi,devlink}` | CLI demos consuming the library |
| Recipes | `docs/recipes/` | End-to-end markdown walkthroughs |
| Examples | `crates/nlink/examples/` | Runnable demos per subsystem |
| Plans | GitHub issues + `CHANGELOG.md ## [Unreleased]` | In-flight work + roadmap (a per-cycle `plans/` dir is recreated when a cycle opens and deleted at cut) |

Inside `crates/nlink/src/`:
- `netlink/` — the core protocol stack (always built). Submodules
  per RTNetlink concept (`tc.rs`, `filter.rs`, `action.rs`,
  `link.rs`, `route.rs`, `rule.rs`, `nexthop.rs`, `mpls.rs`,
  `srv6.rs`, `bridge_vlan.rs`, `fdb.rs`, …) and per non-RTNetlink
  protocol (`netfilter.rs`, `xfrm.rs`, `connector.rs`,
  `uevent.rs`, `audit.rs`, `selinux.rs`, `fib_lookup.rs`,
  `nftables/`, `genl/`).
- `netlink/genl/{wireguard,macsec,mptcp,ethtool,nl80211,devlink}/`
  — Generic Netlink families.
- `netlink/config/` — declarative `NetworkConfig` (diff + apply +
  reconcile + opt-in purge).
- `netlink/{resync,resync_ext,reflector}.rs` — ENOBUFS-resync event
  streams, the `ResyncStreamExt` combinators, and the kube-rs-style
  `Store<K,V>` watch-cache (`ReflectExt::reflect`).
- `netlink/{ratelimit,impair,diagnostics}.rs` — high-level helpers.
  Both `RateLimiter` and `PerHostLimiter` expose the idempotent
  `reconcile{,_dry_run,_with_options}` trio alongside the
  destructive `apply` (0.24, #169).
- `sockdiag/` — high-level socket-diagnostics API (feature
  `sockdiag`; the wire layer is `netlink/sockdiag.rs`): typed
  queries + `FilterExpr` (ss grammar) with a full
  `INET_DIAG_REQ_BYTECODE` compiler (`bytecode.rs` —
  `compile_filter` lowers ports/addresses/or/not kernel-side and
  hoists `state` into `idiag_states`; client-side backstop applies
  automatically when inexact), socket→process/cgroup attribution
  (`procmap.rs` — `/proc` reads are fine here, the sysfs audit gate
  only covers `netlink/`), cookie-keyed TCP goodput deltas
  (`rate.rs` — TCP-only; UDP diag has no byte counters), and typed
  CC internals (`CcInfo`: BBR/DCTCP/vegas). All 0.24, #162/#163/#171.
- `lab/` — namespace + integration-test harness (feature `lab`).

Types are zero-copy via the `zerocopy` crate (`#[repr(C)]` +
`FromBytes` + `IntoBytes` + `Immutable` + `KnownLayout`). No
unsafe pointer casts in `types/`.

### Feature flags

| Feature | Purpose |
|---|---|
| `sockdiag` | Socket diagnostics (`NETLINK_SOCK_DIAG`) |
| `tuntap` | TUN/TAP device management |
| `output` | JSON/text output formatting helpers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p13marc/nlink](https://github.com/p13marc/nlink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
