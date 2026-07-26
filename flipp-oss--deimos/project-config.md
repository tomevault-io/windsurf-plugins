---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Deimos is a Ruby framework for Kafka development that marries Kafka with schema definitions (Avro/Protobuf), ActiveRecord, and provides a comprehensive toolbox. Built on top of Karafka (which itself builds on RDKafka), it provides schema encoding/decoding, database integration, metrics, tracing, and test helpers.

## Development Commands

### Testing

```bash
# Run all tests
bundle exec rspec

# Run a single test file
bundle exec rspec spec/path/to/file_spec.rb

# Run a specific test
bundle exec rspec spec/path/to/file_spec.rb:LINE_NUMBER
```

### Linting

```bash
# Run Rubocop linter
bundle exec rubocop

# Auto-correct issues
bundle exec rubocop -a
```

### Schema Class Generation

When modifying schema-related code, you may need to regenerate test schema classes:

```bash
# Regenerate test schema classes (used after schema changes)
bundle exec ./regenerate_test_schema_classes.rb

# Generate Protobuf classes
protoc -I spec/protos --ruby_out=spec/gen --ruby_opt=paths=source_relative spec/protos/**/*.proto
```

### Rake Tasks

```bash
# Start Deimos consumer (in Rails environment)
rake deimos:start

# Start outbox backend producer
rake deimos:outbox

# Start database poller
rake deimos:db_poller

# Generate schema classes
rake deimos:generate_schema_classes
```

## Architecture

### Layer Structure

Deimos is built in layers:
1. **RDKafka** - Low-level Kafka client providing producer/consumer APIs
2. **Karafka** - Threaded consumers wrapper with lifecycle management
3. **Deimos** - Full framework with schema integration, database features, metrics, and utilities

### Key Directories

- `lib/deimos/` - Core Deimos code
- `lib/deimos/backends/` - Producer backends (kafka, kafka_async, outbox, test)
- `lib/deimos/schema_backends/` - Schema handlers (Avro local/registry/validation, Protobuf, plain, mock)
- `lib/deimos/metrics/` - Metrics providers (DataDog, mock)
- `lib/deimos/tracing/` - Tracing providers (DataDog, mock)
- `lib/deimos/utils/` - Utilities (DB poller, outbox producer, etc.)
- `lib/deimos/config/` - Configuration classes
- `lib/deimos/ext/` - Extensions to Karafka routing
- `lib/generators/` - Rails generators for migrations
- `lib/tasks/` - Rake tasks

### Core Concepts

#### Schema Backends

Schema backends encode/decode payloads. All backends must implement:
- `encode(payload, topic:)` - Encode payload to binary/string format
- `decode(payload)` - Decode binary/string to hash
- `validate(payload)` - Validate payload against schema
- `coerce(payload)` - Coerce payload to match schema types
- `schema_fields` - List fields in schema (used with ActiveRecord)
- Define a `mock` backend for testing

Available backends: `:avro_local`, `:avro_schema_registry`, `:avro_validation`, `:proto_schema_registry`, `:proto_local`, `:mock`, `:plain`

#### Producer Backends

Producer backends determine how messages are sent. All backends inherit from `Deimos::Backends::Base` and implement `execute(messages)`.

Available backends:
- `:kafka` - Send directly to Kafka (default)
- `:kafka_async` - Async variant of kafka backend
- `:outbox` - Transactional outbox pattern (save to DB, send async)
- `:test` - For testing (stores messages in memory)

#### Consumer Types

- `Deimos::Consumer` - Base consumer class
  - Per-message: Override `consume_message(message)` and set `each_message true`
  - Batch: Override `consume_batch` (receives `messages` collection)

- `Deimos::ActiveRecordConsumer` - Automatically saves/updates ActiveRecord models
  - Per-message mode: Uses `fetch_record`, `assign_key`, `destroy_record`
  - Batch mode: Uses `activerecord-import` for bulk operations
  - Override `record_attributes(payload, key)` to customize attributes

#### Producers

- `Deimos::Producer` - Base producer class
  - Call `self.produce([{payload: ..., key: ..., topic: ...}])`
  - Override `partition_key(payload)` for custom partitioning

- `Deimos::ActiveRecordProducer` - Produces from ActiveRecord models
  - Set `record_class Widget, refetch: false`
  - Override `generate_payload(attributes, record)` to customize payload
  - Override `watched_attributes(record)` to add non-schema fields

#### Key Configuration

Every producer must define `key_config`:
- `key_config none: true` - No keys (events)
- `key_config plain: true` - Unencoded keys (legacy)
- `key_config schema: 'MySchema-key'` - Use existing key schema
- `key_config field: 'my_id'` - Auto-generate key schema from value field

#### KafkaSource Mixin

The `Deimos::KafkaSource` mixin adds callbacks to ActiveRecord models to automatically send Kafka messages on save/destroy:

```ruby
class Widget < ActiveRecord::Base
  include Deimos::KafkaSource

  def self.kafka_producers
    [MyProducer]
  end

  def self.kafka_config
    { update: true, delete: true, import: true, create: true }
  end
end
```

#### Outbox Pattern (Database Backend)

The outbox pattern provides transactional guarantees:
1. Messages are validated, encoded, and saved to `kafka_messages` table
2. Separate thread pool (via `Deimos::Utils::OutboxProducer`) reads from DB and sends to Kafka

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipp-oss/deimos](https://github.com/flipp-oss/deimos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
