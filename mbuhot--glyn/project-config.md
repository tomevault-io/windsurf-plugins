---
trigger: always_on
description: 1. [Core PubSub Concepts](#core-pubsub-concepts)
---

# CLAUDE.md - Glyn PubSub Library Development Guidelines

## Table of Contents
1. [Core PubSub Concepts](#core-pubsub-concepts)
2. [Critical Design Lessons](#critical-design-lessons)
3. [Actor Integration Patterns](#actor-integration-patterns)
4. [Distributed Clustering](#distributed-clustering)
5. [FFI and Syn Integration](#ffi-and-syn-integration)
6. [Type Safety Guidelines](#type-safety-guidelines)
7. [Testing PubSub Systems](#testing-pubsub-systems)
8. [API Design Principles](#api-design-principles)
9. [Common Pitfalls](#common-pitfalls)
10. [Project Workflow](#project-workflow)

## Core PubSub Concepts

### The Key Insight: Clean Message Composition
The primary value of glyn is enabling actors to handle **both direct commands AND PubSub events** cleanly:

```gleam
// Unified message type - the core pattern
pub type ActorMessage {
  DirectCommand(UserCommand)    // Direct actor messages
  PubSubEvent(SystemEvent)      // Events from PubSub
}

// Compose using selectors
let selector =
  process.new_selector()
  |> process.select_map(command_subject, DirectCommand)
  |> process.select_map(subscription.subject, PubSubEvent)
```

### Scope vs Type Safety
- **Scope**: Runtime namespace for syn (isolates different PubSub systems)
- **MessageType**: Compile-time type safety (ensures message type consistency across nodes)
- **NEVER reuse the same scope + MessageType for different message types**

## Critical Design Lessons

### 1. **The Distributed Reference Problem**
**Original Issue**: Each node creating unique references broke cross-node messaging.

**Wrong Approach**:
```gleam
// Each node gets different reference = broken clustering
PubSub(scope: scope_atom, tag: reference.new())
```

**Correct Approach**:
```gleam
// Deterministic tag from MessageType = clustering works
PubSub(scope: scope_atom, tag: phash2(message_type.id))
```

**Lesson**: **Always consider distributed scenarios** - what works locally might break in clusters.

### 2. **Type Safety Through Explicit Type IDs**
**Problem**: Generic parameters are error-prone and don't provide type safety.

**Solution**: MessageType pattern with compile-time safety:
```gleam
// Type-safe, self-documenting, prevents message type confusion
pub const user_event_type: glyn.MessageType(UserEvent) = glyn.MessageType("UserEvent_v1")
let pubsub = glyn.new_pubsub(scope: "user_events", message_type: user_event_type)
```

### 3. **True PubSub-Level Type Safety**
- **Syn handles**: Type-tagged group membership, message routing, distribution
- **Type system handles**: Compile-time safety, message structure
- **Groups are tagged**: Different MessageTypes use different syn groups `#(group, tag)`
- **No wasted delivery**: Messages only reach compatible subscribers

## Actor Integration Patterns

### The Selector Pattern
```gleam
pub fn start_actor() -> Result(actor.Started(Subject(Command)), actor.StartError) {
  actor.new_with_initialiser(5000, fn(_) {
    let subscription = glyn.subscribe(pubsub, "events", process.self())
    let command_subject = process.new_subject()

    let selector =
      process.new_selector()
      |> process.select_map(command_subject, DirectCommand)
      |> process.select_map(subscription.subject, PubSubEvent)

    actor.initialised(initial_state)
    |> actor.selecting(selector)
    |> actor.returning(command_subject)  // Return command interface
    |> Ok
  })
}
```

### Multi-System Integration
```gleam
// One actor, multiple PubSub systems
let user_sub = glyn.subscribe(user_pubsub, "business_logic", process.self())
let payment_sub = glyn.subscribe(payment_pubsub, "business_logic", process.self())

let selector =
  process.new_selector()
  |> process.select_map(command_subject, DirectCommand)
  |> process.select_map(user_sub.subject, UserEvent)
  |> process.select_map(payment_sub.subject, PaymentEvent)
```

## Distributed Clustering

### Node Naming Strategies
**Working Approaches**:
1. `gleam shell --name node1@localhost` (most reliable)
2. `ERL_FLAGS="-sname node1@localhost" gleam run -m module`
3. Startup scripts with proper environment variables

**Avoid**: Programmatic `net_kernel:start/2` - has compatibility issues.

### Clustering Rules
- **Same scope + type_id**: Nodes will share messages
- **Different scopes**: Complete isolation
- **Same scope, different type_ids**: Type-safe isolation

```gleam
// Node A and Node B both do this - they'll communicate
pub const order_event_type: glyn.MessageType(OrderEvent) = glyn.MessageType("OrderEvent_v1")
let pubsub = glyn.new_pubsub(scope: "global_events", message_type: order_event_type)
```

### Testing Distributed Behavior
```gleam
// Simulate distributed by creating multiple PubSub instances
pub const message_type: glyn.MessageType(Message) = glyn.MessageType("Message_v1")
let pubsub1 = glyn.new_pubsub(scope: "test_scope", message_type: message_type)  // "Node 1"
let pubsub2 = glyn.new_pubsub(scope: "test_scope", message_type: message_type)  // "Node 2"

let subscription = glyn.subscribe(pubsub1, "channel", process.self())
let reached = glyn.publish(pubsub2, "channel", "cross-node message")
// reached == 1 proves distributed behavior works
```

## FFI and Syn Integration

### Research First, Implement Second
**Critical**: Read the actual Erlang/OTP source before creating FFI bindings.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbuhot/glyn](https://github.com/mbuhot/glyn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
