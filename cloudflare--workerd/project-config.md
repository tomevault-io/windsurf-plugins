---
trigger: always_on
description: An informal specification of how a `node:http` `Server` drives the web
---

# node:http server × web streams

An informal specification of how a `node:http` `Server` drives the web
streams underneath it — the `Request` body it pumps into the
`IncomingMessage`, and the `ReadableStream` the `ServerResponse` builds as
the body of the `Response` it hands back to `fetch()` — derived from and
kept in lockstep with the test suite in this directory. **The tests are
the normative artifact**; this document maps behaviors to the tests that
assert them. Every test runs against the C++ streams implementation
(`http-server-cpp.wd-test`) and the TypeScript one
(`http-server-ts.wd-test`). The general server surface (headers, options,
ports, `listen`/`close` lifecycle, `cloudflare:node` helpers) is owned by
`src/workerd/api/node/tests/http-server-nodejs-test.js`; this suite owns
the STREAMS interaction only.

The implementation under test is `src/node/internal/internal_http_server.ts`
(`Server#onRequest`/`#toReqRes` and its `[captureRejectionSymbol]`, and
`ServerResponse`: the Response promise, `#toFetchResponse`,
`destroy`/`#emitClose`),
`internal_http_incoming.ts` (`IncomingMessage#tryRead`, `_read`,
`_destroy`) and the `OutgoingMessage` write path in
`internal_http_outgoing.ts`.

## Infrastructure

No sidecar. The worker is bound to itself as `SERVICE`; its default
handler (`main.js`) routes every incoming Request to the current test's
server through `handleAsNodeRequest`. `harness.js` offers the two ways a
Request reaches a server:

- `env.SERVICE.fetch(...)`: through the service binding. The runtime pumps
  bodies across it (the production shape); a cancellation on one side
  reaches the other only once the exchange completes.
- `dispatch(new Request(...))`: an in-isolate Request handed to the server
  directly, so its body stream IS the test's stream and cancellation is
  observable at once. Tests that need it call `remember(env, ctrl)` first.

Tests run sequentially, one server (`withServer`) at a time.

## Core semantics

### The request body

- The `Request` body is pumped into the `IncomingMessage` by one default
  reader, acquired on the first `_read()` and held for the message's
  lifetime; the pump reads until `push()` reports backpressure or EOF and
  `_read()` restarts it with the same reader. A request without a body
  (GET) ends at once with `complete` set.
- Chunks arrive as Buffers (strings under `setEncoding`), whole and in
  order; a body the client streams arrives incrementally and is chunked
  (no Content-Length), a `FixedLengthStream` body announces its length. A
  'data' listener attached inside the handler still sees the body.
- `pause()` holds delivery, `resume()` continues it without loss, also for
  a body larger than the high-water mark. `pipe()` to one or several node
  destinations and `pipeline(req, TransformStream, res)` work; `pipe()` is
  the Readable's: a destination's backpressure pauses the body and 'drain'
  resumes it, the destination hears 'pipe'/'unpipe', a destination that
  errors is unpiped (no further write reaches it), `unpipe()` stops
  delivery and pauses a source left without destinations, and a source
  error is not forwarded to the destination (that is `pipeline()`'s job).
- `destroy()`: 'aborted' when the body was not complete, 'error' only when
  the message has an 'error' listener (an unlistened `destroy(err)` is
  swallowed), 'close' always; the body stream is cancelled with the destroy
  reason (`undefined` for a bare `destroy()`), through the held reader when
  the pump had acquired one, unless the body was already read to completion.
  A read pending across `destroy()` is dropped, however the stream settles
  it — also on the runtime's stream across the binding (ledger #1).
- The body stream failing under the message — erroring mid-upload, or while
  the handler has the message paused with a read pending underneath, or
  yielding a chunk the message cannot take (a view over a detached
  ArrayBuffer: the conversion's `TypeError`) — aborts it: 'aborted', the
  error, 'close' with `complete` false; the response can still be sent.
  `pause()` then `resume()` inside every 'data' loses nothing.

### The response body

- Headers go out at the first `write()`/`end()` (`writeHead()` only formats
  them; the first write sends them implicitly if needed), which resolves
  the `Response` — while the handler is still writing. The body is a
  `new ReadableStream({ type: 'bytes' })`: writes before the headers are
  buffered and flushed into it at that point, later writes are enqueued
  as they come, so a client reads chunks before `end()`.
- Every chunk type is delivered (string, Buffer, `Uint8Array`, explicit
  encoding), empty writes contribute nothing, many small and large writes
  arrive whole. A declared Content-Length caps the body (extra bytes
  dropped, fewer sent as they are) at `parseInt`'s reading of it — a
  non-numeric value leaves the body uncapped, zero or a negative value
  drops every chunk, a fraction or padded number caps at its integer part.
  (The header value itself is not validated; whether a malformed one keeps
  reaching the client is deliberately unpinned.) 204 and 304, and the reply to a HEAD
  (marked bodiless before the handler runs), have a null body and drop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/workerd](https://github.com/cloudflare/workerd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
