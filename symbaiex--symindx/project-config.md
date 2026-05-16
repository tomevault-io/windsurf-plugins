---
trigger: always_on
description: APPLY modern modular architecture patterns when developing core agent systems to ensure scalability and edge-first design
---

# Modern SYMindX Architecture Patterns 2025

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** System design, modular architecture, and edge computing

## System Architecture Overview

SYMindX follows a **modular, event-driven architecture** designed for scalability, hot-swappable components, and multi-platform agent deployment. The system is organized as a workspace with three main components:

```
┌─────────────────────────────────────────────────────────────┐
│                     SYMindX Workspace                        │
├─────────────────────────────────────────────────────────────┤
│  mind-agents/     │  website/        │  docs-site/          │
│  (Core Runtime)   │  (React UI)      │  (Documentation)     │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │  SYMindX Runtime  │
                    │   (Event-Driven)  │
                    └─────────┬─────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   ┌────▼────┐        ┌──────▼──────┐       ┌─────▼─────┐
   │ Module  │        │   Event     │       │ Extension │
   │Registry │◄──────►│    Bus      │◄─────►│  Loader   │
   └─────────┘        └─────────────┘       └───────────┘
```

## 2025 Architectural Principles

### 1. Edge-First Modular Design

- **Hot-swappable modules** for memory, emotion, and cognition
- **Edge-deployable components** for distributed processing
- **Plugin-based extensions** for platform integrations
- **Provider pattern** with edge-compatible AI service abstractions
- **Registry pattern** for component discovery and lifecycle
- **Micro-frontend integration** for distributed UI components

### 2. Event-Driven Architecture with Edge Distribution

- **Centralized Event Bus** for component communication
- **Edge-distributed event processing** for reduced latency
- **Pub/Sub pattern** with WebSocket and WebRTC support
- **Message-driven interactions** between agents and extensions
- **Real-time event processing** with streaming capabilities
- **Event sourcing** for distributed state management

### 3. Multi-Agent Coordination with Cloud-Edge Hybrid

- **Agent isolation** with shared resource management
- **Centralized coordination** through multi-agent manager
- **Edge deployment** for latency-sensitive operations
- **Resource pooling** for memory and AI portals
- **Conflict resolution** for concurrent operations
- **Auto-scaling** based on demand and resource availability

### 4. 2025 Performance Optimization Patterns

- **Progressive Web App (PWA)** architecture for offline capabilities
- **Service Worker** integration for background processing
- **WebAssembly (WASM)** modules for compute-intensive operations
- **HTTP/3 and QUIC** protocol support for faster communication
- **Streaming data processing** with backpressure handling
- **Adaptive resource allocation** based on device capabilities

## Module Architecture Patterns

### Memory Module Pattern
```typescript
interface MemoryProvider {
  readonly id: string;
  readonly type: 'sqlite' | 'supabase' | 'neon' | 'postgres' | 'memory';
  
  initialize(config: MemoryConfig): Promise<void>;
  store(conversation: Conversation): Promise<void>;
  retrieve(query: SearchQuery): Promise<Memory[]>;
  search(embedding: number[]): Promise<Memory[]>;
  shutdown(): Promise<void>;
}

// Hot-swappable implementation
abstract class BaseMemoryProvider implements MemoryProvider {
  protected abstract onHotReload(newConfig: MemoryConfig): Promise<void>;
  protected abstract validateConfig(config: MemoryConfig): boolean;
}
```

**Available Providers:**

- `sqlite/` - Local SQLite database with vector search
- `supabase/` - Supabase with pgvector for embeddings
- `neon/` - Neon database with vector capabilities
- `postgres/` - Direct PostgreSQL with pgvector
- `memory/` - In-memory storage (non-persistent)

### Emotion Module Pattern
```typescript
interface EmotionModule {
  readonly id: string;
  readonly emotions: EmotionType[];
  
  processEvent(event: EmotionalEvent): Promise<EmotionState>;
  getCurrentState(): EmotionState;
  getEmotionHistory(): EmotionHistory[];
  influenceResponse(response: string): string;
}

// Composite emotion system
class CompositeEmotion implements EmotionModule {
  private readonly emotions = [
    'happy', 'sad', 'angry', 'curious', 'confident', 
    'anxious', 'empathetic', 'nostalgic', 'proud', 
    'confused', 'neutral'
  ];
}
```

**11 Distinct Emotions** (RuneScape-inspired):

- Individual emotion modules in `emotion/{type}/`
- Composite emotion system combining multiple states
- Context-aware emotional transitions
- Emotion influence on response generation

### Cognition Module Pattern
```typescript
interface CognitionModule {
  readonly id: string;
  readonly type: 'htn_planner' | 'reactive' | 'hybrid';
  
  plan(goal: Goal, context: Context): Promise<Plan>;
  execute(plan: Plan): Promise<ActionResult>;
  adapt(feedback: Feedback): Promise<void>;
  reflect(outcome: Outcome): Promise<void>;
}
```

**Available Cognition Types:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
