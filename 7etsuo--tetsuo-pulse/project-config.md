---
trigger: always_on
description: This is a high-performance, exception-driven socket toolkit for POSIX systems written in C11. The library provides a clean, modern API for TCP, UDP, Unix domain sockets, HTTP/1.1, HTTP/2, QUIC, WebSocket, and TLS/DTLS with comprehensive error handling, zero-copy I/O, and cross-platform event polling.
---

# GitHub Copilot Instructions for Socket Library

## Project Overview

This is a high-performance, exception-driven socket toolkit for POSIX systems written in C11. The library provides a clean, modern API for TCP, UDP, Unix domain sockets, HTTP/1.1, HTTP/2, QUIC, WebSocket, and TLS/DTLS with comprehensive error handling, zero-copy I/O, and cross-platform event polling.

**Maturity Notice**: This library is functional and well-tested but newly released. It is appropriate for development, internal tooling, and controlled environments.

## Architecture and Design Principles

### Core Components

- **Core Infrastructure**: Exception handling (TRY/EXCEPT/FINALLY), arena memory management, circular buffer I/O
- **Networking Stack**: TCP/UDP sockets, Unix domain sockets, TLS/DTLS, proxy support (SOCKS4/5, HTTP CONNECT)
- **HTTP Protocol**: HTTP/1.1, HTTP/2, HPACK, QPACK, client/server implementations
- **QUIC Transport**: RFC 9000 compliant QUIC v1 with connection management, stream multiplexing, loss detection
- **WebSocket**: RFC 6455 compliant with permessage-deflate compression
- **DNS Resolution**: Async resolver with DNS-over-TLS, DNS-over-HTTPS, DNSSEC validation
- **Event System**: Cross-platform polling (epoll/kqueue/poll), async I/O (io_uring/kqueue AIO), timer management
- **Security**: SYN flood protection, rate limiting, request smuggling prevention

### Platform Support

- **Primary**: Linux (epoll, io_uring), BSD/macOS (kqueue)
- **Fallback**: poll(2) for other POSIX systems
- **NOT Windows-compatible** without significant adaptation layer

## Coding Standards

### Language and Compiler

- **C Standard**: C11 with GNU extensions (`-std=gnu11`)
- **Required Features**: POSIX threads (pthread), IPv6 kernel support
- **Compiler Flags**: `-Wall -Wextra -Werror -fno-delete-null-pointer-checks -D_GNU_SOURCE -pthread -fno-strict-aliasing -fPIC`

### Code Style

- **Formatting**: Use `.clang-format` configuration in repository root
- **Naming Conventions**:
  - Types: `PascalCase_T` suffix (e.g., `Socket_T`, `SocketHE_Config_T`)
  - Functions: `Module_functionName` (e.g., `Socket_new`, `Socket_connect`)
  - Constants: `UPPER_SNAKE_CASE` (e.g., `SOCKET_MAX_CONNECTIONS`)
  - Variables: `snake_case` or `camelCase` depending on context
- **Header Guards**: `MODULENAME_INCLUDED` pattern
- **File Headers**: Include MIT license header with Tetsuo AI copyright

### Error Handling

**Primary Pattern**: Exception-based error handling using `TRY/EXCEPT/FINALLY` macros

```c
TRY
    Socket_T server = Socket_new(AF_INET, SOCK_STREAM, 0);
    Socket_bind(server, NULL, 8080);
    Socket_listen(server, 128);
EXCEPT(Socket_Failed)
    fprintf(stderr, "Error: %s\n", Socket_GetLastError());
FINALLY
    Socket_free(&server);
END_TRY;
```

**Secondary Pattern**: Simple API with return codes for convenience (no TRY/EXCEPT needed)

```c
Socket_T sock = Socket_new_simple(AF_INET, SOCK_STREAM, 0);
if (!sock) {
    fprintf(stderr, "Failed to create socket\n");
    return -1;
}
```

**Important**: 
- Use exception handling for complex control flow
- Provide simple API alternatives where appropriate
- Always clean up resources in FINALLY blocks or with explicit `_free()` calls

### Memory Management

- **Arena Allocator**: Use `Arena_alloc()` for temporary allocations that share lifecycle
- **Manual Cleanup**: Use `_free()` functions (e.g., `Socket_free()`, `Arena_free()`)
- **No Dynamic Reallocation**: Avoid `realloc()` in hot paths; use fixed-size buffers or arenas
- **Overflow Protection**: Check bounds before allocation; arena allocator includes overflow detection

### Documentation

- **Doxygen**: All public APIs must have Doxygen comments
- **Format**:
  ```c
  /**
   * @brief Brief one-line description.
   * @ingroup group_name
   * @param[in] param1 Description of input parameter.
   * @param[out] param2 Description of output parameter.
   * @return Description of return value.
   * @threadsafe Yes/No.
   *
   * Detailed description with usage examples.
   *
   * ## Usage Example
   * @code{.c}
   * Socket_T sock = Socket_new(AF_INET, SOCK_STREAM, 0);
   * Socket_bind(sock, NULL, 8080);
   * @endcode
   */
  ```
- **README Updates**: Update feature lists when adding significant functionality
- **RFC References**: Include RFC numbers for protocol implementations (e.g., "RFC 9113" for HTTP/2)

## Build System

### CMake Configuration

- **Minimum Version**: CMake 3.10+
- **Build Types**: Debug (default), Release
- **Options**:
  - `ENABLE_TLS=ON/OFF` - Enable TLS/SSL support (requires OpenSSL/LibreSSL)
  - `ENABLE_SANITIZERS=ON/OFF` - Enable ASan + UBSan
  - `ENABLE_ASAN=ON/OFF` - Enable AddressSanitizer only
  - `ENABLE_UBSAN=ON/OFF` - Enable UndefinedBehaviorSanitizer only
  - `ENABLE_TSAN=ON/OFF` - Enable ThreadSanitizer (incompatible with ASan)
  - `ENABLE_COVERAGE=ON/OFF` - Enable code coverage with gcov
  - `ENABLE_FUZZING=ON/OFF` - Enable libFuzzer harnesses (requires Clang)
  - `ENABLE_IO_URING=ON/OFF` - Enable io_uring backend (Linux only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7etsuo/tetsuo-pulse](https://github.com/7etsuo/tetsuo-pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
