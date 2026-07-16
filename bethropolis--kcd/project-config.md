---
trigger: always_on
description: **`kcd`** is a headless Go daemon implementing KDE Connect protocol v8. Single binary — daemon and CLI client share the same entry point via `urfave/cli/v2` subcommands. The daemon exposes a Unix socket for IPC; the CLI connects to it as a client. No GUI. D-Bus is used only by the MPRIS and SystemVolume plugins, and those fail gracefully if D-Bus is unavailable.
---

# kcd — Agent Instructions

## Project Identity

**`kcd`** is a headless Go daemon implementing KDE Connect protocol v8. Single binary — daemon and CLI client share the same entry point via `urfave/cli/v2` subcommands. The daemon exposes a Unix socket for IPC; the CLI connects to it as a client. No GUI. D-Bus is used only by the MPRIS and SystemVolume plugins, and those fail gracefully if D-Bus is unavailable.

- **Language:** Go 1.25, `CGO_ENABLED=0` everywhere
- **Module:** `github.com/bethropolis/kcd`
- **Static binary** — `ldd kcd` must print "not a dynamic executable"
- **Core constraint:** Memory efficiency. Never buffer what can be streamed. Reuse allocations via the packet pool and a single `bufio.Reader` per connection.

---

## Absolute Rules

Never violate these. If a change would break one, stop and reconsider the approach entirely.

1. **CGO_ENABLED=0 always.** No cgo, no dynamic linking. Build with `CGO_ENABLED=0 go build`.
2. **Never buffer file payloads.** Receive with `io.Copy(dst, io.LimitReader(conn, payloadSize))`. Buffering the entire file causes OOM.
3. **Never write to a `net.Conn` directly.** Always send via `device.Send(packet)`. The device's internal send channel is the only goroutine-safe write path.
4. **One `bufio.Reader` per connection, allocated once at accept/dial time.** Never allocate a new one inside the packet read loop.
5. **Use the packet pool.** Acquire with `protocol.AcquirePacket()`, release with `protocol.ReleasePacket(pkt)` after the plugin finishes — not before.
6. **Protocol version = 8.** Hardcoded as `protocol.ProtocolVersion`. Never send a different value.
7. **deviceId is permanent.** Generated once via `config.EnsureDeviceID`, stored in `kcd.toml`. Never regenerate. It is the stable identity used for cert fingerprint pairing.
8. **Self-signed TLS, `InsecureSkipVerify: true`.** Authentication happens via the SHA-256 fingerprint stored in `devices.json` after pairing — not via CA chain.
9. **`Plugin.Handle()` must return immediately.** Any D-Bus call, subprocess (`exec.Command`), or disk I/O must be spawned in a goroutine *inside* the plugin. Blocking `Handle()` stalls the entire TCP read loop for that device.
10. **No deprecated packages.** No `ioutil` (use `os`/`io`). No `log` (use `go.uber.org/zap`). No `cobra` (use `urfave/cli/v2`).
11. **Keep `Body` as `json.RawMessage` in the router.** Plugins unmarshal their own body types. The packet router never touches body content.
12. **One goroutine per connection.** Dispatch is sequential per device — one packet handled at a time. This is intentional; it removes the need for per-plugin locks.
13. **Cap incoming payload size with `io.LimitReader`.** Never trust the `payloadSize` field from the remote device without a cap.
14. **TLS handshake order:** exchange plaintext identity → upgrade to TLS (initiator = TLS server, acceptor = TLS client) → send full identity over TLS. The inverted TLS roles are a KDE Connect protocol requirement.

---

## Architecture Invariants

These structural constraints must hold at all times:

| Invariant | Why |
|---|---|
| `internal/protocol/` has **zero external imports** (stdlib only) | Protocol types are used everywhere; external deps would create cycles |
| `internal/config/` only imports `github.com/BurntSushi/toml` | Config must stay lean and cycle-free |
| `internal/plugin/plugin.go` only imports `internal/protocol` and `internal/device` | Plugins never import each other |
| Plugins are registered in `daemon.go`, never in their own `init()` | Explicit, ordered, conditional on config |
| `pkg/client/` only imports `internal/ipc` from the `internal/` tree | Public client API must not depend on internals beyond the IPC protocol |
| All binaries built with `CGO_ENABLED=0` | Required for static distribution |

---

## Key File Locations

| Purpose | Path |
|---|---|
| Config file | `~/.config/kcd/kcd.toml` (`$XDG_CONFIG_HOME/kcd/kcd.toml`) |
| Device state (persisted pairs) | `~/.local/state/kcd/devices.json` (`$XDG_STATE_HOME/kcd/devices.json`) |
| TLS cert / key | `~/.config/kcd/cert.pem`, `~/.config/kcd/key.pem` |
| IPC Unix socket | `/run/user/<uid>/kcd/kcd.sock` (`$XDG_RUNTIME_DIR/kcd/kcd.sock`) |
| Downloaded files | `~/Downloads/kcd/` (overridable via `download_dir` in config) |
| systemd user unit | `~/.config/systemd/user/kcd.service` |

> **Note:** The socket lives in a `kcd/` subdirectory of the runtime dir, not directly in `/run/user/<uid>/`.

---

## Daemon Startup Order

`daemon.Run()` wires everything in this exact sequence. Preserve the order when modifying startup:

1. Build `zap.Logger` from config log level
2. Load or generate TLS certificate (`cert.LoadOrGenerate`)
3. Create event bus (`events.NewBus`)
4. Create device registry (`device.NewRegistry`) and load persisted state from `devices.json`
5. Create plugin registry (`plugin.NewRegistry`)
6. Register all enabled plugins — **pair plugin always first**, then the rest conditioned on `cfg.Plugins.*`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bethropolis/kcd](https://github.com/bethropolis/kcd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
