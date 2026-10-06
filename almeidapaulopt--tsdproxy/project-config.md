---
trigger: always_on
description: Proxmox VE target provider: watches a Proxmox cluster (QEMU VMs + LXC containers) for guests whose notes/description field carries a `tsdproxy:` YAML block, resolves target URLs from guest-agent/LXC interface addresses, generates per-proxy config. No event API exists, so lifecycle is discovered by polling `/cluster/resources` and diffing snapshots.
---

# internal/targetproviders/proxmox

Proxmox VE target provider: watches a Proxmox cluster (QEMU VMs + LXC containers) for guests whose notes/description field carries a `tsdproxy:` YAML block, resolves target URLs from guest-agent/LXC interface addresses, generates per-proxy config. No event API exists, so lifecycle is discovered by polling `/cluster/resources` and diffing snapshots.

## STRUCTURE

| File | Role |
|------|------|
| `proxmox.go` | `Client` (TargetProvider impl). `WatchEvents` polls every `pollIntervalSeconds`; first poll doubles as initial scan, its failure goes to errChan (ProxyManager backoff reconnect), later transient failures keep the watcher + snapshot alive. `pollOnce` → `handleRunningGuest` (diff state machine: Start/Stop/Restart) + `handleStoppedGuest` + `emitDeleted`. `AddTarget`/`ReResolve` via `buildProxyConfig`. |
| `rest.go` | `APIClient` interface + hand-rolled `restClient` (no official PVE Go SDK). API-token auth (`Authorization: PVEAPIToken=<id>=<secret>`), PVE JSON envelope (`{"data":...}`) decoding, optional CA cert (`tlsCaCertFile`) / insecure TLS, optional `Doer` injection. `GetVersion` doubles as connect-time credential check. Node filter applied client-side in `ListGuests`. |
| `notes.go` | Notes/description → `notesConfig` parsing (ALWAYS non-nil; `Enable=false` means no tsdproxy data). Accepts BOTH styles: Docker/Incus flat grammar (`tsdproxy.port.443: 8443/https:8080/http`) and list-provider schema (`ports: {"443/https": {targets: [...]}}` + proxyProvider/dashboard/tailscale/... keys). List-style scalars are translated into flat settings keys, filling only absent keys (flat wins). Whole-string base64 notes tolerated (raw YAML tried first). |
| `guest.go` | `guest` struct. `setInterfaces` (eth0-first, IPv4-first, drops loopback/link-local/multicast), `getPorts` merges flat-grammar ports (`model.Ports` + `generateTargetFromFirstTarget`) with list-style entries (`processListPort`: explicit targets verbatim, else target generated from guest address — long-label placeholder carries scheme/port, short labels use proxy protocol/port; funnel/tlsValidate gated by operator settings), `newProxyConfig` builds `model.Config`. |
| `consts.go` | Flat settings key constants (mirror Docker label names), guest type/status strings, timing constants. |
| `errors.go` | Sentinel errors: `ErrGuestNotEnabled`, `ErrGuestNotRunning`, `ErrGuestNotFound`, `ErrNoAddressFound`, `ErrInvalidAPIToken`, `ErrInvalidGuestID`. |
| `*_test.go` | `mockAPIClient` (provider tests), `httptest` server (REST tests, `newPVE` factory serves /version automatically), notes parsing table tests, goleak `TestMain`. |

## CONFIG KEY SCHEMA

Notes are the only free-form field on a Proxmox guest (QEMU `notes`, LXC `description`). A guest is enabled when its notes contain tsdproxy data, unless `tsdproxy.enable: false` (presence = opt-in, list-provider semantics). Two accepted shapes:

```yaml
# Docker/Incus style — port value is the shared label grammar
tsdproxy:
  port:
    443: 8443/https:8080/http     # proxy 8443/https -> guest:8080/http
  dash.icon: jellyfin

# list provider style — same keys as a list file entry
tsdproxy:
  hostname: myapp                # -> tsdproxy.name
  ports:
    "443/https":
      targets:
        - "http://10.0.0.5:8080" # explicit; omit to generate from guest address
      tlsValidate: false          # gated by allowTlsValidateDisable
      tailscale:
        funnel: true              # gated by allowGuestFunnel
```

Rest of the notes must be valid YAML (comment out prose with `#`). Flat keys are canonical; when both styles set the same field, the flat key wins. Typed lookups use the shared `internal/targetproviders/settings` package; flat per-guest overrides (health_*, ratelimit.*, authkey/authkeyfile) work exactly like Docker/Incus.

## KEY DESIGN DECISIONS

- **Polling, not events**: `/cluster/resources?type=vm` every N seconds; per-state diff emits Start (new/restarted/re-enabled), Stop (stopped/disabled/deleted — only when tracked, i.e. a proxy exists), Restart (notes DeepEqual change while tracked; untracked change retries Start).
- **Snapshot survives transient errors**: config-fetch or notes-parse failures keep the previous snapshot (`continue`), never looking like a change or deletion. Only the FIRST poll failure surfaces via errChan → ProxyManager backoff.
- **Tracked map = proxy existence**: `guests` map populated by `AddTarget` gates all Stop emissions — ProxyManager never receives stops for nonexistent proxies.
- **Guest ID is `<type>/<vmid>`** (e.g. `qemu/100`): stable across node migration, unique cluster-wide, matches the resources endpoint `id` field.
- **Target resolution**: provider `targetHostname` override → else first usable guest address (QEMU needs guest agent `network-get-interfaces`; LXC uses `/interfaces`; agent missing/failed → no addresses, fallback decides). Same preference order as Incus: eth0 first, IPv4 before IPv6.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [almeidapaulopt/tsdproxy](https://github.com/almeidapaulopt/tsdproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
