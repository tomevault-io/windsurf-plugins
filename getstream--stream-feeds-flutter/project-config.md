---
trigger: always_on
description: Stream Feeds Dart SDK development context and architecture
---


# Stream Feeds Dart SDK - Development Context

Understand the project architecture, technical stack, and development patterns for building the Stream Feeds Dart SDK.

## Project Overview

### Purpose and Target Users

This is a **production-grade pure Dart SDK** that implements the Stream Feeds API v3 for real-time social feeds and activity streams. It's designed as a professional solution for Dart applications, providing comprehensive feed management, real-time updates, and social interactions.

**Target Users**: Dart developers building social features, activity feeds, and real-time applications across any Dart environment.

### Core Features

- **📊 Feeds**: Activity feed creation, management, and real-time updates
- **🎯 Activities**: Rich activity objects with custom metadata and targeting  
- **👥 Social Features**: Reactions, comments, bookmarks, follows, polls, and user interactions
- **🔍 Queries**: Type-safe filtering, sorting, and pagination specifications
- **⚡ State**: Reactive state management with automatic change notifications
- **🌐 Real-time**: WebSocket-based live updates and event streaming

## Core Architecture

### SDK Structure
- **Public API**: Located in `lib/` root (main entry point: `stream_feeds.dart`)
- **Internal Implementation**: All implementation details in `lib/src/`
- **Pure Dart**: No platform dependencies, compatible with Dart VM, Flutter (N-2 version support), and Dart Web
- **Layered Architecture**: Clear separation between client, core, data, presentation, and state layers

### Design Principles

- **Pure Dart**: Compatible with all Dart environments without platform dependencies
- **Type Safety**: Leverage Dart's null safety and strong typing throughout
- **Reactive Architecture**: StateNotifier-based state management with real-time updates
- **Immutable Data**: @freezed models for predictable state management
- **Result Pattern**: Explicit error handling using Result types
- **Public API Focus**: Clean separation between public and internal APIs

## Technical Implementation

### Transport Layer
- **WebSocket**: Real-time updates and connection management
- **OpenAPI Client**: Generated HTTP client with dio for API communication

### Dependencies & Integration
```yaml
dependencies:
  freezed_annotation: ^0.15.0
  json_annotation: ^4.8.1
  state_notifier: ^1.0.0
  dio: ^5.3.2
  stream_core: ^0.1.0  # Core HTTP client
  barrel_files_annotation: ^0.1.1

dev_dependencies:
  freezed: ^2.4.6
  json_serializable: ^6.7.1
  build_runner: ^2.4.7
  barrel_files: ^0.1.1
```

### Data Flow Architecture
1. **Authentication**: API key and user token validation
2. **Connection**: WebSocket establishment for real-time updates  
3. **Data Access**: Repository pattern with Result-based error handling
4. **State Management**: StateNotifier-based reactive state objects with automatic notifications
5. **Real-time Events**: WebSocket event processing and state synchronization

## Development Patterns

### Code Organization Standards
- **Immutable Models**: Use @freezed for all data classes with required id fields
- **State Objects**: StateNotifier-based reactive patterns
- **Query Classes**: Separate query specifications from data models
- **Repository Classes**: Concrete repository classes with OpenAPI-generated API client
- **Data Mapping**: Use extension functions with `.toModel()` instead of mapper classes
- **Public API**: Use `@includeInBarrelFile` to mark classes for public export
- **Resource Management**: Proper disposal and cleanup patterns

### Quality Standards
- **Testing**: Comprehensive unit, integration, and performance tests using public API testing with HTTP interceptors
- **Documentation**: Full API documentation with practical examples  
- **Performance**: Optimized for memory usage and network efficiency
- **Compatibility**: Dart SDK N-2 version support, pure Dart implementation

### Mono Repo Management
- **Melos**: Use melos for mono repo management
- **Dependencies**: All shared dependencies must be defined in `melos.yaml`
- **Modifications**: Every dependency modification or addition goes through `melos.yaml`
- **Bootstrap**: Use `melos bootstrap` to sync dependencies across packages

## Performance Considerations

### Memory Management
- Efficient pagination and memory management (max 1000 activities)
- Automatic cleanup and disposal patterns

### Network Efficiency  
- Request batching and connection pooling (max 5 concurrent)
- Circuit breaker for unhealthy endpoints (5 failures trigger 30s circuit)
- WebSocket multiplexing for real-time features

### Monitoring & Telemetry
- Built-in metrics collection and health checks
- Optional usage analytics and crash reporting
- Comprehensive logging with adjustable levels
- Debugging support with performance profiling

## Integration Context

### Typical Usage Pattern
```dart
// Initialize once in main()
final client = StreamFeedsClient(
  apiKey: ApiKey('your-api-key'),
  user: User(id: 'user-123', extraData: {'name': 'John Doe'}),
  tokenProvider: UserTokenProvider.static('user-token'),
);

// Connect and create feeds
await client.connect();
final feed = client.feed(FeedId(group: 'user', id: 'john'));

// React to state changes (StateNotifier pattern)
feed.addListener(() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GetStream) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
