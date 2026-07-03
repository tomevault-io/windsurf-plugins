---
trigger: always_on
description: Event-driven trading bot framework in Rust. Typed event bus, typed feeds per
---

# bullet-bots

Event-driven trading bot framework in Rust. Typed event bus, typed feeds per
event kind, strategies implemented as actors. Pluggable exchange adapters and
strategies connect through a shared harness that handles lifecycle, shutdown,
and multi-source event dispatch.

## Build & Test

```sh
cargo build                # full workspace
cargo nextest run          # unit + integration tests (default runner)
cargo test --doc           # doctests only (nextest doesn't run these)
cargo clippy               # lints (pedantic enabled)
cargo +nightly fmt         # format (nightly required for import grouping)
```

First-time setup:

```sh
cargo install cargo-nextest --locked
```

The nextest config lives at `.config/nextest.toml`. CI should use the `ci`
profile (`cargo nextest run --profile ci`) which retries once and doesn't
fail-fast.

Validate a config without connecting:

```sh
cargo run --bin bb-bot -- validate --config config/simple-mm-example.toml
```

Generate a keypair (first time):

```sh
cargo run --bin bb-bot -- keygen --network testnet
# → writes ~/.config/bullet/id.key (base58 secret, 0600), prints address + faucet curl
```

Fund and onboard the account (first time). The faucet credits the on-chain
wallet (**testnet only** — on mainnet you fund the wallet with real
bridged/deposited assets instead); `deposit` then moves funds into the perp
margin account and initializes the trading account. Skipping the deposit makes
order placement fail with `user_variants not found`:

```sh
# Run the faucet curl printed by keygen, then:
cargo run --bin bb-bot -- deposit --network testnet --asset USDC --amount 5000
```

Run a bot (default: reads `~/.config/bullet/id.key`):

```sh
cargo run --bin bb-bot -- run --config config/simple-mm-example.toml
```

Or point at an explicit key file / pass a key string for CI:

```sh
export BB_BULLET_KEY_FILE="/path/to/id.key"          # preferred
# OR (base58 from Phantom/delegation export, or hex)
export BB_BULLET_PRIVATE_KEY="<base58-or-hex>"        # fallback
```

## Architecture — the harness, feeds, and actors

The framework is organized around three typed primitives:

### Events

An **event** is any `Clone + Debug + Send + 'static` value. One Rust type per
kind of world change. The five canonical events live in
[`bb-core/src/events.rs`](crates/bb-core/src/events.rs):

| Event              | Semantics                                                                |
|--------------------|--------------------------------------------------------------------------|
| `Trade`            | An execution against our account. One per fill. **Canonical** source of position/PnL changes. |
| `OrderLifecycle`   | An order's state transition (Open → PartiallyFilled → Filled / Cancelled / Rejected). Used for reconcile — never for position updates. |
| `BookUpdate`       | Orderbook snapshot/update.                                               |
| `MarkPriceUpdate`  | Mark price and/or funding rate.                                          |
| `Tick`             | Periodic heartbeat, produced by the framework-provided `TickFeed`.       |

Splitting fills (`Trade`) from lifecycle (`OrderLifecycle`) is the key
invariant: strategies only update inventory from `Trade`, so the double-count
failure mode where a fill is credited twice (once as trade, once as filled
order) is structurally impossible.

### Feeds

A **feed** publishes events of a single type. Feeds own their upstream —
typically a WebSocket — and handle their own reconnection. Feeds implement
[`EventFeed<E>`](crates/bb-core/src/harness/feed.rs):

```rust
#[async_trait]
pub trait EventFeed<E: Event>: Send + 'static {
    async fn run(self: Box<Self>, tx: EventTx<E>, cx: FeedContext) -> Result<(), BotError>;
}
```

Exchanges expose one feed per event type they can produce. The Bullet adapter
(see [`exchanges/bullet/src/connection.rs`](crates/exchanges/bullet/src/connection.rs))
owns a single `ManagedWebsocket` and demultiplexes its messages into four
feeds: `BulletTradeFeed`, `BulletBookFeed`, `BulletOrderLifecycleFeed`,
`BulletMarkPriceFeed`.

`TickFeed` ([`bb-core/src/helpers/tick_feed.rs`](crates/bb-core/src/helpers/tick_feed.rs))
is a framework-provided feed that emits `Tick` events on a fixed interval, so
periodic strategy work (rebalance checks, logging) flows through the same
model as everything else.

### Actors

An **actor** is a stateful event consumer. Every strategy is one actor. An
actor implements [`Actor`](crates/bb-core/src/harness/actor.rs) for lifecycle
(`init` / `wind_down` / `status`) plus [`EventHandler<E>`](crates/bb-core/src/harness/actor.rs)
once per event type it cares about.

```rust
#[async_trait]
impl Actor for GridActor {
    async fn init(&mut self, cx: &ActorContext) -> Result<(), BotError> { ... }
    async fn wind_down(&mut self, reason: &WindDownReason, cx: &ActorContext) -> Result<(), BotError> { ... }
    fn status(&self) -> serde_json::Value { ... }
}

#[async_trait]
impl EventHandler<Trade> for GridActor {
    async fn on_event(&mut self, event: Trade, cx: &ActorContext) -> Result<(), BotError> { ... }
}

#[async_trait]
impl EventHandler<Tick> for GridActor {
    async fn on_event(&mut self, event: Tick, cx: &ActorContext) -> Result<(), BotError> { ... }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bulletxyz/bullet-bots](https://github.com/bulletxyz/bullet-bots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
