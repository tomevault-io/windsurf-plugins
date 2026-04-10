---
trigger: always_on
description: DDD-ECS Isomorphic Mapping
---

# DDD-ECS Isomorphic Mapping

## Overview

This document defines the mathematical, structure-preserving functor between Domain-Driven Design (DDD) and Entity Component System (ECS) in our architecture. We maintain a strict isomorphism that preserves the semantic meaning and relationships between these two paradigms.

## Core Isomorphism

```
DDD Category                    ECS Category
────────────────────────────────────────────
Entity/Aggregate       ←→       Entity
Value Object          ←→       Component
Command Handler       ←→       System (Write)
Query Handler         ←→       System (Read)
Domain Event          ←→       Event
Repository            ←→       Query<Components>
Domain Service        ←→       System (Pure)
Policy                ←→       System (Reactive)
```

## Detailed Mappings

### 1. Entities and Aggregates → ECS Entities

**DDD Entities/Aggregates** map to **ECS Entities** with identity and structure preserved:

```rust
// DDD Domain Model
pub struct GraphAggregate {
    pub id: GraphId,           // Identity
    pub nodes: HashMap<NodeId, Node>,
    pub edges: HashMap<EdgeId, Edge>,
}

// ECS Representation
#[derive(Component)]
pub struct GraphEntity {
    pub aggregate_id: GraphId,  // Same identity
}

// Spawning preserves identity
commands.spawn((
    GraphEntity { aggregate_id: graph_id },
    // Value objects as components...
));
```

**Key Principles:**
- One ECS Entity per DDD Aggregate Root
- Child entities within aggregates also become ECS Entities
- Identity is preserved through ID components
- Aggregate boundaries are maintained through entity relationships

### 2. Value Objects → Components

**DDD Value Objects** map directly to **ECS Components**:

```rust
// DDD Value Object
#[derive(Debug, Clone, PartialEq)]
pub struct Position3D {
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

// ECS Component (same type, just tagged)
#[derive(Component, Debug, Clone, PartialEq)]
pub struct Position3D {
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

// Usage
commands.spawn((
    NodeEntity { node_id },
    Position3D { x: 0.0, y: 0.0, z: 0.0 },  // Value object as component
    NodeContent { ... },                     // Another value object
));
```

**Key Principles:**
- Value objects ARE components (not just represented by them)
- Immutability is preserved - replace whole component, never mutate
- No identity - compared by value
- Can be freely copied between entities

### 3. Commands → Write Systems

**DDD Command Handlers** map to **ECS Systems** that process commands:

```rust
// DDD Command
pub struct MoveNode {
    pub graph_id: GraphId,
    pub node_id: NodeId,
    pub new_position: Position3D,
}

// ECS System (Command Handler)
fn handle_move_node_commands(
    mut commands: Commands,
    mut events: EventWriter<NodeEvent>,
    nodes: Query<(Entity, &NodeEntity, &Position3D)>,
    mut move_commands: EventReader<MoveNodeCommand>,
) {
    for cmd in move_commands.read() {
        // Find the entity
        if let Some((entity, node, old_pos)) = nodes.iter()
            .find(|(_, n, _)| n.node_id == cmd.node_id)
        {
            // Remove old position (value object)
            commands.entity(entity).remove::<Position3D>();

            // Add new position (value object)
            commands.entity(entity).insert(cmd.new_position);

            // Emit domain events
            events.send(NodeEvent::NodeRemoved { ... });
            events.send(NodeEvent::NodeAdded { ... });
        }
    }
}
```

### 4. Queries → Read Systems

**DDD Query Handlers** map to **ECS Systems** that read state:

```rust
// DDD Query
pub struct FindNodesInRadius {
    pub center: Position3D,
    pub radius: f32,
}

// ECS System (Query Handler)
fn find_nodes_in_radius(
    query: Query<(&NodeEntity, &Position3D)>,
    requests: EventReader<FindNodesInRadiusRequest>,
    mut results: EventWriter<QueryResult>,
) {
    for request in requests.read() {
        let nodes: Vec<NodeId> = query.iter()
            .filter(|(_, pos)| {
                let distance = calculate_distance(pos, &request.center);
                distance <= request.radius
            })
            .map(|(node, _)| node.node_id)
            .collect();

        results.send(QueryResult::NodesFound { nodes });
    }
}
```

### 5. Domain Events → ECS Events

**DDD Domain Events** map directly to **ECS Events**:

```rust
// DDD Domain Event
#[derive(Debug, Clone)]
pub struct NodeAdded {
    pub graph_id: GraphId,
    pub node_id: NodeId,
    pub position: Position3D,
}

// ECS Event (same type, implements Event)
#[derive(Event, Debug, Clone)]
pub struct NodeAdded {
    pub graph_id: GraphId,
    pub node_id: NodeId,
    pub position: Position3D,
}

// Event flow preserved
fn process_domain_events(
    mut events: EventReader<NodeAdded>,
    mut commands: Commands,
) {
    for event in events.read() {
        // React to domain events
        commands.spawn((
            NodeEntity { node_id: event.node_id },
            event.position.clone(),
        ));
    }
}
```

### 6. Repositories → Queries

**DDD Repositories** map to **ECS Queries**:

```rust
// DDD Repository Interface
trait NodeRepository {
    fn find_by_id(&self, id: NodeId) -> Option<Node>;
    fn find_by_type(&self, node_type: NodeType) -> Vec<Node>;
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheCowboyAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
