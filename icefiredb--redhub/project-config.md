---
trigger: always_on
description: This file provides comprehensive guidance for AI coding agents working on the RedHub codebase.
---

# AGENTS.md

This file provides comprehensive guidance for AI coding agents working on the RedHub codebase.

## Project Overview

RedHub is a high-performance RESP (Redis Serialization Protocol) server framework built in Go. It uses the RawEpoll model via the gnet library to achieve ultra-high throughput with multi-threaded support while maintaining low CPU resource consumption.

**Key Features:**
- Ultra high performance (exceeds Redis single-threaded and multi-threaded implementations)
- Fully multi-threaded support using event loops
- Low CPU resource consumption
- Full Redis protocol (RESP) compatibility
- Supports RESP, Tile38, and Telnet protocols
- Create Redis-compatible servers with minimal code

## Architecture

### Core Design Pattern

RedHub implements an event-driven architecture using the gnet framework:

1. **Event Loops**: Multiple event loops run in parallel (multi-threaded mode)
2. **Connection Pool**: Each connection has an associated buffer for command accumulation
3. **Command Pipeline**: Supports pipelining of multiple commands in a single read
4. **Handler Callbacks**: Three primary handlers for application logic:
   - `onOpened`: Called when a new connection is established
   - `onClosed`: Called when a connection is closed
   - `handler`: Called for each parsed command

### Threading Model

- **Single-core mode**: All connections handled by a single event loop
- **Multi-core mode**: Multiple event loops distribute connections using load balancing
- **Connection Buffering**: Each connection maintains its own buffer and command queue
- **Thread Safety**: Uses RWMutex for connection map synchronization (see redhub.go:53)

## Project Structure

```
redhub/
├── redhub.go                    # Main framework core (RedHub server implementation)
├── redhub_test.go               # Core framework tests
├── go.mod                       # Go module definition
├── go.sum                       # Dependency checksums
├── pkg/
│   └── resp/
│       ├── resp.go              # RESP protocol serialization/deserialization
│       ├── comparse.go          # Command parsing logic
│       ├── resp_test.go         # RESP protocol tests
│       └── comparse_test.go     # Command parsing tests
└── example/
    └── memory_kv/
        └── server.go            # Example Redis-compatible server (SET/GET/DEL/PING/QUIT)
```

## Core Components

### 1. RedHub Server (`redhub.go`)

**Key Types:**
- `RedHub`: Main server structure (line 48-54)
- `Options`: Server configuration options (line 30-45)
- `Conn`: Connection wrapper around gnet.Conn (line 25-27)
- `Action`: Post-event action type (line 13-22)

**Action Values:**
- `None`: No action
- `Close`: Close the connection
- `Shutdown`: Shutdown the server

**Server Options:**
```go
type Options struct {
    Multicore        bool              // Enable multi-core support
    LockOSThread     bool              // Lock OS thread
    ReadBufferCap    int               // Read buffer capacity
    LB               gnet.LoadBalancing // Load balancing strategy
    NumEventLoop     int               // Number of event loops
    ReusePort        bool              // Enable port reuse
    Ticker           bool              // Enable ticker
    TCPKeepAlive     time.Duration     // TCP keep-alive interval
    TCPKeepCount     int               // TCP keep-alive count
    TCPKeepInterval  time.Duration     // TCP keep-alive interval
    TCPNoDelay       gnet.TCPSocketOpt // TCP no-delay option
    SocketRecvBuffer int               // Socket receive buffer
    SocketSendBuffer int               // Socket send buffer
    EdgeTriggeredIO  bool              // Edge-triggered I/O
}
```

**Main Functions:**
- `NewRedHub(onOpened, onClosed, handler)`: Create new RedHub instance (line 63-75)
- `ListenAndServe(addr, options, rh)`: Start the server (line 161-205)

**Event Handlers:**
- `OnBoot(eng)`: Called when engine is ready (line 78-80)
- `OnShutdown(eng)`: Called when engine is shutting down (line 83-84)
- `OnOpen(c)`: Called when new connection opens (line 87-93)
- `OnClose(c, err)`: Called when connection closes (line 96-101)
- `OnTraffic(c)`: Called when data is received (line 104-153)
- `OnTick()`: Called on timer ticks (line 156-158)

### 2. RESP Protocol Package (`pkg/resp/`)

#### resp.go - Protocol Serialization

**RESP Types:**
```go
const (
    Integer = ':'  // Integers (e.g., :1000\r\n)
    String  = '+'  // Simple strings (e.g., +OK\r\n)
    Bulk    = '$'  // Bulk strings (e.g., $6\r\nfoobar\r\n)
    Array   = '*'  // Arrays (e.g., *2\r\n$3\r\nGET\r\n$3\r\nkey\r\n)
    Error   = '-'  // Errors (e.g., -ERR unknown command\r\n)
)
```

**Key Functions:**

**Reading/Parsing:**
- `ReadNextRESP(b []byte) (n int, resp RESP)`: Parse next RESP value (line 45-134)
- `ReadNextCommand(packet, argsbuf)`: Parse next command (line 159-226)
- `ForEach(iter func(resp RESP) bool)`: Iterate over array elements (line 32-41)

**Writing/Serializing:**
- `AppendInt(b []byte, n int64)`: Append integer (line 378-380)
- `AppendString(b []byte, s string)`: Append simple string (line 402-406)
- `AppendBulk(b []byte, bulk []byte)`: Append bulk bytes (line 388-392)
- `AppendBulkString(b []byte, bulk string)`: Append bulk string (line 395-399)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IceFireDB/redhub](https://github.com/IceFireDB/redhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
