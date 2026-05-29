---
trigger: always_on
description: This document provides essential information for AI agents working with the RabbitMQ AMQP Python Client codebase.
---

# AGENTS.md - Guide for AI Agents

This document provides essential information for AI agents working with the RabbitMQ AMQP Python Client codebase.

## Project Overview

The RabbitMQ AMQP Python Client is a Python library for interacting with RabbitMQ 4.x using the AMQP 1.0 protocol. It provides both synchronous and asynchronous interfaces for publishing, consuming, and managing RabbitMQ resources.

## Key Components

### Core Classes

- **`Environment`**: Connection pooler that manages AMQP connections
- **`Connection`**: Main connection class for interacting with RabbitMQ
- **`Publisher`**: Publishes messages to RabbitMQ queues/exchanges
- **`Consumer`**: Consumes messages from RabbitMQ queues/exchanges
- **`Management`**: Manages RabbitMQ resources (queues, exchanges, bindings)

### Async Interface

Located in `rabbitmq_amqp_python_client/asyncio/`:
- **`AsyncEnvironment`**: Async version of Environment
- **`AsyncConnection`**: Async version of Connection
- **`AsyncPublisher`**: Async version of Publisher
- **`AsyncConsumer`**: Async version of Consumer
- **`AsyncManagement`**: Async version of Management

The async classes are facades that:
- Wrap synchronous classes
- Execute blocking operations in thread pool executors
- Use `asyncio.Lock` for thread safety
- Implement async context managers


### Thread Safety

The TCP connection is not thread safe. 
If you want to use the client in a multithreaded environment, you should create a connection per thread. 


### Entities and Options

Located in `rabbitmq_amqp_python_client/entities.py`:
- **`StreamConsumerOptions`**: Configuration for stream consumers (supports `offset_specification` as `OffsetSpecification`, `int`, or `datetime`)
- **`StreamFilterOptions`**: Filter options for stream consumers
- **`OffsetSpecification`**: Enum for stream offset positions (first, next, last, timestamp)
- **`ConsumerOptions`**: Configuration for FIFO (Classic/Quorum) consumers; uses `settle_strategy` (ConsumerSettleStrategy) for uniform AMQP 1.0 client interface
- **`ConsumerSettleStrategy`**: Enum for settle strategy: ExplicitSettle, DirectReplyTo, PreSettled
- Queue/Exchange specifications: `QuorumQueueSpecification`, `StreamSpecification`, `ExchangeSpecification`, etc.

## Project Structure

```
rabbitmq_amqp_python_client/
├── __init__.py              # Public API exports
├── address_helper.py        # Address validation and formatting
├── amqp_consumer_handler.py # Base message handler class
├── connection.py            # Connection management
├── consumer.py              # Message consumer implementation
├── publisher.py              # Message publisher implementation
├── management.py             # RabbitMQ management operations
├── environment.py             # Connection pooling
├── entities.py                # Data classes and options
├── exceptions.py              # Custom exceptions
├── options.py                 # Receiver options
├── queues.py                  # Queue specifications
├── ssl_configuration.py       # SSL/TLS configuration
├── utils.py                   # Utility functions
└── asyncio/                   # Async interface implementations
    ├── connection.py
    ├── consumer.py
    ├── publisher.py
    └── management.py

tests/
├── conftest.py               # Pytest fixtures and test helpers
├── test_consumer.py           # Consumer tests (includes stream consumer tests)
├── test_streams.py            # Stream-specific tests
├── test_connection.py          # Connection tests
├── test_publisher.py           # Publisher tests
├── test_management.py          # Management tests
└── asyncio/                   # Async tests

examples/
├── getting_started/           # Basic usage examples
├── streams/                   # Stream queue examples
├── stream_consumer_offset_datetime/  # Datetime offset example
└── ...                        # Other examples
```

## Testing Patterns

### Test Structure

1. **Fixtures**: Defined in `tests/conftest.py`
   - `connection`: Synchronous connection fixture
   - `environment`: Environment fixture
   - `connection_with_reconnect`: Connection with reconnection enabled

2. **Message Handlers**: Custom handlers in `tests/conftest.py`
   - `MyMessageHandlerAccept`: Accepts all messages
   - `MyMessageHandlerAcceptStreamOffset`: For stream offset testing
   - `MyMessageHandlerNoack`: No acknowledgment handler
   - `MyMessageHandlerDiscard`: Discards messages
   - `MyMessageHandlerRequeue`: Requeues messages

3. **Test Pattern**:
   ```python
   def test_feature(connection: Connection, environment: Environment) -> None:
       # Setup
       queue_name = "test-queue-name"
       management = connection.management()
       management.declare_queue(QueueSpecification(name=queue_name))
       
       # Test logic
       try:
           # ... test code ...
       except ConsumerTestException:
           pass  # Expected termination
       finally:
           # Cleanup
           consumer.close()
           management.delete_queue(queue_name)
           management.close()
   ```

### Stream Consumer Tests

When testing `StreamConsumerOptions` with datetime offset:
- Use `StreamSpecification` (not `QuorumQueueSpecification`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rabbitmq/rabbitmq-amqp-python-client](https://github.com/rabbitmq/rabbitmq-amqp-python-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
