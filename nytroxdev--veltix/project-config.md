---
trigger: always_on
description: Guidelines for AI coding agents working on the Veltix project.
---

# AGENTS.md - Veltix

Guidelines for AI coding agents working on the Veltix project.

## Project Overview

Veltix is a high-level TCP library for Python: sync, thread-friendly, zero dependencies.  
It handles framing, threading, handshake, routing, and reconnection.

- **Version:** 1.7.2
- **Python:** 3.8+
- **License:** MIT
- **Zero runtime dependencies:** pure stdlib only.

## Use Cases (When to Use Veltix)

Veltix is designed for projects that need **raw TCP communication** without the complexity of async frameworks or the
boilerplate of raw sockets.

**Perfect for:**

- **LAN tools:** file transfer, remote control, chat apps (e.g., [Nexo](https://github.com/NytroxDev/Nexo))
- **Multiplayer game servers:** real-time state sync, 64+ players at 64Hz tick rate
- **Real-time dashboards:** live data streaming between microservices
- **Custom protocols:** you control the message types, framing, and routing
- **IPC / inter-process communication:** lightweight进程间通信 on localhost
- **Remote tooling:** SSH-like command execution, remote file management
- **IoT / embedded:** minimal memory footprint (46 KB idle), no heavy dependencies

**Not ideal for:**

- **HTTP/REST APIs:** use Flask, FastAPI, or Django REST Framework
- **Browser clients:** Veltix uses raw TCP, not WebSocket; use websockets or Socket.IO for browser apps
- **Async-first codebases:** Veltix is sync by design; if your whole project is async, use `asyncio` directly
- **Very high throughput (>100k msg/s per connection):** consider a compiled language for the hot path

**Key differentiator:** Veltix gives you FastAPI-style routing (`@server.route(MY_TYPE)`) over raw TCP, with built-in
framing, handshake, ping/pong, and reconnection: zero dependencies, zero boilerplate.

## Performance

> Benchmarked on Python 3.14.5: 12-core CPU, 30.5 GB RAM, Linux (loopback). All numbers are 5-run averages.

| Metric                             | Threading    | Async            |
|------------------------------------|--------------|------------------|
| Idle server memory                 | 45.6 KB      | 4 KB             |
| Per client memory (avg)            | 36.08 KB     | 12.4 KB          |
| Average latency                    | 0.032 ms     | 0.035 ms         |
| Burst send                         | 52,109 msg/s | 52,296 msg/s     |
| Burst receive                      | 41,327 msg/s | 41,343 msg/s     |
| Concurrent stress (100 clients)    | 37,676 msg/s | **76,929 msg/s** |
| FPS simulation (64 players @ 64Hz) | 4,488 msg/s  | 4,488 msg/s      |

Async stress throughput is **2x higher** than Threading under high concurrency.

## Tech Stack

| Tool           | Purpose                 | Config                                        |
|----------------|-------------------------|-----------------------------------------------|
| setuptools     | Build/packaging         | `pyproject.toml`                              |
| pytest         | Testing                 | `[tool.pytest.ini_options]` in pyproject.toml |
| pytest-cov     | Code coverage           | `[tool.coverage.*]` in pyproject.toml         |
| pytest-asyncio | Async test support      |                                               |
| ruff           | Linting & formatting    | `[tool.ruff.*]` in pyproject.toml             |
| mypy           | Static type checking    | `[tool.mypy]` in pyproject.toml               |
| mkdocs         | Documentation           | `mkdocs.yml`                                  |
| mkdocstrings   | Auto-generated API docs | Google-style docstrings                       |

## Project Structure

```
veltix/
├── client/              # TCP client & reconnect
│   ├── client.py        # Client class
│   ├── config.py        # ClientConfig dataclass
│   ├── disconnect.py    # DisconnectState, DisconnectReason
│   └── reconnect_handler.py
├── server/              # TCP server
│   ├── server.py        # Server class
│   ├── config.py        # ServerConfig dataclass
│   └── client_info.py   # ClientInfo dataclass
├── network/             # Protocol layer
│   ├── request.py       # Request, Response, MAGIC, HEADER_SIZE
│   ├── sender.py        # Sender, Mode
│   ├── types.py         # MessageType, MessageTypeRegistry
│   ├── system_types.py  # PING, PONG, HELLO, HELLO_ACK
│   └── message_buffer.py
├── handler/             # Request routing & callbacks
│   ├── request_handler.py   # RequestHandler
│   ├── handshake_handler.py # HandshakeHandler
│   ├── callback_executor.py # CallbackExecutor (thread pool)
│   ├── rules.py             # PingRule, HelloRule, RouteRule, etc.
│   └── rules_manager.py     # RulesManager, MessageContext, Rule
├── socket_core/         # Swappable socket backends
│   ├── core.py          # SocketCore enum (THREADING, ASYNC)
│   ├── base_socket.py   # BaseSocket, SocketEvents
│   ├── threading_socket.py
│   ├── async_socket.py
│   └── managers/
│       └── clients_manager.py
├── internal/            # Internal helpers
│   ├── events.py        # Events enum
│   ├── buffer_size.py   # BufferSize enum
│   ├── compatibility.py # Version, COMPATIBILITY
│   ├── mode.py          # Mode enum
│   └── network.py
├── logger/              # Singleton logger (thread-safe, colorized, rotation)
│   ├── core.py          # Logger class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NytroxDev/Veltix](https://github.com/NytroxDev/Veltix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
