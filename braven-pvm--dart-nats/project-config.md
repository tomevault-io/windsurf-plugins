---
trigger: always_on
description: **Last Updated**: February 23, 2026
---

# nats_dart Project Constitution

**Last Updated**: February 23, 2026  
**Applies To**: All development in `nats_dart` package

---

## 1. Core Principles (Non-Negotiable)

### Pure Dart
- **Every line of code must be valid Dart.** No platform-specific logic except in transport implementations.
- Platform differences are handled **exclusively** via conditional imports in `transport_factory.dart` 
- All protocol logic, parsing, JetStream, and KV APIs are **100% identical** across all platforms
- Test all protocol code without native dependencies (tests use stubs, not platform code)

### Test-Driven Design (TDD)
- **Tests exist BEFORE implementation**. Add test skeletons (`test.skip()`) before the implementation
- Every public method has at least one test
- Parser, encoder, NUID generator, and protocol logic are unit-tested in isolation
- Integration tests exercise real server connections (Docker NATS)
- Minimum coverage target: 80% for core protocol; 60% for platform-specific code

### SOLID Principles
- **Single Responsibility**: Each class has one reason to change
- **Open/Closed**: Extend via subclassing/interfaces, not modification
- **Liskov Substitution**: Implementations are true to their contracts
- **Interface Segregation**: No client forced to depend on methods it doesn't use
- **Dependency Inversion**: Depend on abstractions, not concretions

---

## 2. Code Organization & Patterns

### Directory Layout
```
lib/src/
├── transport/           # I/O abstraction (TCP, WebSocket)
│   ├── transport.dart           # Abstract interface
│   ├── transport_factory.dart   # Conditional import selector
│   ├── tcp_transport.dart       # dart:io implementation
│   └── websocket_transport.dart # web_socket_channel implementation
├── protocol/            # Wire protocol (codec, parsing)
│   ├── parser.dart      # Stateful MSG/HMSG/INFO parser
│   ├── encoder.dart     # HPUB/PUB/SUB/CONNECT encoder
│   ├── message.dart     # Protocol message model
│   └── nuid.dart        # Unique ID generator
├── client/              # High-level NATS API
│   ├── connection.dart  # NatsConnection (pub/sub, request/reply)
│   ├── subscription.dart
│   └── options.dart     # ConnectOptions & enums
├── jetstream/           # JetStream-specific APIs
│   ├── jetstream.dart
│   ├── stream_manager.dart
│   ├── consumer_manager.dart
│   ├── pull_consumer.dart
│   └── js_msg.dart
└── kv/                  # KeyValue store layer
    ├── kv.dart
    └── kv_entry.dart
```

### Abstraction Strategy
- **Transport** is the only location where platform logic lives
- All other code uses the `Transport` interface—no imports of `dart:io` elsewhere
- Conditional imports (`if (dart.library.io)` / `if (dart.library.html)`) **only** in `transport_factory.dart`
- Protocol parser/encoder work identically whether reading from TCP or WebSocket

### SOLID Application

#### S — Single Responsibility
```dart
// ✅ Good: Parser only parses
class NatsParser {
  Stream<NatsMessage> get messages;
  void addBytes(Uint8List data);
}

// ❌ Bad: Parser does auth AND parsing
class NatsParser {
  Future<void> authenticate() { /* ... */ }
  void parse() { /* ... */ }
}
```

#### O — Open/Closed
```dart
// ✅ Good: Extend Transport, don't modify it
abstract class Transport { /* interface */ }
class TcpTransport implements Transport { /* implementation */ }
class WebSocketTransport implements Transport { /* implementation */ }

// ❌ Bad: Add platform code directly to Transport
class Transport {
  if (kIsWeb) { /* WebSocket code */ }
  if (!kIsWeb) { /* TCP code */ }
}
```

#### L — Liskov Substitution
```dart
// ✅ Good: All implementations can replace Transport
NatsConnection(Transport transport) {
  // Works with TcpTransport, WebSocketTransport, or mock
}

// ❌ Bad: Implementation violates interface contract
class TcpTransport implements Transport {
  @override
  Future<void> write(Uint8List data) {
    throw UnimplementedError(); // Violates contract!
  }
}
```

#### I — Interface Segregation
```dart
// ✅ Good: Subscription interface only has what's needed
abstract class Subscription {
  Stream<NatsMessage> get messages;
}

// ❌ Bad: Forces implementation of unrelated methods
abstract class Subscription {
  Future<void> reconnect(); // Not all subscriptions need this
}
```

#### D — Dependency Inversion
```dart
// ✅ Good: Connection depends on Transport abstraction
class NatsConnection {
  final Transport _transport;
  NatsConnection(this._transport);
}

// ❌ Bad: Connection depends on concrete TCP implementation
class NatsConnection {
  final TcpTransport _transport = TcpTransport();
}
```

---

## 3. Test Requirements

### Unit Tests (before implementation)
```dart
// test/unit/parser_test.dart
void main() {
  test('parse MSG without reply-to', () {
    final parser = NatsParser();
    parser.addBytes(Uint8List.fromList('MSG subject 1 5\r\nHello\r\n'.codeUnits));
    final msg = parser.messages.first;
    
    expect(msg.subject, 'subject');
    expect(msg.payload, [72, 101, 108, 108, 111]); // 'Hello'
  });

  test('parse HMSG with headers and status code', () { /* ... */ });
  test('handle partial frames', () { /* ... */ });
  test('throw on invalid format', () { /* ... */ });
}
```

### Integration Tests (run against Docker NATS)
```dart
// test/integration/connection_test.dart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/braven-pvm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
