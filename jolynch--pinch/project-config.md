---
trigger: always_on
description: This document covers the protocol, server, client library, and testing conventions for the filexfer subsystem.
---

# Filexfer — Architecture Guide

This document covers the protocol, server, client library, and testing conventions for the filexfer subsystem.

## Directory layout

```
server/filexfer/                   # Public client library
  client.go                        # Client type, options, request/response types
  client_tcp.go                    # TCP transport implementation
  client_test.go                   # Integration tests against a real server
  docs/
    PROTOCOL.md                    # FTCP line protocol (AUTH, TXFER, SEND, ACK, CXSUM, STATUS, PROBE)
    MANIFEST.md                    # FM/1 manifest wire format
    FRAMING.md                     # FX/1 frame wire format
    CLI.md                         # CLI usage reference

server/internal/filexfer/
  ftcp/                            # Server-side FTCP command handlers
    server.go                      # Listener loop and connection dispatch
    verb.go                        # Verb enum
    request.go                     # Protocol line parser (ParseRequest)
    auth.go                        # AUTH handler + age encryption setup
    txfer.go                       # TXFER handler — manifest generation
    send.go                        # SEND handler — file streaming
    ack.go                         # ACK handler — progress acknowledgment
    cxsum.go                       # CXSUM handler — checksum streaming
    status.go                      # STATUS handler — transfer status/list
    probe.go                       # PROBE handler — latency/bandwidth probe
    sync.go                        # SYNC handler
    deps.go                        # Deps interface + runtimeDeps (thin wrapper over store)
    errors.go                      # protocolErr, writeOKLine, writeErrFrame helpers
  encoding/
    manifest.go                    # FM/1 marshal/parse (front-coded paths + mtimes)
    frame.go                       # FX/1 frame marshal/parse
    codec.go                       # Compression codec pools (zstd, lz4, identity)
    format.go                      # Byte/duration string parsing
  store/
    store.go                       # In-memory transfer state (global map, TTL eviction)
  policy/
    policy.go                      # Adaptive compression policy
  limit/
    limit.go                       # Rate-limited io.Writer
  progress.go                      # Background progress-file writer

server/internal/cmd/filexfercli/
  cli.go                           # CLI commands: copy, get, status, verify
  cli_test.go                      # End-to-end CLI tests with fake TCP servers
```

## Protocol

Full specification lives in `docs/PROTOCOL.md`. Key points:

- **Transport**: one TCP connection per command, server closes after completion.
- **Line format**: `VERB args...\r\n` → optional streaming payload → `OK [msg]\r\n` or `ERR <code> <msg>\r\n`.
- **AUTH**: optional first command; supports age encryption for both the command line and response stream.
- **Token encoding**: path/blob args are quoted (`"..."`) or length-prefixed (`<len>:<bytes>`).

### Command sequence for a typical download

```
PROBE cpu=<n> probe-bytes=<n> cts0=<ms>        → measure link
TXFER "<dir>" mode=fast link-mbps=<n> ...       → get FM/1 manifest stream
SEND <tid> fd=0 "<path>" [fd=1 "<path>" ...]    → receive FX/1 frame stream
ACK  <tid> fd=0 "<path>" ack-token=<tok> ...    → confirm windows received
STATUS <tid>                                    → poll progress JSON
STATUS                                          → list all active transfers (count + N JSON lines)
```

## Server internals (`internal/filexfer/`)

### `ftcp/` — command handlers

`Serve()` in `server.go` accepts connections and dispatches on the parsed verb. Each handler receives `(ctx, Request, io.Writer, Deps)`.

**`Deps` interface** (`deps.go`) abstracts all state mutations so handlers are testable without a real store. `runtimeDeps` is a thin pass-through to the `store` package. Tests construct a `mockDeps` implementing the same interface.

**`txfer.go`** handles both directory TXFER (walks the tree with `WalkDir`) and single-file TXFER (`encodeSingleFileManifest`). After writing the manifest it calls `deps.ClipTransfer` to seal the file count.

**`send.go`** is the hot path. It:
1. Opens files through `deps.GetFile` (validates transfer + path).
2. Streams windows as FX/1 frames with per-frame adaptive compression (`policy.CompressionPolicy`).
3. Uses zero-copy sendfile when available, falls back to buffered read.
4. Sets `SetTransferFileWindowHash` so subsequent ACKs can be verified.

**`status.go`** — two modes:
- `STATUS <txferid>`: returns `OK <json>` with a single `TransferStatus`.
- `STATUS` (no args): returns `OK <count>\r\n` followed by `<count>` JSON lines, one per active transfer.

Completed transfers remain in the store until TTL expiry (default 10 min), so `ListTransfers()` returns them.

### `encoding/` — wire formats

- **FM/1 manifest**: header line + one entry line per file. Paths and mtimes use front-coding (delta from previous entry) to compress the manifest without a compression codec.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolynch/pinch](https://github.com/jolynch/pinch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
