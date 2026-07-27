---
trigger: always_on
description: A comprehensive Azure OpenAI chat application with advanced AI capabilities including Sora video generation, real-time voice chat, and multi-modal interactions. Built with Node.js backend and modular vanilla JavaScript frontend.
---

# Azure ChatGPT Demo - AI Coding Instructions

## Project Overview
A comprehensive Azure OpenAI chat application with advanced AI capabilities including Sora video generation, real-time voice chat, and multi-modal interactions. Built with Node.js backend and modular vanilla JavaScript frontend.

## Core Architecture Patterns

### Backend: Controller-Service Pattern
- **Controllers** (`/controllers/`): Handle HTTP requests, validation, and response formatting
- **Services** (`/services/`): Business logic and external API integrations
- **API Routes** (`apiRoutes.js`): Centralized route definitions with Azure AD auth middleware
- **Event-Driven Communication**: `eventbus.js` + `websocket.js` for real-time features

Example controller pattern:
```javascript
// controllers/soraController.js
class SoraController {
    constructor() {
        this.soraService = new SoraApiService();
        this.activeJobs = new Map(); // In-memory job tracking
    }
    async generateVideo(req, res) {
        const validation = this.soraService.validateParameters(params);
        if (!validation.isValid) {
            return res.status(400).json({ success: false, error: "Validation failed" });
        }
        const result = await this.soraService.generateVideo(params);
        // Job management and response handling
    }
}
```

### Frontend: Component-Based Architecture
- **UIManager** (`public/components/UIManager.js`): Central coordinator for all UI components
- **Managers**: Specialized components for different concerns (MessageManager, StorageManager, SyncManager)
- **Event-Driven**: EventManager handles all DOM interactions and component communication
- **Modular Design**: Components in `/public/components/` and `/public/modules/`

Key component initialization pattern:
```javascript
// public/components/UIManager.js
initializeManagers() {
    this.messageManager = new MessageManager(this);
    this.storageManager = new StorageManager(this);
    this.syncManager = new SyncManager(this);
    // Manager dependencies are injected via constructor
}
```

### New Model Integration & Feature Detection

#### GPT-5 Series Models
- **gpt-5, gpt-5-mini, gpt-5-nano**: Full feature support including:
  - ✅ Function calling & parallel tool calling
  - ✅ System messages & developer messages
  - ✅ Vision capabilities (text + image input)
  - ✅ Structured outputs & reasoning effort control
  - ✅ **NEW**: Verbosity control (`low`, `medium`, `high`)
  - ✅ **NEW**: Preamble support for function call planning
  - ✅ **NEW**: Minimal reasoning effort setting
  - ✅ **NEW**: Custom tool types for raw text outputs
  - ✅ **NEW**: Lark tool for Python lark grammar constraints
  - 📊 Context: 272K input, 128K output tokens

- **gpt-5-chat**: Limited preview model
  - ❌ No function calling or vision (text-only)
  - ✅ System messages & structured outputs
  - 📊 Context: 128K input, 16K output tokens

#### O3-PRO Advanced Reasoning
- ✅ Full function calling & parallel tool calling support
- ✅ Advanced reasoning with summary capabilities (limited access)
- ⚠️ Requires background mode to avoid timeouts
- ❌ No image generation capability
- 📊 Context: 200K input, 100K output tokens

#### Feature Detection Pattern
Use `supportsFeature(model, feature)` for capability checks:
```javascript
// Check model capabilities before API calls
if (supportsFeature(model, 'supportsFunctionCalling')) {
    // Add function tools to request
}
if (supportsFeature(model, 'supportsVerbosity')) {
    // Use GPT-5 verbosity control
}
if (supportsFeature(model, 'requiresBackgroundMode')) {
    // Use background mode for O3-PRO
}
```

## Essential Development Workflows

### Environment Setup
1. **Required .env variables**: GPT_4O_API_URL, SORA_API_URL, AZURE_STORAGE_CONNECTION_STRING
2. **New model support**: GPT-5 series (gpt-5, gpt-5-mini, gpt-5-nano, gpt-5-chat) and O3-PRO with advanced reasoning capabilities
3. **Development**: `npm run dev` (runs server + webpack watch)
4. **Production build**: `npm run build` then `npm start`
5. **Azure AD config**: Must manually update `utils/authConfig.js` (known CI/CD limitation)

### Message Flow Architecture
Messages follow a specific lifecycle:
1. **Input**: MessageManager.sendMessage() → MessageProcessor (factory pattern)
2. **Storage**: StorageManager (local) → SyncManager (Azure Table Storage)
3. **UI**: MessageUIHandler → DOMManager → EventManager (event listeners)
4. **State**: Active/inactive messages via conversation context system

### Real-time Communication
- **WebSocket** (`websocket.js`): Maintains userId-based client connections
- **EventBus** (`eventbus.js`): Server-side event coordination
- **Client events**: All handled through EventManager's delegation pattern

## Critical Integration Points

### Azure Storage Pattern
All data persistence follows this dual-storage pattern:
```javascript
// Local storage first, then sync to cloud
this.storageManager.saveMessage(chatId, message);
this.syncManager.syncMessageCreate(chatId, message);
```

### Sora Video Generation Workflow
1. **Job Creation**: SoraController.generateVideo() → SoraApiService

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hddevteam/Azure-chatGPT-demo](https://github.com/hddevteam/Azure-chatGPT-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
