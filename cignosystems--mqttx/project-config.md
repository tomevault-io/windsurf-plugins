---
trigger: always_on
description: Guidance for AI coding assistants integrating **MqttX** into a project.
---

# AGENTS.md

Guidance for AI coding assistants integrating **MqttX** into a project.
Read this before suggesting code that uses this library — it captures the
mental model and the mistakes agents most often make.

> Modifying MqttX itself? See `CONTRIBUTING.md` for repo layout, test commands,
> and deferred work.

## What MqttX is

A single hex package (`{:mqttx, "~> 0.10"}`) that ships **three independent
pieces** — choose only what you need:

| Piece | Module | Use when |
|-------|--------|----------|
| **Wire codec** | `MqttX.Packet.Codec` | You have your own transport and just need encode/decode |
| **Client** | `MqttX.Client` | Your app connects to an MQTT broker (AWS IoT, EMQX, HiveMQ, Mosquitto, …) |
| **Broker** | `MqttX.Server` | You are *running* an MQTT broker (e.g. an IoT backend that owns its devices) |

Most apps want only the **client**. Build a broker only when you need to own
the message routing — for talking to a third-party broker, the client is
sufficient on its own.

## Picking a transport

The codec is dep-free; transports are optional packages:

| Transport | Add to deps |
|-----------|-------------|
| TCP / TLS client (`tcp` / `ssl`) | nothing extra |
| WebSocket client (`ws` / `wss`) | nothing extra (RFC 6455 client is built-in) |
| TCP server | `{:thousand_island, "~> 1.4"}` (preferred) or `{:ranch, "~> 2.2"}` |
| WebSocket server | `{:bandit, "~> 1.6"} + {:websock_adapter, "~> 0.5"}` |

If `MqttX.Transport.ThousandIsland` (or `Ranch`, or `Bandit`) fails at server
startup with an undefined-module / undefined-function error, the
corresponding optional dep is missing from `mix.exs` — that's the single most
common setup mistake.

## Mental model — client side

```
your code  ──MqttX.Client.subscribe──▶  broker
your code  ──MqttX.Client.publish───▶  broker
                                          │
              MqttX.Client ◀─PUBLISH──────┘
                   │
                   ▼
       handler_module.handle_mqtt_event(:message, {topic, payload, packet}, state)
```

- The client is a **GenServer**. You don't poll it — it pushes events to a
  handler module.
- `MqttX.Client.connect/1` blocks until CONNACK arrives, so once it returns
  `{:ok, pid}` the session is live and you can immediately subscribe/publish.
- `subscribe/3` is synchronous and **waits for SUBACK** before returning
  `{:ok, granted_qos_list}`. `publish/4` returns `:ok` as soon as the packet
  is written to the socket (it does not wait for PUBACK at QoS 1/2 — those
  acks are tracked in the background and surfaced via the handler module).
- If the connection has dropped (and not yet reconnected), `subscribe`,
  `publish`, and `unsubscribe` return `{:error, :not_connected}` immediately —
  they do not queue.
- The handler module implements **`handle_mqtt_event/3`**, which receives:
  - `(:connected, %{properties: props, session_present: bool}, state)` — after CONNACK success
  - `(:disconnected, reason, state)` — `reason` is `:closed`, `:pingresp_timeout`, `{:error, posix}`, or `{:server_disconnect, code, props}`
  - `(:message, {topic, payload, full_packet}, state)` — for each PUBLISH

`topic` arrives as a **list of segments** (`["sensors", "room1", "temp"]`),
not the original string — use `Enum.join(topic, "/")` if you need to round-trip.

## Mental model — broker side

`use MqttX.Server` defines a behaviour with one callback per MQTT verb:

```
device  ──CONNECT──▶  handle_connect(client_id, creds, info, state)
device  ──SUBSCRIBE─▶  handle_subscribe(topics, state)         → grant per-topic QoS
device  ──PUBLISH──▶  handle_publish(topic, payload, opts, state)
device  ──DISCONNECT▶  handle_disconnect(reason, state)

your app ──send(broker_pid, msg)─▶ handle_info(msg, state)
                                       └─▶ {:publish, topic, payload, state}  (fan out to device)
```

Servers are *per-connection* state machines — `state` is one device's state.
For app-wide state (subscriber registry, message bus), use Phoenix.PubSub or
`:pg` from inside the callbacks.

## Idiomatic patterns

### Receive messages on the client

```elixir
defmodule MyApp.MqttHandler do
  def handle_mqtt_event(:connected, _info, state), do: state
  def handle_mqtt_event(:disconnected, _reason, state), do: state

  def handle_mqtt_event(:message, {topic, payload, _packet}, state) do
    Logger.info("got #{payload} on #{Enum.join(topic, "/")}")
    state
  end
end

{:ok, c} = MqttX.Client.connect(
  host: "broker.example.com",
  client_id: "my-app-#{node()}",
  handler: MyApp.MqttHandler,
  handler_state: %{}
)

{:ok, _granted} = MqttX.Client.subscribe(c, "sensors/#", qos: 1)
```

### Bridge MQTT broker ↔ Phoenix.PubSub (fan-out)

```elixir
defmodule MyBroker do
  use MqttX.Server

  def init(_), do: %{}

  def handle_connect(client_id, _creds, _info, state) do
    Phoenix.PubSub.subscribe(MyApp.PubSub, "downlink:#{client_id}")
    {:ok, Map.put(state, :client_id, client_id)}
  end

  def handle_publish(topic, payload, _opts, state) do
    Phoenix.PubSub.broadcast(MyApp.PubSub, "uplink", {state.client_id, topic, payload})
    {:ok, state}
  end

  def handle_info({:downlink, topic, payload}, state) do
    {:publish, topic, payload, %{qos: 1, retain: false}, state}
  end


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cignosystems/mqttx](https://github.com/cignosystems/mqttx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
