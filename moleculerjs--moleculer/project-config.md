---
trigger: always_on
description: **Fast & powerful microservices framework for Node.js**
---

# Moleculer Framework

**Fast & powerful microservices framework for Node.js**

## Project Overview

Moleculer is a modern, production-ready microservices framework for Node.js that enables you to build efficient, reliable, and scalable distributed systems. It provides a comprehensive set of features for microservices architecture including service discovery, load balancing, fault tolerance, and more.

### Key Features
- Promise-based solution with async/await support
- Request-reply concept with event-driven architecture
- Built-in service registry and dynamic service discovery
- Load balanced requests and events (round-robin, random, CPU usage, latency, sharding)
- Comprehensive fault tolerance features (Circuit Breaker, Bulkhead, Retry, Timeout, Fallback)
- Plugin/middleware system
- Versioned services support
- Node.js Streams support
- Service mixins
- Built-in caching solutions (Memory, MemoryLRU, Redis)
- Metrics and tracing capabilities
- Multiple transporters (TCP, NATS, MQTT, AMQP, Redis, Kafka)
- Multiple serializers (JSON, Avro, MessagePack, CBOR)

## Technology Stack

### Core Technologies
- **Node.js**: Version 20.x.x or higher (required)
- **TypeScript**: Full TypeScript support with type definitions
- **JavaScript ES2022**: Modern JavaScript features
- **CommonJS**: Primary module system with ESM support

### Dependencies
- **Core Dependencies**: args, eventemitter2, fastest-validator, glob, ipaddr.js, kleur, lodash, lru-cache, recursive-watch
- **Transport Layer**: Multiple optional peer dependencies for different transporters
- **Logging**: Multiple optional logger integrations
- **Monitoring**: Optional tracing and metrics integrations

## Project Structure

```
moleculer/
├── src/                          # Main source code
│   ├── cachers/                  # Caching implementations
│   │   ├── base.js              # Base cacher class
│   │   ├── memory.js            # Memory cache
│   │   ├── memory-lru.js        # LRU memory cache
│   │   └── redis.js             # Redis cache
│   ├── transporters/            # Transport layer implementations
│   │   ├── base.js              # Base transporter class
│   │   ├── amqp.js              # AMQP transporter
│   │   ├── nats.js              # NATS transporter
│   │   ├── redis.js             # Redis transporter
│   │   ├── mqtt.js              # MQTT transporter
│   │   ├── kafka.js             # Kafka transporter
│   │   └── tcp/                 # TCP transporter implementation
│   ├── serializers/             # Data serialization
│   ├── strategies/              # Load balancing strategies
│   ├── middlewares/             # Middleware implementations
│   ├── tracing/                 # Distributed tracing
│   ├── metrics/                 # Metrics collection
│   ├── registry/                # Service registry
│   ├── validators/              # Validation
│   ├── loggers/                 # Logger implementations
│   ├── service-broker.js        # Core broker class
│   ├── service.js               # Service base class
│   ├── context.js               # Request context
│   ├── transit.js               # Message transport
│   ├── utils.js                 # Utility functions
│   ├── errors.js                # Error handling
│   ├── constants.js             # Framework constants
│   └── middleware.js            # Middleware system
├── types/                       # TypeScript type definitions
├── test/                        # Test suite
│   ├── unit/                    # Unit tests
│   ├── integration/             # Integration tests
│   ├── e2e/                     # End-to-end tests
│   ├── leak-detection/          # Memory leak tests
│   ├── typescript/              # TypeScript tests
│   └── services/                # Test services
├── examples/                    # Example implementations
├── docs/                        # Documentation
├── benchmark/                   # Performance benchmarks
├── bin/                         # CLI tools
│   ├── moleculer-runner.js      # Main CLI runner
│   └── moleculer-runner.mjs     # ESM version
├── index.js                     # Main entry point (CommonJS)
├── index.mjs                    # ESM entry point
├── index.d.ts                   # TypeScript definitions
└── package.json                 # Project configuration
```

## Key Entry Points

### Main Entry Points
- **index.js**: CommonJS main entry point - exports all core classes and utilities
- **index.mjs**: ESM entry point for modern module systems
- **index.d.ts**: TypeScript type definitions

### Core Classes and Modules
- **ServiceBroker**: Main broker class that orchestrates the entire system
- **Service**: Base class for creating microservices
- **Context**: Request context handling
- **Transit**: Message transport layer
- **Registry**: Service registry and discovery
- **Cachers**: Caching implementations
- **Transporters**: Message transport implementations
- **Serializers**: Data serialization
- **Strategies**: Load balancing strategies
- **Middlewares**: Middleware system
- **TracerExporters**: Distributed tracing exporters
- **MetricTypes/MetricReporters**: Metrics collection and reporting

## Development Workflow

### Available Scripts
```bash
# Development
npm run dev                    # Start development server with nodemon


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moleculerjs/moleculer](https://github.com/moleculerjs/moleculer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
