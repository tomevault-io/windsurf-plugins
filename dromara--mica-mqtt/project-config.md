---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mica-mqtt** is a high-performance, low-latency MQTT IoT component for Java 8+, maintained by the Dromara organization. It provides both client and server (broker) implementations supporting MQTT v3.1, v3.1.1, and v5.0 protocols.

**Key Features:**
- Lightweight core (only 500KB)
- Built on t-io async networking framework
- WebSocket MQTT sub-protocol support (compatible with mqtt.js)
- HTTP REST API for server management
- Shared subscriptions and queue subscriptions
- Client/Server support for Spring Boot, Solon, and JFinal
- GraalVM native compilation support
- **Clustering via `mica-mqtt-broker` module** (t-io cluster for node-to-node communication)

**Documentation:** https://mica-mqtt.dreamlu.net/guide/
**Demo Server:** mqtt.dreamlu.net (username: mica, password: mica)

## Common Commands

### Build & Test
```bash
# Full build with tests (includes example module via develop profile)
mvn clean install

# Build without tests
mvn package -DskipTests

# Run all tests
mvn test

# Run specific test class
mvn -Dtest=TopicFilterTypeTest test

# Run specific test method
mvn -Dtest=TopicFilterTypeTest#methodName test

# Compile only
mvn compile
```

### Module-Specific Build
```bash
# Build mica-mqtt-broker (cluster module, not in develop profile by default)
mvn clean install -pl mica-mqtt-broker -am
```

### Release Commands
```bash
# Create snapshot release
mvn clean deploy -P snapshot

# Create production release (includes GPG signing, javadoc)
mvn clean deploy -P release
```

## Architecture Overview

### Module Structure

```
mica-mqtt/
├── mica-mqtt-codec       # MQTT protocol encoding/decoding (no business logic)
├── mica-mqtt-common      # Common utilities, topic filters, message models
├── mica-mqtt-client      # MQTT client implementation
├── mica-mqtt-server      # MQTT broker/server implementation
├── mica-mqtt-broker      # Cluster broker (t-io based, node-to-node communication)
└── starter/              # Framework integrations
    ├── mica-mqtt-client-spring-boot-starter
    ├── mica-mqtt-server-spring-boot-starter
    ├── mica-mqtt-client-solon-plugin
    ├── mica-mqtt-server-solon-plugin
    ├── mica-mqtt-client-jfinal-plugin
    └── mica-mqtt-server-jfinal-plugin
```

**Dependency Flow:**
- `client/server` → `common` → `codec` + `mica-net-core` (both direct dependencies)
- `broker` → `server` + `mica-net-core`

**Active Profile Modules** (`develop` profile, active by default):
`common`, `client`, `server`, `starter`, `example`

Note: `mica-mqtt-codec` and `mica-mqtt-broker` are built as transitive dependencies but not as top-level modules in the develop profile.

### Core Classes

**Client Side:**
- `MqttClient` / `MqttClientCreator`: Main client API with builder pattern
- `IMqttClientSession`: Session management (subscription tracking, message queuing)
- `DefaultMqttClientProcessor`: Message processing pipeline
- `MqttClientAioHandler` / `MqttClientAioListener`: t-io integration layer

**Server Side:**
- `MqttServer` / `MqttServerCreator`: Main server API with builder pattern
- `MqttServerProcessor`: Message routing and delivery
- `IMqttSessionManager`: Client session lifecycle management
- `MqttMessageStore`: Retained messages and will messages storage
- `MqttServerAioHandler` / `MqttServerAioListener`: t-io integration layer

**Broker/Cluster Side (mica-mqtt-broker):**
- `MqttBroker`: Entry point for cluster brokers (`MqttBroker.create()`)
- `MqttClusterManager`: Manages cluster node communication
- `ClusterMqttSessionManager`: Session synchronization across nodes (decorator wrapping `IMqttSessionManager`)
- `ClusterMessageDispatcher`: Routes messages across broker instances
- `BrokerMessage`: Base class for cluster messages

### Design Patterns

1. **Builder Pattern**: `MqttClient.create()`, `MqttServer.create()`, `MqttPublishMessage.builder()`
2. **Listener Pattern**: Event-driven architecture for connections, messages, subscriptions (`MqttProtocolListener`, `IMqttMessageListener`)
3. **Processor Pattern**: Message pipeline with interceptors (`IMqttMessageInterceptor`)
4. **Strategy Pattern**: Pluggable auth (`IMqttServerAuthHandler`), serialization (`IMqttSerializer`), message dispatching (`IMqttMessageDispatcher`)
5. **Decorator Pattern**: `ClusterMqttSessionManager` wraps `IMqttSessionManager` for session sync

### Thread Safety & Async I/O

- Built on **t-io** framework - avoid blocking operations in I/O callback threads
- Core classes like `MqttServer` and `MqttClient` handle concurrent access
- Use `ChannelContext` for per-connection state
- Session storage uses concurrent data structures (`ConcurrentHashMap`)

### Protocol Ports (Default)

| Port  | Protocol          | Purpose                          |
|-------|-------------------|----------------------------------|
| 1883  | TCP               | Standard MQTT                    |
| 8883  | TCP SSL           | MQTT over TLS                    |
| 8083  | WebSocket         | WebSocket MQTT sub-protocol      |
| 8084  | WebSocket SSL     | Secure WebSocket MQTT            |
| 18083 | HTTP              | REST API & MCP interface         |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dromara/mica-mqtt](https://github.com/dromara/mica-mqtt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
