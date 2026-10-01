---
trigger: always_on
description: How to author a new RustStream broker crate (the Broker contract, capabilities, conformance)
---


- A broker is its own crate that depends on `ruststream` with `default-features = false` (broker
  clients never go into core). Pre-1.0 the broker crate's version tracks the core minor line
  (core 0.7.x -> broker 0.7.x); breaking broker changes ship as a patch within the line.
- The lifecycle is a ladder of consuming transitions, not a flag: `Broker::connect(self) ->
  Connected` and `ConnectedBroker::shutdown(self) -> Closed`. Each state is a distinct type, so
  subscribing before connect, publishing after shutdown, or a double shutdown do not compile for
  the owner of the handle. The broker itself carries no subscriber or publisher type, so one
  application can mix broker kinds.
- Construction stays synchronous and I/O-free: `new(addrs)` only records config, all network work
  happens in `connect`, and the connected form holds the live client directly - it never checks a
  "maybe connected" state. A shared cell that `connect` fills is a legal internal pattern for
  handles given out before connect, but it is not the contract.
- A task the broker starts on its own behalf (a delayed-redelivery timer, a reply dispatcher, a
  commit window) runs on the runtime `connect` ran on: keep `Handle::current()` from `connect` in
  the connected form and spawn through it, never `tokio::spawn` from a publish, settle or request
  path. `harness::lifecycle` and the `capabilities::*` suites call from a runtime that stops.
- What stays a runtime rule is aliasing: publishers handed out earlier and clones of a shareable
  broker must surface an error when used after shutdown, never a silent success against a dead
  connection. `shutdown` never blocks or panics; do all fallible teardown there.
- Implement `Subscribe` on the **connected** form for the by-name `#[subscriber("topic")]` form,
  and ship a `SubscriptionSource<Connected<B>>` descriptor for broker-specific options (even a
  parameter-less broker ships one). Give the descriptor a constructor and derive `Clone`; add
  `FromName` when a name alone is enough to build it.
- Every descriptor declares `type Copies`, one of three. `AddressedCopies`: this process publishes
  the copies a retry is made of and the descriptor knows where they go (a subject, a topic, a
  stream, a queue); implement `RedeliveryAddressed` beside `SubscriptionSource`, which is what
  makes the address a property of the type. `NamedCopies`: this process publishes them but the
  descriptor addresses none (a wildcard subject, an MQTT filter, a Pulsar pattern, a list of
  topics), and the mount site names one with `.out_retry(policy).to(name)` or with a naming
  transform. `BrokerMoves`:
  the server or the client library moves the delivery itself (a delivery limit with a dead-letter
  exchange, a Pub/Sub dead-letter policy, an SQS redrive policy), so `.out_retry(..)` is a compile
  error at every mount site of that descriptor. The first two pair a retry publisher from
  `DefaultPublish`, so either over a broker without it does not compile.
- `Subscribe` carries the same `type Copies` for the by-name form: `AddressedCopies` where a
  publish under a subscribe name reaches the subscription opened by it, and the name is then the
  address; `NamedCopies` where it is not (a Pub/Sub subscription, an MQTT filter).
  `harness::redelivery_address` holds an addressed descriptor, and the bare `Name` where
  `Subscribe::Copies` is `AddressedCopies`, to its answer; `retry::broker_moves` holds a
  `BrokerMoves` descriptor to the cap and dead-letter destination it declares; and
  `harness::lifecycle` covers the rest of the ladder.
- `declare_retry` hands the descriptor the registration's `max_attempts(..)` and `dead_letter(..)`
  before it subscribes. Apply them to your own topology where the broker has a mechanism for them,
  and only when both are declared; keep the default otherwise and the runtime applies them.
- Override `IncomingMessage::redelivery_count` where the transport counts its own deliveries
  (JetStream `num_delivered`, SQS `ApproximateReceiveCount`, Pub/Sub `delivery_attempt`), so a
  registration's cap counts the broker's redeliveries and not only the copies the runtime
  published. The first delivery answers 1. That count is then the only one a cap reads, the
  framework's header never mixed in; increment `RETRY_COUNT_HEADER` on a copy your crate
  republishes (a wait queue, a retry topic) only where the transport counts nothing.
- Publishers split into policy plus live form: ship a freely constructible policy type carrying the
  options, implement `PublishPolicy<Connected>` to pair it into the live publisher, and implement
  `DefaultPublish` on the connected form when the plain policy works as-is (so `b.include(def)`
  compiles with no `.out_reply(..)`). Make mode selection a policy type transition, not a runtime
  flag: only the transactional policy's live form implements `TransactionalPublisher`.
- Per-message settings (a QoS, a priority, an ordering key, an expiration) are `Publisher::Options`,
  a type you own with every field optional (`()` when you have none). The policy holds the defaults,
  `publish(msg, options)` resolves the call's fields over them, and `options` is `None` on every

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [powersemmi/ruststream](https://github.com/powersemmi/ruststream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
