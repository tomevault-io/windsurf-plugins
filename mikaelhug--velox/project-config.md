---
trigger: always_on
description: Velox is a lightweight, open-source Docker Desktop / OrbStack alternative for macOS.
---

# Velox — Project Conventions

Velox is a lightweight, open-source Docker Desktop / OrbStack alternative for macOS.

## North Star — the main objective

Be **as lean, efficient, fast, and small-footprint as possible**, and on every
metric that matters — startup time, RAM, CPU, disk footprint, network throughput,
filesystem I/O — **beat Docker Desktop and OrbStack where it's possible, and be at
least on par otherwise.** Every decision in this repo serves that goal. The five
pillars that deliver it:

1. **Apple's kernel networking (VZNAT).** The container datapath is Apple's
   in-kernel NAT — the fastest networking path on `Virtualization.framework`
   (measured ~14 Gbit/s down / ~80 Gbit/s up, beating Docker Desktop). **No
   userspace netstack:** a host-side `PortForwarder` maps published ports to the
   host address in `VeloxConfig.publishHostIP` — **`0.0.0.0` by default, matching
   Docker's own default**, so a published port is reachable from other machines
   (`"127.0.0.1"` restores host-only; see `PublishBind`). The guest always publishes
   guest-locally on `0.0.0.0`, so a macOS host address is never sent to the guest —
   per-container `-p <hostIP>:…` is unsupported (the guest dockerd can't bind a Mac
   address) and the global setting is the knob. dockerd `--host-gateway-ip` wires
   `host.docker.internal`.
   (Truly *beating* VZNAT on raw throughput would need a custom hypervisor —
   OrbStack's moat, out of scope; VZNAT keeps us on-par-or-better within VZ.)
   **Direct named access** (`<name>.velox.local` → the container's *real* IP, any
   protocol — OrbStack-style, no `-p`) is **pure DNS + routing, never a proxy or
   netstack**: a loopback DNS responder (`NameDNSResponder`, reached via
   `/etc/resolver/velox.local`) answers names from the event-driven container map
   (`NameRegistry`, filled by `DockerEventsWatcher`); a porthelper-installed host route
   (`NamedAccessRouter`) carries the Mac to the container subnet over VZNAT; and `vinit`
   adds one `nft` allow for the gateway IP in **both** dockerd-29 host→container drop
   chains (`filter-FORWARD` *and* `raw-PREROUTING`), re-asserted event-driven: the
   dockerd supervisor signals on respawn, and a guest-local docker `/events` network
   informer signals on endpoint changes (dockerd rewrites `raw-PREROUTING`'s
   per-container drops on every `docker run`, flushing the allow — measured).
   No entitlement, no ongoing root (one-time grant only — folded into the porthelper).
   Don't replace this with a reverse proxy or re-add `trusted_host_interfaces` (it doesn't
   apply to `docker0`).
2. **A 100% Swift host.** The whole supervisor — VM lifecycle, Docker-API VSOCK
   proxy, port forwarding, clock sync, the SwiftUI GUI — is pure Swift on
   `Virtualization.framework`. **No Go, no host-side Rust, no helper daemons** — with
   **one sanctioned exception**: `velox-porthelper`, a tiny root LaunchDaemon (still
   Swift, `Sources/velox-porthelper/`) that exists *only* for the things macOS gives
   an unprivileged process no other way to do, all **control-plane**: (a) bind a
   port `<1024` — loopback, or all interfaces when the request carries the explicit `any`
   argument, so a published `-p 22:22`/`-p 80:80` is reachable off-box like Docker's
   default — and pass the listening socket fd back over a unix socket, (b) add/remove
   a host route to a container subnet (for direct **named-container access** — `<name>.velox.local`
   → the container's real IP), and (c) restore `net.inet.ip.forwarding=1` — **restore-only,
   it can never switch forwarding off**. (c) exists because some VPN clients (measured:
   the OpenVPN-based AWS VPN Client) zero that sysctl on connect, killing the entire
   vmnet NAT datapath for every container while the routing table stays clean; the
   `ForwardingGuard` (event-driven: NWPathMonitor push → unprivileged sysctl re-read →
   helper restore) heals it instantly, no engine restart. A VPN merely holding the
   default route — e.g. full-tunnel WireGuard — does NOT break vmnet and needs no
   handling (measured); don't add default-route-based VPN gates. It never touches
   connection data, so **root stays out of the datapath**. It is installed on first use with a single admin prompt (no Developer ID ⇒ no
   `SMAppService`; a manual `osascript`-authorized LaunchDaemon install that *also* writes
   `/etc/resolver/velox.local`), and the host degrades gracefully (skip privileged ports / no
   named access) if the user declines. Keep it minimal: this is the *only* privileged component,
   nothing else may grow a daemon, and any new helper command must stay control-plane (route/port
   setup, never connection bytes). See `Sources/VeloxCore/Proxy/PortHelper.swift`.
3. **Rust only for the tiny guest `vinit`.** One static-musl Rust binary is the
   guest's PID 1 and entire userland orchestration. Lean and fast by design.
4. **A custom kernel, as lean as possible.** Built from kernel.org source —
   `tinyconfig` + a curated fragment, only what the VM needs, nothing else. Where a
   guest component must be faster or smaller, add a focused high-performance Rust
   piece (like `vinit`) rather than pull in heavyweight userland.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikaelhug/Velox](https://github.com/mikaelhug/Velox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
