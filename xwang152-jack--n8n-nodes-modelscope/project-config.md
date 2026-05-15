---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an n8n community node package that provides integration with ModelScope's API-Inference services. The package supports:
- Large Language Models (LLM) for chat completions
- Vision Models for image understanding
- Text-to-Image Models for image generation
- AI Agent/Chain integration via LangChain

## Architecture

### Dual Node Design

The package implements two complementary nodes:

1. **ModelScope** (`nodes/ModelScope/ModelScope.node.ts`): Traditional API node with resource/operation pattern
2. **ModelScopeChain** (`nodes/ModelScopeChain/ModelScopeChain.node.ts`): LangChain-based chat model for AI Agent/Chain workflows

### Resource/Operation Pattern

The main ModelScope node follows n8n's resource/operation architecture:
- **Resources**: LLM, Vision, Image (defined in `nodes/ModelScope/resources/{resource}/index.ts`)
- **Operations**: Each resource has specific operations (e.g., `chatCompletion`, `visionChat`, `textToImage`)
- **Fields**: Dynamic UI fields that appear based on selected resource/operation
- **Executors**: Implementation files named `{operation}.operation.ts` that execute the API calls

### Key Components

**API Client** (`nodes/ModelScope/utils/apiClient.ts`):
- Wraps OpenAI SDK for chat completions (ModelScope provides OpenAI-compatible API)
- Handles image generation via native fetch with async task polling
- Includes timeout support with AbortController
- Manages authentication via ModelScope access tokens

**Type Definitions** (`nodes/ModelScope/types/api.types.ts`):
- Complete TypeScript interfaces for all API requests and responses
- Error code enums and parameter types
- Ensures type safety throughout the codebase

**Operation Helper** (`nodes/ModelScope/utils/operationHelper.ts`):
- `getClientAndCredentials()` - Get authenticated API client
- `extractModelParams()` - Extract common model parameters
- `extractImageParams()`, `extractVisionParams()` - Resource-specific params
- `validateMessages()`, `validatePrompt()`, `validateImageUrl()` - Input validation

**Response Builder** (`nodes/ModelScope/utils/responseBuilder.ts`):
- `buildChatCompletionResponse()` - Standardized chat completion responses
- `buildVisionChatResponse()` - Vision chat responses with image metadata
- `buildImageResponse()` - Image generation responses
- `buildStreamResponse()`, `buildErrorResponse()` - Specialized responses

**Field Factory** (`nodes/ModelScope/utils/fieldFactory.ts`):
- Reusable UI field definition factories
- Eliminates duplication across resource index files
- Provides 15+ field creation methods

**Constants** (`nodes/ModelScope/utils/constants.ts`):
- `SUPPORTED_MODELS`: Centralized model lists for LLM, Vision, and Image
- `API_CONFIG`: API endpoint, headers, and timeouts
- `POLLING_CONFIG`: Polling intervals and retry settings
- `ERROR_MESSAGES`: Centralized error message constants
- `ModelType` enum for type-safe model type references

**Error Handler** (`nodes/ModelScope/utils/errorHandler.ts`):
- Centralized error handling with specific error code support
- `handleApiError()` - Handles OpenAI SDK errors
- `handleHttpResponse()` - Handles HTTP response errors
- HTTP status code handling (401, 429, 400, 500, etc.)

**Credentials** (`credentials/ModelScopeApi.credentials.ts`):
- Defines the API credential type for storing ModelScope access tokens
- Includes credential test endpoint

## Common Development Commands

```bash
# Install dependencies
npm install

# Build the package (TypeScript compilation + icon processing)
npm run build

# Watch mode for development
npm run dev

# Format code
npm run format

# Lint code
npm run lint

# Fix lint issues automatically
npm run lintfix

# Local testing with n8n
npm link
cd ~/.n8n/nodes
npm link n8n-nodes-modelscope
```

## Adding a New Model

To add support for a new model, update `nodes/ModelScope/utils/constants.ts`:

```typescript
export const SUPPORTED_MODELS = {
  [ModelType.LLM]: [
    'existing/model',
    'new/model-id',  // Add new model here
  ],
  // ... other model types
};
```

Models are automatically available in both the ModelScope node's dropdown and the ModelScopeChain node's search.

## Adding a New Resource/Operation

1. Create directory: `nodes/ModelScope/resources/{resourceName}/`
2. Create `index.ts` using FieldFactory for reusable UI components:
   ```typescript
   export const {resourceName}Operations: INodeProperties[] = [...];
   export const {resourceName}Fields: INodeProperties[] = [
     FieldFactory.createUsageNoticeField('info', {...}),
     FieldFactory.createModelField(ModelType.LLM, 'operationName'),
     // ... other fields
   ];
   ```
3. Create `{operationName}.operation.ts` using OperationHelper and ResponseBuilder
4. Import and add operations/fields to main node's `description.properties`
5. Add case in `execute()` switch statement

## Async Task Pattern (Image Generation)

Image generation uses async task submission with polling:

1. **Submit task**: `generateImage()` returns a `task_id`
2. **Poll status**: `getTaskStatus()` checks task status periodically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xwang152-jack/n8n-nodes-modelscope](https://github.com/xwang152-jack/n8n-nodes-modelscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
