---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A standalone Spring Boot 3.5 application (Java 21) that connects to the **Coinbase Prime WebSocket feed** (`wss://ws-feed.prime.coinbase.com`) and listens to the `heartbeats` and `orders` channels. Products are partitioned into groups of up to 10, with a dedicated WebSocket connection per group. Received orders are buffered in a shared thread-safe queue and consumed by a background logger.

## Build & Run

```bash
# Build (skipping tests)
mvn package -DskipTests

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=PrimeMessageProcessorTest

# Run a single test method
mvn test -Dtest=PrimeMessageProcessorTest#checkSequence_triggersReconnectOnGap

# Run the application (requires env vars below)
mvn spring-boot:run
```

### Required environment variables

```bash
export ACCESS_KEY=...
export SIGNING_KEY=...
export PASSPHRASE=...
export PORTFOLIO_ID=...
```

## Architecture

### Technology stack

| Concern | Choice |
|---|---|
| Runtime | Java 21 (virtual threads via `Thread.ofVirtual()`) |
| Framework | Spring Boot 3.5 (no embedded web server — `web-application-type: none`) |
| WebSocket | Jakarta WebSocket API 2.2 backed by Tyrus 2.2 (Grizzly NIO transport) |
| JSON | Jackson (managed by Spring Boot BOM) |
| Logging | SLF4J + Logback, console only (no file appenders) |

### Package layout

```
com.coinbase.prime.samples
├── PrimeWebSocketApplication        # @SpringBootApplication entry point
├── config/
│   ├── CoinbasePrimeProperties      # @ConfigurationProperties("coinbase.prime")
│   └── WebSocketClientConfig        # @Bean ClientManager + ObjectMapper
├── model/
│   ├── PrimeMessage                 # Top-level inbound envelope (channel, sequence_num, events)
│   ├── Order                        # Individual order (all fields as String to preserve precision)
│   └── FeeDetails                   # Nested fee breakdown within an Order
├── service/
│   ├── SignatureService             # HMAC-SHA256 signing for subscribe/unsubscribe messages
│   ├── OrderQueueService            # LinkedBlockingQueue<Order>(5 000) — shared bounded buffer; enqueue() returns boolean for back-pressure
│   └── PrimeMessageProcessor        # Core business logic (sequence tracking, routing, reconnect trigger)
│                                    # NOT a Spring bean — one instance per PrimeConnectionWorker
├── websocket/
│   └── PrimeWebSocketEndpoint      # Jakarta @ClientEndpoint — NOT a Spring bean; created fresh per connection
├── connection/
│   ├── PrimeConnectionManager      # ApplicationRunner + DisposableBean; partitions products, owns workers
│   ├── PrimeConnectionWorker       # Per-connection loop (not a Spring bean)
│   ├── ConnectionRegistry          # @Service; holds ConnectionHealth for all workers, logs report every 60 s
│   ├── ConnectionHealth            # Mutable health record per worker (status + failure counts)
│   ├── ConnectionStatus            # Enum: CONNECTING, CONNECTED, CLOSED
│   └── FailureType                 # Enum: failure categories + FailureType.from(String) classifier
└── consumer/
    └── OrderConsumer                # Drains OrderQueueService on a virtual thread; logs each order
```

### Data flow

```
Coinbase Prime WSS (one connection per ≤10-product partition)
       │ raw JSON frames
       ▼
PrimeWebSocketEndpoint (@ClientEndpoint, per-connection instance)
       │ delegates
       ▼
PrimeMessageProcessor (one instance per PrimeConnectionWorker)
  ├── global sequence check → reconnect if gap
  ├── heartbeats  → debug log
  ├── orders      → OrderQueueService.enqueue()
  └── error frame → reconnect
       │
       ▼
OrderQueueService (LinkedBlockingQueue, 5 000 capacity, shared by all workers)
       │
       ▼
OrderConsumer (virtual thread, classifies and logs each order)
```

### Connection lifecycle

`PrimeConnectionManager.run()` partitions `product-ids` into batches of at most 10 and starts one `PrimeConnectionWorker` per batch. It then blocks on a `CountDownLatch` to keep the main thread alive (Spring Boot with `web-application-type: none` exits as soon as `run()` returns; virtual threads are daemon threads and do not prevent JVM exit).

Each `PrimeConnectionWorker` runs a connection loop on a dedicated virtual thread (`ws-connection-loop-N`):

1. Sets status to `CONNECTING`, creates a new `PrimeWebSocketEndpoint` instance.
2. Calls `ClientManager.connectToServer(endpoint, uri)` — blocks until the WebSocket handshake completes (connect timeout: 10 s).
3. Sets status to `CONNECTED`, resets sequence numbers, sends signed `subscribe` frames for `heartbeats` then `orders`.
4. Schedules a 30-second ping via `ScheduledThreadPoolExecutor`.
5. Awaits `endpoint.getCloseFuture()` — a `CompletableFuture<Void>` that completes in `@OnClose` or `@OnError`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rcbgr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
