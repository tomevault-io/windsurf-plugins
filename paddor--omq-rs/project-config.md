---
trigger: always_on
description: Six-crate Cargo workspace; `bindings/` is excluded and built
---

# CLAUDE.md

## Workspace layout

Six-crate Cargo workspace; `bindings/` is excluded and built
out-of-tree (maturin etc.).

- **`omq-proto`** -- sans-I/O ZMTP 3.x core. Codec (`Connection`),
  message/payload types, greeting + frame state machines, mechanism
  handshakes (NULL / PLAIN / CURVE / BLAKE3ZMQ), compression transforms
  (lz4), endpoint parsing, options, subscription matcher.
  No async, no I/O. Mirrors `rustls::ConnectionCommon` / `quinn-proto`.
- **`omq-tokio`** -- multi-thread tokio backend. **Default backend.**
  Works on Linux and macOS (and likely other mio targets).
- **`omq-compio`** -- single-threaded compio backend (io_uring on
  Linux, IOCP on Windows). Not available on macOS.
- **`blume`** -- batching MPSC channel for `omq-compio` inbound delivery.
- **`yring`** -- bounded SPSC ring buffer for inproc transport.
- **`omq-libzmq`** -- libzmq-compatible C interface (`libomq_zmq.so` /
  `.a`). Drop-in replacement: ships `zmq.h`, implements the `zmq_*`
  API. Backed by `omq-tokio`.
- **`bindings/pyomq`** -- PyO3 wrapper over `omq-tokio`. Own `Cargo.lock`.
  Build: `cd bindings/pyomq && maturin develop --release`.

Both backends re-export `omq-proto`'s public API and share an identical
public `Socket` API. Verified by `tests/coverage_matrix.rs` (both) and
`tests/interop/` (cross-runtime TCP and WS tests).

## Build / test / bench

See [`DEVELOPMENT.md`](DEVELOPMENT.md) for the full command reference
(unit tests, feature-gated tests, fuzz, soak, stress tests, benchmarks).

Quick reference:

```sh
cargo build --workspace
cargo fmt                                # pre-commit hook checks this
cargo clippy --workspace --all-targets   # pre-commit hook checks this
./scripts/test-all.sh                    # full sweep, both backends
```

**HARD RULE:** Clippy must pass under all three configurations before
pushing to GitHub. Never push code that produces clippy warnings or
errors. Run all three before every `git push`:

```sh
cargo clippy --workspace --all-targets                # default features
cargo clippy --workspace --all-targets --all-features # feature-gated paths
(cd bindings/pyomq && cargo clippy --all-targets)     # separate workspace
```

`#[allow]` vs `#[expect]`: use `#[expect]` by default. Use `#[allow]`
only when the lint fires in some feature combinations but not others
(the expectation would be unfulfilled when the lint is silent).

Lints: `missing_debug_implementations` = **deny**,
`unsafe_op_in_unsafe_fn` = **deny**, clippy `pedantic` = **warn**.

## Benchmarks, charts, releasing

See [`DEVELOPMENT.md`](DEVELOPMENT.md) for comparison benchmark infra,
chart generation, and release process.

**interop dep constraint:** `tests/interop/Cargo.toml`'s compio
dep must use the same version as `omq-compio`'s dep. Different
versions link two `compio-runtime` instances -> TLS mismatch panic.

## Cargo features

| feature | adds | deps |
|---------|------|------|
| `plain` | PLAIN auth (RFC 24) | - |
| `curve` | CURVE handshake (RFC 26) | `crypto_box`, `crypto_secretbox` |
| `blake3zmq` | BLAKE3 + ChaCha20 mechanism | `blake3`, `chacha20-blake3`, `x25519-dalek` |
| `lz4` | `lz4+tcp://` transform | `lz4rip` |
| `ws` | `ws://` / `wss://` WebSocket transport | `rustls`, `rustls-native-certs` (backend-level) |
| `fuzz` | fuzz test suites | - |
| `soak` | soak test suites | - |

## ZMQ fundamentals

ZMQ sockets are opaque message queues that abstract away the network.
The user sends and receives messages. The socket handles connections,
reconnections, framing, and multiplexing internally. The transport
(TCP, IPC, inproc, UDP) is chosen by endpoint URI and is transparent
to the application.

**Reliability is non-negotiable.** ZMQ users expect the library to
Just Work. No errors from peer failures. No hangs. No stuck states.
Connections self-heal automatically. Back-pressure is applied silently.
The library must always take the optimal performance path and recover
from any transient failure without user intervention. A ZMQ library
that surfaces transport-level errors to the user, requires manual
reconnection, or gets stuck in a degraded state is broken. Never
propose a fix that weakens self-healing, adds user-visible failure
modes, or trades reliability for convenience.

Core guarantees that omq must uphold:

- **Send/recv never fail due to peers.** A peer disconnecting, a TCP
  connection dropping, or a slow consumer does not cause `send` or
  `recv` to return an error. The socket reconnects automatically and
  resumes delivery. The only user-visible send errors are protocol
  violations (e.g. REQ sending twice without recv) or socket closed.
- **Connect-before-bind works.** `connect()` queues internally and
  waits for the bind to appear. Never suggest connection ordering as
  a cause for failures or hangs.
- **Automatic reconnection.** ZMTP peers reconnect on disconnect
  with configurable backoff. The application does not manage
  connection lifecycle.
- **Messages are atomic.** A multipart message is delivered in full
  or not at all. No partial delivery.
- **HWM back-pressure, not errors.** When the outbound queue is
  full, the socket either drops (PUB default) or blocks (PUSH
  default, configurable via OnMute). It does not return an error.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paddor/omq.rs](https://github.com/paddor/omq.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
