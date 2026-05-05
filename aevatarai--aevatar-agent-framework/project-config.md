---
trigger: always_on
description: Aevatar Agent Framework is a distributed agent system built on Actor Model principles, designed for massive-scale agent interactions with event-driven architecture. The framework supports horizontal scaling through Orleans and ProtoActor runtimes while maintaining a clean abstraction layer.
---

# Aevatar Agent Framework Development Rules

## 🌟 Framework Overview

### Core Vision
Aevatar Agent Framework is a distributed agent system built on Actor Model principles, designed for massive-scale agent interactions with event-driven architecture. The framework supports horizontal scaling through Orleans and ProtoActor runtimes while maintaining a clean abstraction layer.

### Key Design Principles
1. **Event-Driven Architecture**: All agent communication happens through events
2. **Actor Model Encapsulation**: Agents run within actors for scalability
3. **Stream-Based Communication**: Each agent has a stream for event propagation
4. **Hierarchical Organization**: Parent-child relationships enable group coordination
5. **Runtime Agnostic**: Same agent code works across Local, Orleans, and ProtoActor

## 🚫 Port Policy

- **禁止使用 5000 端口**：仓库内任何服务/示例/文档/默认配置都不要绑定或示例化 `:5000`（避免冲突与误导）。如需本地默认端口，优先使用 `:5678`（例如 sidecar）。

## 📦 Core Concepts

### GAgent (智能体 / Agent)
- **Definition**: The core business logic unit that processes events and maintains state
- **Base Classes**:
  - `GAgentBase<TState>`: Basic agent with state management
  - `GAgentBase<TState, TEvent>`: Agent with event type filtering
  - `GAgentBase<TState, TEvent, TConfiguration>`: Configurable agent
- **Responsibilities**:
  - Define event handlers using `[EventHandler]` attributes
  - Maintain internal state (must be Protobuf)
  - Publish events through `PublishAsync()`
  - Process incoming events automatically

### GAgentActor (Actor包装器 / Actor Wrapper)
- **Definition**: Actor model wrapper that enables agents to run in distributed environments
- **Implementations**:
  - `LocalGAgentActor`: In-memory execution
  - `OrleansGAgentGrain`: Orleans runtime
  - `ProtoActorGAgentActor`: ProtoActor runtime
- **Responsibilities**:
  - Manage agent lifecycle
  - Provide stream infrastructure
  - Handle event routing and propagation
  - Maintain parent-child relationships
  - Enable horizontal scaling

### Relationship: GAgent ↔ GAgentActor
```
┌─────────────────────────────────────┐
│         GAgentActor (Actor)         │
│  ┌─────────────────────────────┐    │
│  │      GAgent (Business)       │   │
│  │  - Event Handlers            │   │
│  │  - State Management          │   │
│  │  - Business Logic            │   │
│  └─────────────────────────────┘    │
│  - Stream Management                 │
│  - Event Routing                     │
│  - Parent-Child Relations            │
│  - Distribution Support              │
└─────────────────────────────────────┘
```

## 🔴 CRITICAL: Protobuf Serialization Requirements

### MANDATORY: All Serializable Types MUST Use Protocol Buffers

**This is a non-negotiable framework requirement.** Any type that needs to be serialized or transmitted across runtime boundaries MUST be defined in a `.proto` file.

#### Types That MUST Use Protobuf:

1. **Agent State Types** (`TState` in `GAgentBase<TState>` or `GAgentBase<TState, TEvent>`)
   - ❌ NEVER manually define state classes in C#
   - ✅ ALWAYS define states in `.proto` files
   - Example:
     ```protobuf
     message MyAgentState {
         string id = 1;
         int32 count = 2;
         google.protobuf.Timestamp last_update = 3;
     }
     ```

2. **Event Messages** (Any message sent through the streaming system)
   - ❌ NEVER implement `IMessage` interface manually in C#
   - ✅ ALWAYS define events in `.proto` files
   - Example:
     ```protobuf
     message TaskAssignedEvent {
         string task_id = 1;
         string assigned_to = 2;
         string description = 3;
     }
     ```

3. **Event Sourcing Events** (State change events for persistence)
   - Must be Protobuf messages for version compatibility
   - Ensures events can be replayed after schema changes

#### Why This Is Critical:

1. **Orleans Streaming**: Uses `byte[]` for message transmission
2. **Cross-Runtime Compatibility**: Ensures messages work across Local, Orleans, and ProtoActor
3. **Version Compatibility**: Protobuf provides forward/backward compatibility
4. **Performance**: Efficient binary serialization
5. **Type Safety**: Generated code prevents serialization errors

## 📝 Event Handler Development

### Writing Event Handlers

Event handlers are methods that process incoming events. The framework automatically discovers and invokes them based on attributes and conventions.

#### 1. Specific Event Handler
```csharp
[EventHandler]
public async Task HandleTaskAssigned(TaskAssignedEvent evt)
{
    // Process specific event type
    State.MyTasks.Add(evt.TaskId);
    await Task.CompletedTask;
}
```

#### 2. All Events Handler
```csharp
[AllEventHandler]
public async Task HandleAnyEvent(EventEnvelope envelope)
{
    // Process any event with full envelope context
    Logger.LogInformation("Received event: {EventId}", envelope.Id);
    await Task.CompletedTask;
}
```

#### 3. Convention-Based Handler
```csharp
// No attribute needed if method name is HandleAsync or HandleEventAsync
public async Task HandleAsync(MyCustomEvent evt)
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aevatarAI/aevatar-agent-framework](https://github.com/aevatarAI/aevatar-agent-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
