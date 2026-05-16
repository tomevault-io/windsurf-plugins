---
trigger: always_on
description: APPLY data management best practices when working with memory providers
---


# Data Management and Database Patterns

## Data Architecture Overview

SYMindX implements a multi-provider data storage architecture supporting SQLite, PostgreSQL, Supabase, and Neon with vector embeddings, conversation persistence, and real-time synchronization across multiple agent instances.

### Core Data Principles

**🗃️ Provider Abstraction**

- Unified interface across all memory providers
- Hot-swappable database backends
- Provider-specific optimization strategies

**📊 Vector-First Design**

- All text data stored with semantic embeddings
- Hybrid search capabilities (text + semantic)
- Efficient similarity search and clustering

**🔄 Real-time Synchronization**

- Multi-agent conversation synchronization
- Event-driven data updates
- Conflict resolution for concurrent modifications

## Database Schema Design

### Core Agent Tables

```sql
-- Agent registry and metadata
CREATE TABLE agents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  character_id VARCHAR(100) NOT NULL,
  status VARCHAR(50) DEFAULT 'inactive',
  config JSONB NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_active TIMESTAMP WITH TIME ZONE
);

-- Character definitions and personalities
CREATE TABLE characters (
  id VARCHAR(100) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  personality_traits JSONB NOT NULL,
  behavioral_patterns JSONB DEFAULT '{}',
  voice_settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Platform integrations and sessions
CREATE TABLE platform_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  platform_user_id VARCHAR(255),
  session_data JSONB NOT NULL,
  is_active BOOLEAN DEFAULT true,
  expires_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(agent_id, platform, platform_user_id)
);
```

### Conversation and Memory Tables

```sql
-- Conversation threads
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,
  platform_conversation_id VARCHAR(255),
  title VARCHAR(255),
  participants JSONB NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(agent_id, platform, platform_conversation_id)
);

-- Individual messages with vector embeddings
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  sender_id VARCHAR(255) NOT NULL,
  sender_name VARCHAR(255),
  content TEXT NOT NULL,
  content_embedding vector(1536), -- OpenAI embedding dimension
  message_type VARCHAR(50) DEFAULT 'text',
  platform_message_id VARCHAR(255),
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Vector similarity search index
  INDEX USING ivfflat (content_embedding vector_cosine_ops) WITH (lists = 100)
);

-- Memory fragments for long-term storage
CREATE TABLE memory_fragments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  content_embedding vector(1536),
  fragment_type VARCHAR(50) NOT NULL, -- 'conversation', 'knowledge', 'preference', 'experience'
  importance_score FLOAT DEFAULT 0.5,
  access_count INTEGER DEFAULT 0,
  last_accessed TIMESTAMP WITH TIME ZONE,
  source_conversation_id UUID REFERENCES conversations(id) ON DELETE SET NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX USING ivfflat (content_embedding vector_cosine_ops) WITH (lists = 100)
);
```

### Emotion and Cognition Data

```sql
-- Emotional state tracking
CREATE TABLE emotion_states (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  emotion_type VARCHAR(50) NOT NULL,
  intensity FLOAT NOT NULL CHECK (intensity >= 0 AND intensity <= 1),
  triggers JSONB DEFAULT '[]',
  context JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  expires_at TIMESTAMP WITH TIME ZONE
);

-- Cognitive state and decision history
CREATE TABLE cognition_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
  event_type VARCHAR(50) NOT NULL, -- 'decision', 'planning', 'reflection'
  cognitive_module VARCHAR(50) NOT NULL,
  input_data JSONB NOT NULL,
  output_data JSONB NOT NULL,
  processing_time_ms INTEGER,
  success BOOLEAN DEFAULT true,
  error_message TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

## Data Provider Implementations

### Base Provider Interface

```typescript
interface DataProvider {
  // Connection management
  connect(): Promise<void>;
  disconnect(): Promise<void>;
  healthCheck(): Promise<boolean>;
  
  // Agent management
  createAgent(agent: AgentData): Promise<Agent>;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
