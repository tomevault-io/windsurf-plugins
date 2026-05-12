---
trigger: always_on
description: Devcontainer Bridge is a dual-daemon tool that transparently forwards TCP ports, Unix sockets, and opens browser URLs between Linux devcontainers and the macOS/Linux host. It solves the gap left by the devcontainer CLI (vs VS Code) where container-bound ports are unreachable from the host, host Unix sockets are inaccessible from containers, and browser-opening requests fail in headless containers.
---

# Devcontainer Bridge (`dbr`)

## Project overview

Devcontainer Bridge is a dual-daemon tool that transparently forwards TCP ports, Unix sockets, and opens browser URLs between Linux devcontainers and the macOS/Linux host. It solves the gap left by the devcontainer CLI (vs VS Code) where container-bound ports are unreachable from the host, host Unix sockets are inaccessible from containers, and browser-opening requests fail in headless containers.

**Primary use case:** Tools like Atlassian MCP's OAuth flow that bind a random port, open the host browser, and expect the callback on `localhost:PORT` — all of which fail without this bridge.

### How it works

Two daemons cooperate via a pair of TCP channels on loopback:

- **Host daemon** (`dbr host-daemon`) — long-lived process on the host. Binds control and data ports, accepts registrations from containers, binds per-port listeners for forwarded ports, scans for Unix sockets to forward into containers, and opens URLs in the host browser.
- **Container daemon** (`dbr container-daemon`) — runs inside each devcontainer. Polls `/proc/net/tcp` for new listeners, sends Forward/Unforward messages, creates mirror Unix sockets for host-side sockets, and handles reverse data connections for proxying.

All TCP connections are initiated **container to host** (reverse connection model). This is required because macOS Docker Desktop cannot route to container IPs — containers run in a Linux VM with no direct host-to-container networking.

---

## Architecture

### Channels

| Channel | Default port | Default bind | Purpose |
|---------|-------------|-------------|---------|
| Control | `19285` | auto-detected | JSON-line protocol for registration, Forward/Unforward, SocketForward/SocketUnforward/SocketConnectRequest, OpenUrl, Ping/Pong, ListRequest/ListResponse |
| Data | `19286` | auto-detected | Reverse data connections from containers for TCP and Unix socket proxying |

Control and data ports use auto-detected bind addresses: `0.0.0.0` if Docker is running (so containers can reach the host via Docker Desktop's gateway IP), `127.0.0.1` otherwise. Configurable with `--bind-addr` or `--no-docker-detect`.

### Data flow for a proxied connection

```
1. Client connects to host:PORT (forwarded port listener)
2. Host daemon sends ConnectRequest{port, conn_id} via control channel
3. Container daemon connects to localhost:PORT inside container
4. Container daemon opens NEW TCP connection to host data port (19286)
5. Container daemon sends ConnectReady{conn_id} on data connection
6. Host daemon matches conn_id, bridges client <-> data connection
7. Bidirectional copy via tokio::io::copy_bidirectional
8. When either side closes, both connections tear down
```

### Data flow for a socket-forwarded connection

```
1. Host daemon scanner discovers Unix socket matching watch_paths glob
2. Host sends SocketForward{socket_id, host_path, container_path} to containers
3. Container creates mirror UnixListener at container_path (mode 0600)
4. Client in container connects to mirror socket
5. Container sends SocketConnectRequest{socket_id, conn_id} on control channel
6. Host connects to original Unix socket at host_path
7. Container opens reverse TCP to host data port, sends ConnectReady{conn_id}
8. Host bridges Unix socket <-> TCP data stream bidirectionally
```

### Key design decisions

- **Reverse data connections** — All connections flow container-to-host because macOS Docker Desktop cannot route to container IPs (containers live in a Linux VM). Same pattern as SSH `-R` reverse port forwarding.
- **TCP control channel (not Unix socket)** — No Docker volume mount or `devcontainer.json` modification required. Works through `host.docker.internal` DNS.
- **Two ports (control + data)** — Separates the framed JSON-line protocol from raw TCP byte streams cleanly. Control messages stay parseable; data connections switch to raw bytes after a single handshake line.
- **Two-tier binding** — Control and data ports use auto-detected bind addresses (`0.0.0.0` if Docker is running, `127.0.0.1` otherwise), configurable via `--bind-addr` or `--no-docker-detect`. Forwarded per-port listeners always bind to loopback (`[::1]`/`127.0.0.1`) only. The control protocol is hardened against untrusted clients (message limits, field validation, resource caps).
- **Single binary** — `dbr host-daemon` and `dbr container-daemon` are subcommands of the same binary. `dbr-open` is a hardlink for `BROWSER` env var integration. The container daemon is auto-started via the devcontainer feature's entrypoint script.

---

## Module map

```
src/
  main.rs               CLI entrypoint, clap dispatch, tracing init, dbr-open hardlink detection
  dbr/entrypoint.sh     Devcontainer feature entrypoint — starts container daemon on container boot

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradleybeddoes/devcontainer-bridge](https://github.com/bradleybeddoes/devcontainer-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
