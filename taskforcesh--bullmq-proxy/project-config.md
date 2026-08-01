---
trigger: always_on
description: **BullMQ Proxy** is a lightweight HTTP-based proxy service for [BullMQ](https://bullmq.io/), enabling interaction with Redis-backed job queues through RESTful HTTP and WebSocket APIs. Built with [Bun.js](https://bun.sh/), it provides a language-agnostic interface to BullMQ's powerful job queue capabilities.
---

# BullMQ Proxy - Comprehensive Guide for AI Agents

## Project Overview

**BullMQ Proxy** is a lightweight HTTP-based proxy service for [BullMQ](https://bullmq.io/), enabling interaction with Redis-backed job queues through RESTful HTTP and WebSocket APIs. Built with [Bun.js](https://bun.sh/), it provides a language-agnostic interface to BullMQ's powerful job queue capabilities.

### Core Purpose

- **Language Agnostic**: Work with BullMQ from any language that supports HTTP
- **Serverless Support**: Run workers in serverless environments without direct Redis connections
- **Redis Isolation**: Protect Redis instances from untrusted sources
- **Access Control**: Token-based authentication for queue operations
- **Scalability**: Deploy multiple proxy instances for redundancy and load distribution

### Repository Information

- **Repository**: https://github.com/taskforcesh/bullmq-proxy
- **License**: MIT
- **Author**: Manuel Astudillo (Taskforce.sh Inc.)
- **Current Version**: 1.5.3
- **Runtime**: Bun.js
- **Primary Language**: TypeScript (91.8%)

## Architecture

### Technology Stack

- **Runtime**: Bun.js (high-performance JavaScript runtime)
- **Queue System**: BullMQ v5.41.0+
- **Database**: Redis/DragonflyDB (via ioredis)
- **Protocols**: HTTP/REST and WebSockets
- **Validation**: TypeBox (@sinclair/typebox)
- **Language**: TypeScript

### System Components

```
┌─────────────┐
│   Clients   │ (HTTP/WebSocket)
└──────┬──────┘
       │
┌──────▼──────────────┐
│  BullMQ Proxy       │
│  ┌───────────────┐  │
│  │ HTTP Routes   │  │
│  │ - Queues      │  │
│  │ - Workers     │  │
│  │ - Jobs        │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ WebSocket     │  │
│  │ - Queue Ops   │  │
│  │ - Job Process │  │
│  │ - Events      │  │
│  └───────────────┘  │
└──────┬──────────────┘
       │
┌──────▼──────┐
│    Redis    │
└─────────────┘
```

### Core Modules

#### 1. **Proxy Core** (`src/proxy.ts`, `src/index.ts`)
- Server initialization with Bun.serve
- Redis connection management (separate connections for queues and workers)
- WebSocket upgrade handling
- Graceful shutdown with cleanup

#### 2. **HTTP Controllers**

**Queue Controller** (`src/controllers/http/queue-http-controller.ts`)
- `POST /queues/:queueName/jobs` - Add jobs (bulk)
- `GET /queues/:queueName/jobs` - Get jobs with pagination and status filtering
- `GET /queues/:queueName/jobs/:jobId` - Get single job

**Worker Controller** (`src/controllers/http/worker-http-controller.ts`)
- `POST /workers` - Register worker with endpoint metadata
- `GET /workers` - List all registered workers
- `DELETE /workers/:queueName` - Remove worker
- Worker stream listener for real-time worker updates
- Lua scripts for atomic Redis operations

**Job Controller** (`src/controllers/http/worker-job-http-controller.ts`)
- `POST /queues/:queueName/jobs/:jobId/progress` - Update job progress
- `POST /queues/:queueName/jobs/:jobId/logs` - Add job logs
- `GET /queues/:queueName/jobs/:jobId/logs` - Get job logs

#### 3. **WebSocket Controllers**

**Queue WebSocket** (`src/controllers/websocket/queue.ts`)
- Endpoint: `ws://host/ws/queues/:queueName`
- Direct BullMQ Queue method invocation via message passing
- Bidirectional communication with message broker

**Worker WebSocket** (`src/controllers/websocket/worker.ts`)
- Endpoint: `ws://host/ws/queues/:queueName/process/:concurrency`
- Job processing with configurable concurrency
- Real-time job data streaming to client

**Queue Events WebSocket** (`src/controllers/websocket/queue-events.ts`)
- Endpoint: `ws://host/ws/queues/:queueName/events`
- Subscribe to queue events (completed, failed, progress, etc.)

#### 4. **Message Broker** (`src/controllers/websocket/message-broker.ts`)
- Custom implementation for non-acknowledged protocols
- Request-response pattern over WebSockets
- Timeout management (15s default)
- Promise-based API

#### 5. **Authentication**

**Token-Based Auth** (`src/authware/auth-by-tokens.ts`)
- Bearer token authentication
- Configured via `AUTH_TOKENS` environment variable
- Applied to queue and worker operations

**Worker Job Auth** (`src/authware/auth-for-workers.ts`)
- Token validation against job lock keys
- Ensures only the processing worker can modify job state
- Prevents unauthorized job manipulation

#### 6. **Validation** (`src/validators/`)
- Queue name validation (length, characters)
- Job schema validation
- Worker metadata validation (endpoint, options)
- Repeat options validation
- Deduplication options validation

#### 7. **Utilities**

**Queue Factory** (`src/utils/queue-factory.ts`)
- LRU cache for Queue instances (configurable size)
- Automatic queue instantiation and cleanup
- Connection reuse

**Route Matcher** (`src/utils/router-matcher.ts`)
- Path-based routing with parameter extraction
- HTTP method matching
- WebSocket upgrade support
- Authentication middleware integration

**Cache** (`src/cache.ts`)
- Generic LRU cache implementation
- Automatic eviction with cleanup callbacks
- Used for queue instance management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taskforcesh/bullmq-proxy](https://github.com/taskforcesh/bullmq-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
