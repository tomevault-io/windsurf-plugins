---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**go-pubbing** is a production-grade, single-instance in-memory pub/sub system for Go with hierarchical topics, wildcard subscriptions, and optional statefulness. The motto is: **Excellence. Always.**

### Phase 1 Implementation Status ✅

**COMPLETED** - Phase 1 (Exact Topic Matching) - First dev cycle

All core infrastructure and fundamental features are implemented and tested:

#### ✅ Implemented Features
- **Broker**: Full lifecycle (New, Publish, Subscribe, Shutdown)
- **Topic System**: Hierarchical topics with exact matching (no wildcards yet)
- **Subscriptions**: Context-aware, handler-based, automatic cleanup
- **Messages**: Immutable with defensive copies, sequence numbers, timestamps, headers
- **Options**: Functional options for broker, publish, and subscribe
- **Logging**: Injectable logger interface with Zap adapter
- **Error Handling**: Rich error types with context
- **Testing**: 79 tests, 100% pass, race detection enabled
- **Performance**: 6M+ topic matches/sec, 157ns/op
- **Documentation**: Comprehensive README, 11 runnable examples

#### Test Coverage
- Main package: 67.0%
- Adapters: 100.0%
- Internal: 69.6%
- Topic system: 98.7%

#### ⏳ Planned for Future Phases
- **Phase 2**: Wildcard patterns (`*` and `>`)
- **Phase 3**: Message retention and replay
- **Phase 4**: Acknowledgment and delivery guarantees
- **Phase 5**: Queue groups and load balancing

### Core Philosophy
- **Single-threaded broker loop**: No locks, all mutable state owned by one goroutine ✅
- **Safety first**: No panics (except init), recover from user panics, no goroutine leaks ✅
- **Zero external dependencies**: Pure Go (except testing dependencies) ✅
- **Command pattern**: All state changes via commands sent to broker loop ✅
- **Future-proof**: Architecture supports multi-instance extension ✅

## Critical Style Requirements

**NO LITERAL STRINGS IN CODE** - Always use CONSTANTS for:
- Error messages
- Configuration keys
- Topic patterns
- Field names in maps/structs
- Any string that appears in code logic

Bad:
```go
if headers["content-type"] == "application/json" { ... }
```

Good:
```go
const (
    HeaderContentType = "content-type"
    MimeTypeJSON = "application/json"
)
if headers[HeaderContentType] == MimeTypeJSON { ... }
```

## Architecture Overview

### Threading Model
```
User Code → Broker API (thread-safe) → Command Channels → Central Broker Loop (single goroutine)
                                                           ↓
                                                    Topic Trie + Retention + Subscribers
                                                           ↓
                                              Subscriber Goroutines (one per subscription)
```

**Key principle**: Broker loop is single-threaded and owns ALL mutable state. No locks needed in loop. External API is thread-safe via channels.

### Core Components

1. **Broker** (`broker.go`, `broker_internal.go`)
   - Public API: `Publish()`, `Subscribe()`, `Request()`, `Shutdown()`
   - Internal loop: Processes commands sequentially
   - Uses channels for communication (cmdCh, stopCh, stoppedCh)

2. **Topic Trie** (`topic/trie.go`)
   - Efficient O(segments) wildcard matching
   - Supports `*` (single segment) and `>` (multi-level)
   - Pre-structured at subscribe time, not publish time
   - RWMutex because reads >> writes

3. **Retention Store** (`retention/store.go`, `retention/ringbuffer.go`)
   - Per-topic ring buffers
   - Lazy allocation (only when messages published)
   - Policy inheritance: specific > wildcard > global
   - Bounded memory by design

4. **Subscription** (`subscription.go`)
   - Each has own goroutine for isolation
   - Supports callback OR channel (not both)
   - Handles ack/nack, redelivery, panic recovery

5. **Message** (`message.go`)
   - ID: NanoID (not UUID - shorter)
   - Sequence: uint64 (global monotonic via atomic)
   - Payload: []byte (zero-copy)
   - Headers: map[string]string

6. **Commands** (`internal/command.go`)
   - publishCmd, subscribeCmd, unsubscribeCmd, ackCmd, shutdownCmd, statsCmd
   - Synchronous commands use result channels
   - Async commands (ack) are fire-and-forget

### Topic Rules
- Segments separated by `.` (dot)
- No empty segments: `mission..state` is INVALID
- No leading/trailing dots: `.mission` or `mission.` is INVALID
- `*` matches exactly one segment: `mission.*.change.*`
- `>` matches one or more segments, must be last: `mission.>`
- Only valid characters: `a-zA-Z0-9-_`

## Common Development Commands

### Setup
```bash
# Initialize module (if not done)
go mod init github.com/itsatony/go-pubbing

# Get dependencies (minimal - only test deps)
go mod tidy
```

### Testing
```bash
# Run all tests
go test ./...

# Run with race detector (ALWAYS use during development)
go test -race ./...

# Run with coverage
go test -cover ./...

# Run specific test
go test -run TestBrokerPublishSubscribe ./...

# Run benchmarks
go test -bench=. -benchmem ./...

# Run benchmarks for specific component
go test -bench=BenchmarkTopicMatching ./topic/
```

### Building
```bash
# Vet code
go vet ./...

# Format code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsatony) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
