---
trigger: always_on
description: This file provides guidance to AI coding assistants working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants working with code in this repository.

## Project Overview

MonsterMQ is a MQTT broker built with Kotlin on Vert.X and Hazelcast with data persistence through PostgreSQL, CrateDB, MongoDB, or SQLite. It features a web dashboard, GraphQL API, MCP (Model Context Protocol) Server for AI integration, REST API, Prometheus metrics, I3X API for manufacturing data, AI agents with GenAI providers, a flow engine for automation workflows, and device bridging for OPC UA, PLC4X, Kafka, NATS, Redis, Telegram, WinCC OA/Unified, Neo4j, JDBC loggers, and SparkplugB.

### MonsterMQ Edge Broker

MonsterMQ also has an edge broker variant implemented in Go for a lightweight footprint. The edge broker exposes the same GraphQL interface shape as the main broker where supported, but currently implements only a limited subset of the full broker functionality. More GraphQL-backed functionality will be added over time, so new dashboard and API work should consider compatibility with both the full broker and the edge broker when practical.

## Build and Run Commands

### Building the Project
```bash
cd broker
mvn clean package
```

### Running Locally
```bash
cd broker
# Run the broker
java -classpath target/classes:target/dependencies/* at.rocworks.MonsterKt [-cluster] [-log INFO|FINE|FINER|FINEST|ALL]
# Or use the convenience script
./run.sh [-cluster] [-log INFO|FINE|FINER|FINEST|ALL]
```

#### iX Dashboard (dashboard/)
The dashboard using Siemens iX lives in `dashboard/` and is built with Vite. It outputs to `dashboard/dist/`.

```bash
cd dashboard
npm install
npm run dev          # Vite dev server on http://localhost:5173, proxies /graphql to :4000
npm run build        # Outputs to dashboard/dist/
```

### Running Tests

#### Kotlin Unit Tests
```bash
cd broker
mvn test
```

#### Python Integration Tests
```bash
cd tests
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run all tests
pytest

# Run by category
pytest pytest_tests/mqtt3/     # MQTT v3.1.1 tests
pytest pytest_tests/mqtt5/     # MQTT v5 tests
pytest pytest_tests/graphql/   # GraphQL API tests
pytest pytest_tests/opcua/     # OPC UA tests
pytest pytest_tests/database/  # Database backend tests

# Run single test
pytest pytest_tests/mqtt3/test_basic_pubsub.py::test_basic_pubsub_qos0 -v
```

Test configuration via environment variables:
```
MQTT_BROKER=localhost  MQTT_PORT=1883  MQTT_USERNAME=Test  MQTT_PASSWORD=Test
GRAPHQL_URL=http://localhost:4000/graphql
```

### Docker Commands
```bash
# Build Docker image
cd docker
./build

# Run with Docker
docker run -v ./log:/app/log -v ./config.yaml:/app/config.yaml rocworks/monstermq [-cluster] [-log INFO|FINE|FINER|FINEST|ALL]
```

## Code Architecture

### Core Components

- **Main Entry Point**: `broker/src/main/kotlin/Main.kt` -> `Monster.kt`
- **MQTT Server**: `MqttServer.kt` - Handles MQTT protocol implementation (v3.1.1 and v5)
- **MQTT Client Handler**: `MqttClient.kt` - Manages individual client connections and session state
- **Message Bus**: `bus/` directory - Abstraction for message distribution (Vertx EventBus or Kafka)
- **Storage Layer**: `stores/` directory - Modular storage implementations for sessions, retained messages, and archives
- **GraphQL Server**: `graphql/GraphQLServer.kt` - GraphQL API for dashboard and external integrations
- **Flow Engine**: `flowengine/` directory - Automation workflow engine with script execution
- **Auth System**: `auth/` directory - User management, password encoding, ACL caching

### Storage Architecture

The system uses different store types for different purposes:
- **SessionStore**: Persistent client sessions (PostgreSQL, CrateDB, MongoDB, SQLite)
- **QueueStore**: Queued messages for offline clients (PostgreSQL, MongoDB, SQLite). V1 uses a two-table design, V2 uses a single-table PGMQ-inspired design. Defaults to matching SessionStoreType if not set.
- **RetainedStore**: Retained MQTT messages (Memory, Hazelcast, PostgreSQL, CrateDB)
- **MessageArchive**: Historical message storage (PostgreSQL, CrateDB, MongoDB, SQLite, Kafka)
- **LastValueStore**: Current value cache for topics (Memory, Hazelcast, PostgreSQL, CrateDB)
- **MetricsStore**: Broker metrics history (PostgreSQL, CrateDB, MongoDB, SQLite)

### Device Bridging Architecture

Devices follow a pattern of Extension (cluster-aware coordinator) + Connector (per-device verticle):

- `devices/mqttclient/` - MQTT client bridge
- `devices/kafkaclient/` - Kafka client bridge
- `devices/natsclient/` - NATS client bridge
- `devices/redisclient/` - Redis client bridge
- `devices/telegramclient/` - Telegram bot client bridge
- `devices/opcua/` - OPC UA client
- `devices/opcuaserver/` - OPC UA server
- `devices/plc4x/` - PLC4X protocol (industrial PLCs, supports JSON payload extraction via `jsonPath`)
- `devices/winccoa/` - WinCC OA integration
- `devices/winccua/` - WinCC Unified integration
- `devices/neo4j/` - Neo4j graph database bridge
- `devices/sparkplugb/` - SparkplugB decoder

For device integration guidance, see `dev/plans/DEVICE_INTEGRATION.md`.

### Key Directories

- `broker/src/main/kotlin/bus/` - Message bus implementations (Vertx, Kafka)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vogler75/monster-mq](https://github.com/vogler75/monster-mq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
