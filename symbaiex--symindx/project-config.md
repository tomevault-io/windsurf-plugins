---
trigger: always_on
description: APPLY AI portal integration standards when working with AI provider code
---

globs: mind-agents/src/portals/**/*
alwaysApply: false
---
# AI Integration Patterns

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** AI provider integrations and portal management

## AI Portal Architecture

SYMindX implements a **unified portal architecture** that abstracts AI provider interactions through the Vercel AI SDK v5, enabling seamless switching between providers and models.

### Portal Abstraction Layer
```typescript
interface AIPortal {
  readonly id: string;
  readonly provider: string;
  readonly models: ModelInfo[];
  readonly capabilities: PortalCapability[];
  
  // Core operations
  generate(request: GenerationRequest): Promise<GenerationResponse>;
  stream(request: StreamRequest): AsyncIterable<StreamChunk>;
  embed(text: string, options?: EmbedOptions): Promise<number[]>;
  moderate(content: string): Promise<ModerationResult>;
  
  // Management
  initialize(config: PortalConfig): Promise<void>;
  healthCheck(): Promise<HealthStatus>;
  shutdown(): Promise<void>;
}

// Base portal implementation
abstract class BasePortal implements AIPortal {
  protected abstract createProvider(): Provider;
  protected abstract handleRateLimit(error: Error): Promise<void>;
  protected abstract selectModel(task: TaskType): string;
}
```

## Supported AI Providers

### OpenAI Portal (`portals/openai/`)
```typescript
interface OpenAIConfig {
  apiKey: string;
  organization?: string;
  baseURL?: string;
  models: {
    chat: string;          // Default: "gpt-4o"
    tools: string;         // Default: "gpt-4.1-mini"
    embedding: string;     // Default: "text-embedding-3-small"
  };
}

// Implementation with retries and fallbacks
class OpenAIPortal extends BasePortal {
  private readonly client: OpenAI;
  
  async generate(request: GenerationRequest): Promise<GenerationResponse> {
    return await this.withRetry(async () => {
      const result = await generateText({
        model: this.openai(this.selectModel('chat')),
        messages: request.messages,
        temperature: request.temperature,
        maxTokens: request.maxTokens,
        tools: request.tools
      });
      
      return this.transformResponse(result);
    });
  }
}
```

### Anthropic Portal (`portals/anthropic/`)
```typescript
interface AnthropicConfig {
  apiKey: string;
  baseURL?: string;
  models: {
    chat: string;          // Default: "claude-3-6-sonnet-20250101"
tools: string;         // Default: "claude-3-6-haiku-20250101"
  };
}

class AnthropicPortal extends BasePortal {
  private readonly client: Anthropic;
  
  // Handle Anthropic-specific message format
  protected transformMessages(messages: Message[]): AnthropicMessage[] {
    return messages.map(msg => ({
      role: msg.role === 'assistant' ? 'assistant' : 'user',
      content: msg.content
    }));
  }
}
```

### Groq Portal (`portals/groq/`)
```typescript
interface GroqConfig {
  apiKey: string;
  models: {
    chat: string;          // Default: "llama-3.3-70b-versatile"
    tools: string;         // Default: "llama-3.1-8b-instant"
  };
}

// Optimized for speed
class GroqPortal extends BasePortal {
  // Ultra-fast inference configuration
  protected getDefaultOptions(): GenerationOptions {
    return {
      temperature: 0.7,
      maxTokens: 2048,
      stream: true,           // Always stream for responsiveness
      parallel_tool_calls: true
    };
  }
}
```

### Google Vertex AI Portal (`portals/google-vertex/`)
```typescript
interface VertexConfig {
  projectId: string;
  location: string;
  credentialsPath?: string;
  models: {
    chat: string;          // Default: "gemini-1.5-pro"
    embedding: string;     // Default: "text-embedding-004"
  };
}

class GoogleVertexPortal extends BasePortal {
  // Handle Google's safety settings
  protected getSafetySettings(): SafetySetting[] {
    return [
      { category: 'HARM_CATEGORY_HARASSMENT', threshold: 'BLOCK_MEDIUM_AND_ABOVE' },
      { category: 'HARM_CATEGORY_HATE_SPEECH', threshold: 'BLOCK_MEDIUM_AND_ABOVE' }
    ];
  }
}
```

### Local AI Portals

#### Ollama Portal (`portals/ollama/`)
```typescript
interface OllamaConfig {
  baseURL: string;       // Default: "http://localhost:11434"
  models: {
    chat: string;        // e.g., "llama3.2:latest"
    embedding: string;   // e.g., "nomic-embed-text"
  };
}

class OllamaPortal extends BasePortal {
  // Check model availability
  async checkModelAvailability(model: string): Promise<boolean> {
    try {
      const response = await fetch(`${this.baseURL}/api/tags`);
      const { models } = await response.json();
      return models.some((m: any) => m.name === model);
    } catch {
      return false;
    }
  }
}
```

#### LM Studio Portal (`portals/lmstudio/`)
```typescript
interface LMStudioConfig {
  baseURL: string;       // Default: "http://localhost:1234"
  model: string;         // Currently loaded model
}

class LMStudioPortal extends BasePortal {
  // OpenAI-compatible API
  protected createProvider(): Provider {
    return openai({
      baseURL: this.config.baseURL + '/v1',
      apiKey: 'not-needed'  // LM Studio doesn't require API key
    });
  }
}
```

## Portal Integration Patterns

### Provider Selection Strategy
```typescript
interface ProviderSelector {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
