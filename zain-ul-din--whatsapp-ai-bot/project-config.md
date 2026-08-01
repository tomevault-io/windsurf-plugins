---
trigger: always_on
description: WhatsApp AI Bot is a TypeScript-based chatbot that integrates multiple AI models with WhatsApp using the Baileys library. The bot supports text-to-text models (ChatGPT, Gemini, Ollama) and text-to-image models (DALL-E, Flux, Stability AI), with a flexible custom model system.
---

# CLAUDE.md - AI Assistant Development Guide

## Project Overview

WhatsApp AI Bot is a TypeScript-based chatbot that integrates multiple AI models with WhatsApp using the Baileys library. The bot supports text-to-text models (ChatGPT, Gemini, Ollama) and text-to-image models (DALL-E, Flux, Stability AI), with a flexible custom model system.

**Key Technologies:**
- TypeScript (strict mode)
- Baileys (WhatsApp Web API)
- Multiple AI Provider SDKs (OpenAI, Google Generative AI, Stability AI, etc.)
- MongoDB (optional session storage)
- vite-node (development runtime)

## Repository Structure

```
whatsapp-ai-bot/
├── src/
│   ├── index.ts                    # Entry point
│   ├── whatsapp-ai.config.ts       # Bot configuration
│   ├── baileys/                    # WhatsApp integration layer
│   │   ├── index.ts                # Connection setup
│   │   ├── env.ts                  # Environment variables
│   │   ├── handlers/
│   │   │   ├── messages.ts         # Message batch handler
│   │   │   └── message.ts          # Individual message handler
│   │   ├── hooks/
│   │   │   └── useMessageParser.ts # Message metadata parser
│   │   └── database/
│   │       └── mongo.ts            # MongoDB connection
│   ├── models/                     # AI model implementations
│   │   ├── BaseAiModel.ts          # Abstract base class
│   │   ├── GeminiModel.ts          # Google Gemini
│   │   ├── OpenAIModel.ts          # ChatGPT & DALL-E
│   │   ├── StabilityModel.ts       # Stability AI
│   │   ├── FluxModel.ts            # Hugging Face Flux
│   │   ├── OllamaModel.ts          # Ollama
│   │   └── CustomModel.ts          # Context-aware custom models
│   ├── types/                      # TypeScript type definitions
│   │   ├── AiModels.d.ts          # Model type unions
│   │   └── Config.d.ts            # Configuration interfaces
│   └── util/                       # Utility functions
│       ├── Util.ts                 # Prefix matching, file reading
│       └── MessageTemplates.ts     # Response templates
├── docs/                           # Documentation
├── .env.example                    # Environment template
├── package.json                    # Dependencies & scripts
└── tsconfig.json                   # TypeScript config (strict mode)
```

## Architecture

### Core Design Patterns

1. **Abstract Base Model Pattern**: All AI models extend `AIModel<AIArguments, CallBack>` abstract class (src/models/BaseAiModel.ts:63)
2. **Configuration-Driven**: Models are enabled/disabled via .env and configured in whatsapp-ai.config.ts
3. **Prefix-Based Routing**: Messages are routed to models based on prefix matching (!gemini, !chatgpt, etc.)
4. **Session Management**: Each user gets isolated conversation history per model
5. **Metadata-Rich Processing**: Messages are parsed into rich metadata objects before processing

### Message Flow

```
WhatsApp Message
    ↓
[baileys/index.ts] connectToWhatsApp() - Event listener setup
    ↓
[handlers/messages.ts] messagesHandler() - Batch processing
    ↓
[hooks/useMessageParser.ts] useMessageParser() - Extract metadata
    ↓
[handlers/message.ts] handleMessage() - Route to model
    ↓
[util/Util.ts] getModelByPrefix() - Match prefix to model
    ↓
[models/*Model.ts] sendMessage() - Generate response
    ↓
[baileys] client.sendMessage() - Send reply
```

## Configuration System

### Environment Variables (.env)

All models are **disabled by default** for security. Enable explicitly:

```bash
# Enable a model
GEMINI_ENABLED=True
GEMINI_PREFIX=!gemini
API_KEY_GEMINI=your_api_key_here

# Optional: Customize icon prefix
GEMINI_ICON_PREFIX=🔮
```

**Important ENV patterns:**
- Boolean values use string "True" (case-sensitive): `process.env.GEMINI_ENABLED === 'True'` (src/baileys/env.ts:83)
- All API keys are optional but required if model is enabled
- Processing message customizable via `PROCESSING` env var (src/baileys/env.ts:58)

### Model Configuration (whatsapp-ai.config.ts)

```typescript
const config: Config = {
  sendWelcomeMessage: true,
  models: {
    ChatGPT: { prefix: ENV.OPENAI_PREFIX, enable: ENV.OPENAI_ENABLED },
    Gemini: { prefix: ENV.GEMINI_PREFIX, enable: ENV.GEMINI_ENABLED },
    // ... other models
    Custom: [
      {
        modelName: 'whatsapp-ai-bot',
        prefix: '!wa',
        enable: true,
        context: './docs/wa-ai-bot.md', // Can be file path, URL, or text
        baseModel: 'Gemini' // or 'ChatGPT'
      }
    ]
  },
  prefix: {
    enabled: true, // If false, uses defaultModel for all messages
    defaultModel: 'ChatGPT'
  },
  sessionStorage: { enable: true, wwjsPath: './' },
  selfMessage: { skipPrefix: false }
};
```

## AI Model System

### BaseAiModel Abstract Class (src/models/BaseAiModel.ts)

All models must implement:

```typescript
abstract class AIModel<AIArguments, CallBack> {
  // Session management
  public sessionCreate(user: string): void
  public sessionRemove(user: string): void
  public sessionExists(user: string): boolean
  public sessionAddMessage(user: string, args: any): void

  // Required implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zain-ul-din/whatsapp-ai-bot](https://github.com/Zain-ul-din/whatsapp-ai-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
