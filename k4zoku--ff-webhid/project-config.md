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
- "SAB is zero-copy, so it must be fastest": false by half. SAB write is zero-copy but drain still copies once (`HIDInputReportEvent` needs exclusive ownership). The no-SAB path (`postMessage` + Transferable) is zero-copy end to end.
- "Transfer the whole WS frame buffer instead of copying the payload" broke Chromium WebHID semantics: pages do `new Uint8Array(event.data.buffer)`, so `event.data.buffer.byteLength` must equal `event.data.byteLength` (`byteOffset === 0`). Reverted (saved ~30-50µs/report). Lesson: count copies and shape changes to the consumer; the `.buffer` contract is part of the journey.

### WS data plane runs in a dedicated Worker

Profiler-confirmed: WS receive/parse on the content main thread competes with page rendering (510 vs 681 msg/s late-phase under render load); NM is CPU-isolated (subprocess).

**Final WS architecture**: `daemon <-WS-> Worker (no SAB) <-MessageChannel port, transferred once at setup-> page`.

- The Worker owns the WS connection and posts each report directly to the page via the transferred port. One hop per report, zero copies.
- The bridge only spawns/despawns workers and relays the port once; it sits on no per-message hot path. Data worker is per-device, spawned on `open()`, killed on close.
- Fallback: 2-hop relay (worker → bridge → page, both Transferable) if direct port transfer fails.
- Data plane switches mid-session (WS ↔ NM) via one control-plane command; a duplicated/dropped report in the switch instant is accepted as user-caused.

### SAB removed entirely (2026-07)

After the ring-buffer alloc bug was fixed, SAB lost to the simpler no-SAB path while carrying ongoing costs (COOP/COEP, Atomics/ring-buffer complexity). It was a necessary rung: it found the alloc bug and the CPU contention. Ring buffer detail: 8192 slots sized by an _estimated_ max report size was a 16MB init allocation; exact per-report size at parse time + 64 slots fixed it (later removed with SAB). Drain never needed >1 occupied slot, even at 8000Hz.

### Rate-gated WS batching (2026-08)

Under a render-saturated main thread, WS loses ~1.6-4.3% of input reports at >= ~3kHz (source-verified in mozilla-central: WS-to-worker delivery crosses the content main thread via PWebSocket IPC → `RecvOnBinaryMessageAvailable` → unbounded `ChannelEventQueue`; no drop mechanism, just late delivery counted as lost after the drain window). NM drops 0. Drop curve scales with frame rate (~1-4% at 1000-8000 frames/s, 0 at 125/s): queuing delay, not per-frame probability.

Fix: `run_sender` (`crates/webhid-daemon/src/batching.rs`) tracks reports flushed per 4ms window; at 12 reports/4ms (~3kHz) it accumulates ~8ms frames instead of flushing per ~1ms. Sparse traffic keeps the 25µs coalesce path (p50 unchanged, 0.86-0.92ms). Render-load loss: 4.32% worst-run → 0.000-0.583%. WT shares `run_sender` (loss 0.000). Tradeoff: >= ~3kHz streams pay up to 8ms delivery latency (chunked, in order). Knobs: `WEBHID_WS_HIGH_RATE_MS` (8), `WEBHID_WS_RATE_WINDOW_MS` (4), `WEBHID_WS_HIGH_RATE_COUNT` (12); fixed `WEBHID_WS_BATCH_MS` path also exists. Do not remove the rate gate: idle benchmarks never see the drop; repro needs the 8000Hz loss workload on a render-saturated page.

### Tradeoffs without a universally correct answer become settings

Single `dataPlane` setting (`ws` / `wt` / `nm`). Control plane (handshake, open/close, setDataPlane) always goes over the NM port in the background: NM control verifies the peer at OS level (SO_PEERCRED + webhid group on the abstract socket) on top of the session token; ws control was token-only and removed (c964326, 2026-07-22). Default is `wt` (profiler: ~18pp less content main-thread CPU, 3.8x less main-thread IPC under render load; WT reads a DataPipe on the worker with zero WebSocket IPC), `ws` on older Firefox; WS is the fallback when the WT port is not offered or page CSP forces NM.

## Project facts

### Benchmarks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [K4zoku/FF-WebHID](https://github.com/K4zoku/FF-WebHID) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
