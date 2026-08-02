---
trigger: always_on
description: This file provides guidance for AI coding agents working in this repository.
---

# ClickHouse Kafka Connect Development Guide

This file provides guidance for AI coding agents working in this repository.

## Project Overview

`clickhouse-kafka-connect` is the official Apache Kafka Connect **sink connector** for ClickHouse. It is implemented according to the sink connector specification (`SinkConnector`) and writes messages to ClickHouse tables using the official ClickHouse Java client with support for **exactly-once delivery semantics**. 

Key design goals:
- Independent of ClickHouse internals — runs as a standard Kafka Connect worker
- Push-based — ClickHouse never connects to Kafka
- Supports all ClickHouse types (including complex types: Array, Map, Tuple)
  - Converts accurately from `SinkRecord` -> `Record` -> ClickHouse data types
  - See public docs page for the full list of supported data types: https://clickhouse.com/docs/integrations/kafka/clickhouse-kafka-connect-sink#supported-data-types
- Exactly-once delivery via a state machine + ClickHouse's native block deduplication
- Eager schema validation
- Supports all Kafka record formats (see the Data Format Support section below)

Full design rationale: [`docs/DESIGN.md`](./docs/DESIGN.md)

### Package Structure

All source code lives under `src/main/java` in the `com.clickhouse.kafka.connect` package:

| Sub-package           | Contents                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sink`                | - `ClickHouseSinkTask`: main Kafka Connect `SinkTask` implementation, delegates to `ProxySinkTask` <br/> - `ClickHouseSinkConfig`: defines `ConfigDef` for the connector<br/> - `ProxySinkTask`: defines main start/stop/put logic                                                                                                                                                                                                                         |
| `sink.data`           | - `Record`: in-memory representation of a record read from kafka (`SinkRecord`) + metadata<br/> - `Data`: generic wrapper of data (`java.lang.Object`) with a schema<br/> - `SchemaType`: enum that determines the write path of a `Record`                                                                                                                                                                                                                |
| `sink.data.convert`   | - `RecordConvertor`: abstract class that converts `SinkRecord` (Kafka Connect API) to `Record` (clickhouse connector API)<br/> All other classes in this package are implementations of `RecordConverter` for each `SchemaType`                                                                                                                                                                                                                            |
| `sink.db`             | - `DBWriter`: main interface describing the API for start/stop/insert to ClickHouse<br/> - `ClickHouseWriter`: primary implementation of `DBWriter` <br/> - `InMemoryDBWriter`: test implementation of `DBWriter`<br/>- `TableMappingRefresher`: periodic task to update in-memory mapping between ClickHouse table names and their descriptions (`Table`) in `ClickHouseWriter`                                                                           |
| `sink.db.mapping`     | - `Column`: description of a ClickHouse table column (name + `Type` + metadata) <br/>- `Table`: description of a ClickHouse table (name + database + `List<Column>` + metadata) <br/>- `Type`: enum of ClickHouse data types the connector can serialize                                                                                                                                                                                                   |
| `sink.db.helper`      | Contains utilities for writing to ClickHouse                                                                                                                                                                                                                                                                                                                                                                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/clickhouse-kafka-connect](https://github.com/ClickHouse/clickhouse-kafka-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
