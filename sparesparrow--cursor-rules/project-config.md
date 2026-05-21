---
trigger: always_on
description: Core implementation rules and guidelines for Model Context Protocol
---


# MCP Core Implementation Rules

## Client Implementation

### Client Configuration
```typescript
interface MCPClientConfig {
  name: string;                 // Client identifier
  version: string;             // Client version
  capabilities?: Capabilities; // Optional capabilities
  options?: {
    timeout?: number;         // Default request timeout
    retryPolicy?: RetryPolicy; // Retry configuration
    maxConcurrent?: number;   // Max concurrent requests
    logLevel?: LogLevel;      // Logging configuration
  };
}

interface RetryPolicy {
  maxAttempts: number;
  initialDelay: number;
  maxDelay: number;
  backoffFactor: number;
  jitter?: boolean;
}

enum LogLevel {
  DEBUG = 'debug',
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error'
}
```

### Client Implementation
```typescript
class MCPClient {
  private transport: Transport;
  private handlers: Map<string, Handler>;
  private capabilities: Capabilities;
  private requestQueue: RequestQueue;
  private logger: Logger;

  constructor(config: MCPClientConfig) {
    this.validateConfig(config);
    this.setupCapabilities(config.capabilities);
    this.initializeHandlers();
    this.setupRequestQueue(config.options);
    this.configureLogging(config.options?.logLevel);
  }

  private validateConfig(config: MCPClientConfig): void {
    if (!config.name || !config.version) {
      throw new Error('Client name and version are required');
    }
    if (config.capabilities) {
      this.validateCapabilities(config.capabilities);
    }
  }

  private setupRequestQueue(options?: ClientOptions): void {
    this.requestQueue = new RequestQueue({
      maxConcurrent: options?.maxConcurrent ?? 10,
      timeout: options?.timeout ?? 30000
    });
  }

  async request<T>(
    method: string,
    params?: unknown,
    options?: RequestOptions
  ): Promise<T> {
    const message = this.createRequestMessage(method, params);
    return this.requestQueue.enqueue(
      () => this.sendRequest<T>(message, options)
    );
  }

  async notify(
    method: string,
    params?: unknown
  ): Promise<void> {
    const message = this.createNotificationMessage(method, params);
    await this.transport.send(message);
  }
}
```

### Connection Management
```typescript
interface ConnectionOptions {
  timeout?: number;
  retryPolicy?: RetryPolicy;
  onError?: (error: Error) => void;
  onClose?: () => void;
  heartbeat?: {
    interval: number;
    timeout: number;
  };
}

class ConnectionManager {
  private retryCount: number = 0;
  private heartbeatTimer?: NodeJS.Timer;
  private reconnectTimer?: NodeJS.Timer;

  async connect(
    transport: Transport,
    options?: ConnectionOptions
  ): Promise<void> {
    try {
      await this.negotiateCapabilities();
      await this.initializeConnection();
      this.setupHeartbeat(options?.heartbeat);
      this.resetRetryCount();
    } catch (error) {
      await this.handleConnectionError(error, options);
    }
  }

  private async handleConnectionError(
    error: Error,
    options?: ConnectionOptions
  ): Promise<void> {
    this.logger.error('Connection error:', error);
    
    if (this.shouldRetry(options?.retryPolicy)) {
      await this.retryConnection(options);
    } else {
      this.handleFatalError(error);
    }
  }

  private setupHeartbeat(
    config?: { interval: number; timeout: number }
  ): void {
    if (!config) return;

    this.heartbeatTimer = setInterval(async () => {
      try {
        await this.sendHeartbeat();
      } catch (error) {
        this.handleHeartbeatFailure();
      }
    }, config.interval);
  }
}
```

## Message Handling

### Message Format
```typescript
interface Message {
  jsonrpc: "2.0";
  id?: string | number;
  method?: string;
  params?: unknown;
  result?: unknown;
  error?: {
    code: number;
    message: string;
    data?: unknown;
  };
}

class MessageValidator {
  static validate(message: unknown): message is Message {
    if (!this.hasJsonRpcVersion(message)) {
      return false;
    }

    if (this.isRequest(message)) {
      return this.validateRequest(message);
    }

    if (this.isResponse(message)) {
      return this.validateResponse(message);
    }

    if (this.isNotification(message)) {
      return this.validateNotification(message);
    }

    return false;
  }
}
```

### Message Processing
```typescript
class MessageProcessor {
  private handlers: Map<string, Handler>;
  private pendingRequests: Map<string | number, PendingRequest>;

  async processMessage(message: Message): Promise<void> {
    try {
      MessageValidator.validate(message);
      
      if (MessageValidator.isRequest(message)) {
        await this.processRequest(message);
      } else if (MessageValidator.isResponse(message)) {
        await this.processResponse(message);
      } else if (MessageValidator.isNotification(message)) {
        await this.processNotification(message);
      }
    } catch (error) {
      this.handleProcessingError(error, message);
    }
  }
}
```

## Error Handling

### Error Categories
```typescript
enum MCPErrorCode {
  // Protocol Errors (-32768 to -32000)
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603,

  // Implementation Errors (-32000 to -31000)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sparesparrow/cursor-rules](https://github.com/sparesparrow/cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
