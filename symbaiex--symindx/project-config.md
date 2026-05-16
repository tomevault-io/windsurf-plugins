---
trigger: always_on
description: APPLY extension patterns when developing platform integrations
---

globs: mind-agents/src/extensions/**/*
alwaysApply: false
---
# Extension System Patterns

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** Platform integrations, extensions, and external service connections

## Extension Architecture Overview

SYMindX implements a **pluggable extension system** that enables seamless integration with multiple communication platforms, APIs, and external services through a unified interface architecture.

### Extension System Structure
```
┌─────────────────────────────────────────────────────────────┐
│                  SYMindX Extension System                    │
├─────────────────────────────────────────────────────────────┤
│  Communication Extensions  │  API Extensions  │  MCP       │
│  ├─ Telegram Bot           │  ├─ REST API     │  ├─ Server │
│  ├─ Discord Bot            │  ├─ WebSocket    │  ├─ Client │
│  ├─ Slack Integration      │  ├─ GraphQL      │  └─ Tools  │
│  ├─ Twitter/X Bot          │  └─ Webhooks     │            │
│  └─ RuneLite Plugin        │                  │            │
└─────────────────────────────────────────────────────────────┘
```

## Base Extension Framework

### Extension Interface
```typescript
interface Extension {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  readonly capabilities: ExtensionCapability[];
  readonly dependencies: ExtensionDependency[];
  
  // Lifecycle methods
  initialize(context: ExtensionContext): Promise<void>;
  activate(): Promise<void>;
  deactivate(): Promise<void>;
  dispose(): Promise<void>;
  
  // Event handling
  onMessage(message: Message): Promise<void>;
  onEvent(event: SystemEvent): Promise<void>;
  handleError(error: Error): Promise<void>;
}

abstract class BaseExtension implements Extension {
  protected context: ExtensionContext;
  protected eventBus: EventBus;
  protected logger: Logger;
  
  constructor(
    public readonly id: string,
    public readonly name: string,
    public readonly version: string
  ) {}
  
  async initialize(context: ExtensionContext): Promise<void> {
    this.context = context;
    this.eventBus = context.eventBus;
    this.logger = context.logger.child({ extension: this.id });
    
    await this.onInitialize();
  }
  
  protected abstract onInitialize(): Promise<void>;
  protected abstract onActivate(): Promise<void>;
  protected abstract onDeactivate(): Promise<void>;
}
```

### Extension Registry
```typescript
interface ExtensionRegistry {
  register(extension: Extension): Promise<void>;
  unregister(extensionId: string): Promise<void>;
  get(extensionId: string): Extension | null;
  getAll(): Extension[];
  getByCapability(capability: ExtensionCapability): Extension[];
}

class RuntimeExtensionRegistry implements ExtensionRegistry {
  private extensions = new Map<string, Extension>();
  private capabilityIndex = new Map<ExtensionCapability, Set<string>>();
  
  async register(extension: Extension): Promise<void> {
    // Validate dependencies
    await this.validateDependencies(extension);
    
    // Initialize extension
    await extension.initialize(this.createContext(extension));
    
    // Register in indexes
    this.extensions.set(extension.id, extension);
    this.indexCapabilities(extension);
    
    this.logger.info(`Extension registered: ${extension.id}`);
  }
  
  private createContext(extension: Extension): ExtensionContext {
    return {
      extensionId: extension.id,
      eventBus: this.eventBus,
      logger: this.logger,
      config: this.getExtensionConfig(extension.id),
      storage: this.createExtensionStorage(extension.id)
    };
  }
}
```

## Communication Extensions

### Telegram Extension (`extensions/communication/telegram/`)
```typescript
interface TelegramConfig {
  botToken: string;
  webhookUrl?: string;
  allowedUsers?: string[];
  commandPrefix: string;
  features: {
    inlineKeyboards: boolean;
    fileUploads: boolean;
    groupChats: boolean;
  };
}

class TelegramExtension extends BaseExtension {
  private bot: TelegramBot;
  private config: TelegramConfig;
  
  protected async onInitialize(): Promise<void> {
    this.config = this.context.config as TelegramConfig;
    this.bot = new TelegramBot(this.config.botToken, {
      polling: !this.config.webhookUrl,
      webhook: this.config.webhookUrl ? {
        url: this.config.webhookUrl,
        port: process.env.WEBHOOK_PORT ? parseInt(process.env.WEBHOOK_PORT) : 3000
      } : undefined
    });
    
    this.setupEventHandlers();
  }
  
  private setupEventHandlers(): void {
    this.bot.on('message', async (msg) => {
      try {
        await this.handleTelegramMessage(msg);
      } catch (error) {
        await this.handleError(error as Error);
      }
    });
    
    this.bot.on('callback_query', async (query) => {
      await this.handleCallbackQuery(query);
    });
  }
  
  private async handleTelegramMessage(msg: TelegramMessage): Promise<void> {
    // Security check
    if (!this.isAuthorizedUser(msg.from?.id)) {
      await this.bot.sendMessage(msg.chat.id, 'Unauthorized access');
      return;
    }
    
    // Convert to internal message format
    const message: Message = {
      id: msg.message_id.toString(),
      content: msg.text || '',
      sender: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
