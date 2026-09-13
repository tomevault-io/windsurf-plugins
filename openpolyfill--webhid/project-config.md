---
trigger: always_on
description: Before you act, read `CONTRIBUTING.md` for the project's design principles. They apply to you too.
---

# Agent Guidelines for FF-WebHID

Before you act, read `CONTRIBUTING.md` for the project's design principles. They apply to you too.

## Terminology

- **FF**: Firefox, the project's target browser.
- **HID**: Human Interface Device (USB/Bluetooth input devices).
- **WS / WT / NM**: WebSocket / WebTransport / Native Messaging, the three data-plane transports of the `dataPlane` setting.
- **SAB**: SharedArrayBuffer (removed from the data plane in 2026-07, see below).
- **TLV**: Type-Length-Value, the binary wire format for packed NM messages and collections.
- **VID / PID**: Vendor ID / Product ID (USB device identifiers).
- **COOP / COEP**: Cross-Origin-Opener-Policy / Cross-Origin-Embedder-Policy.
- **IPC**: Inter-Process Communication. **GC**: Garbage Collection.

## Architecture decisions

### Verify "ceilings" before accepting them

Bugs have repeatedly been mistaken for architectural ceilings. Check logs/behavior first. Past false ceilings (each cost a debug cycle):

- "Worker + SAB at the performance ceiling": Worker+SAB never engaged; everything silently fell back to NM.
- "SAB push must run on a Worker to avoid blocking via `Atomics.wait`": false after the drain moved to non-blocking `Atomics.waitAsync`; that mechanism was never in the flow.
- "SAB is zero-copy, so it must be fastest": false by half. SAB write is zero-copy but drain still copies once (`HIDInputReportEvent` needs exclusive ownership). The no-SAB path uses Transferable buffers for the worker-to-page handoff, but the worker parser first copies each report payload into an owned buffer.
- "Transfer the whole WS frame buffer instead of copying the payload" broke Chromium WebHID semantics: pages do `new Uint8Array(event.data.buffer)`, so `event.data.buffer.byteLength` must equal `event.data.byteLength` (`byteOffset === 0`). Reverted (saved ~30-50µs/report). Lesson: count copies and shape changes to the consumer; the `.buffer` contract is part of the journey.

### WS data plane runs in a dedicated Worker

Profiler-confirmed: WS receive/parse on the content main thread competes with page rendering (510 vs 681 msg/s late-phase under render load); NM is CPU-isolated (subprocess).

**Final WS architecture**: `daemon <-WS-> Worker (no SAB) <-MessageChannel port, transferred once at setup-> page`.

The worker owns the WS connection and posts each report directly to the page through the transferred port. The worker parser allocates an owned buffer for each report before that transfer, so "zero-copy" applies only to the worker-to-page handoff. The bridge is not on the steady-state report path. It creates and tears down the worker and handles setup, mode changes, and NM fallback.

There is no implemented worker-to-bridge-to-page relay fallback. A worker spawn or transport failure selects the NM path instead.

- Data plane switches mid-session (WS ↔ NM) via one control-plane command; a duplicated/dropped report in the switch instant is accepted as user-caused.

### SAB removed entirely (2026-07)

After the ring-buffer alloc bug was fixed, SAB lost to the simpler no-SAB path while carrying ongoing costs (COOP/COEP, Atomics/ring-buffer complexity). It was a necessary rung: it found the alloc bug and the CPU contention. Ring buffer detail: 8192 slots sized by an _estimated_ max report size was a 16MB init allocation; exact per-report size at parse time + 64 slots fixed it (later removed with SAB). Drain never needed >1 occupied slot, even at 8000Hz.

### Rate-gated WS batching (2026-08)

Under a render-saturated main thread, an earlier loss benchmark recorded WS losses of about 1.6-4.3% at high rates. The current rate-gated `run_sender` path in `crates/webhid-daemon/src/batching.rs` tracks reports flushed per 4ms window; at 12 reports per window it uses an 8ms coalesce window. Sparse traffic keeps the 25µs coalesce path. These are benchmark observations, not guarantees for every browser or workload. Knobs: `WEBHID_WS_HIGH_RATE_MS` (8), `WEBHID_WS_RATE_WINDOW_MS` (4), `WEBHID_WS_HIGH_RATE_COUNT` (12); fixed `WEBHID_WS_BATCH_MS` path also exists. Do not remove the rate gate without rerunning the dedicated 8000Hz render-saturated workload.

### Tradeoffs without a universally correct answer become settings

The `dataPlane` setting selects `wt`, `ws`, or `nm`. Daemon-backed control requests use the persistent Native Messaging Port, while browser-local and extension-page operations terminate in addon state. Native Messaging trust differs by deployment: direct daemon-as-host mode relies on the browser's Native Messaging host authorization and OS process boundary; the forwarder profile adds platform IPC checks. On Linux, the daemon checks a Unix-socket peer's `webhid` group and the forwarder accepts a root or same-UID daemon peer. These checks do not apply universally, and handshake/open are pre-session operations that do not carry a Session token. WS and WT report transports use a per-session derived authentication hash plus live Session authority. `wt` is preferred where the handshake offers it, `ws` is the alternate worker network path, and a missing WT port uses WS; failed worker setup or transport setup falls back to NM.

## Project facts

### Benchmarks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenPolyfill/WebHID](https://github.com/OpenPolyfill/WebHID) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
