---
trigger: always_on
description: This is a **PHP clone of the Anthropic Python SDK** (https://github.com/anthropics/anthropic-sdk-python), designed as a universal, framework-agnostic PHP SDK following **PSR standards** for easy integration with Laravel, Symfony, Slim, and other frameworks.
---

# Copilot Instructions for Claude PHP SDK

This is a **PHP clone of the Anthropic Python SDK** (https://github.com/anthropics/anthropic-sdk-python), designed as a universal, framework-agnostic PHP SDK following **PSR standards** for easy integration with Laravel, Symfony, Slim, and other frameworks.

## Project Goals

- **API Parity**: Full implementation of latest Claude API endpoints (Messages, Files, Batches, Models APIs)
- **PSR Compliance**: Follow PHP Standards Recommendations (PSR-12 coding style, PSR-11 containers for DI)
- **Dependency Injection Ready**: Support framework-agnostic DI patterns for seamless integration
- **Latest Models & Features**: Support Claude Sonnet 4.5, Haiku 4.5, Opus 4.1, plus extended thinking, streaming, tool use, vision, embeddings
- **Async & Sync Support**: Both synchronous and asynchronous clients via Amphp/PHP async patterns
- **Python SDK Parity**: Mirror error handling, configuration, and patterns from the official Python SDK

## Architecture & Core Components

### Directory Structure (to be created)

```
Claude-PHP-SDK/
├── src/
│   ├── Anthropic.php           # Main client class (factory/entry point)
│   ├── Client/                 # Core HTTP client implementation
│   ├── Messages/               # Messages API (main resource)
│   ├── Files/                  # Files API resource
│   ├── Batches/                # Batch processing API
│   ├── Models/                 # Models API
│   ├── Embeddings/             # Embeddings API
│   ├── Resources/              # Base resource classes
│   ├── Requests/               # Request builders/DTOs
│   ├── Responses/              # Response objects & streaming
│   └── Contracts/              # Interfaces for DI
├── tests/
├── composer.json
└── README.md
```

### Key Architectural Patterns

1. **Client Pattern**: `Anthropic` class acts as main entry point, returns resource instances

   ```php
   $client = new Anthropic(apiKey: $key);
   $response = $client->messages->create([...]);
   ```

2. **Resource-Based API**: Each API endpoint group (Messages, Files, etc.) is a resource class inheriting from `Resource`

3. **Stateless Requests**: Follow Messages API pattern—always send full conversation history, no server-side state

4. **Dependency Injection Ready**:
   - Accept `HttpClientInterface` in constructor (PSR-18)
   - Use constructor injection for configuration
   - Support service container binding for frameworks

### Latest Claude Models (as of Nov 2025)

| Model                 | ID                           | Best For               | Input/Output Tokens        |
| --------------------- | ---------------------------- | ---------------------- | -------------------------- |
| **Claude Sonnet 4.5** | `claude-sonnet-4-5-20250929` | Complex agents, coding | $3/$15 per MTok, 200K ctx  |
| **Claude Haiku 4.5**  | `claude-haiku-4-5-20251001`  | Fast, cost-effective   | $1/$5 per MTok, 200K ctx   |
| **Claude Opus 4.1**   | `claude-opus-4-1-20250805`   | Specialized reasoning  | $15/$75 per MTok, 200K ctx |

**Aliases** (auto-update, prefer versioned IDs in production):

- `claude-sonnet-4-5` → latest Sonnet 4.5
- `claude-haiku-4-5` → latest Haiku 4.5

## API Endpoints to Implement

### Messages API (Core)

- `POST /v1/messages` - Create message (supports streaming)
- `POST /v1/messages/count_tokens` - Token counting

**Key Features**:

- Tool use (function calling)
- Vision (image inputs: base64 or URL)
- Streaming (server-sent events)
- Extended thinking (with `thinking` blocks)
- Batch processing via batch API

### Files API

- `POST /v1/files` - Upload file
- `GET /v1/files` - List files
- `GET /v1/files/{id}` - Get metadata
- `GET /v1/files/{id}/content` - Download
- `DELETE /v1/files/{id}` - Delete

### Batch Processing API

- `POST /v1/messages/batches` - Create batch (50% cost savings)
- `GET /v1/messages/batches` - List batches
- `GET /v1/messages/batches/{id}` - Get batch status
- `GET /v1/messages/batches/{id}/results` - Retrieve results
- `POST /v1/messages/batches/{id}/cancel` - Cancel batch
- `DELETE /v1/messages/batches/{id}` - Delete batch

### Models API

- `GET /v1/models` - List available models
- `GET /v1/models/{id}` - Get model details

### Embeddings API

- `POST /v1/embeddings` - Generate embeddings

## Critical Developer Workflows

### Client Initialization & Configuration

The SDK supports both **synchronous** (`Anthropic`) and **asynchronous** (`AsyncAnthropic`) clients:

```php
// Synchronous client
$client = new Anthropic(
    apiKey: $_ENV['ANTHROPIC_API_KEY'],  // Falls back to env var if not provided
    baseUrl: 'https://api.anthropic.com/v1',  // Override default
    timeout: 30.0,  // Seconds (default: 30)
    maxRetries: 2,  // Default retries for 429/5xx errors
);

// Asynchronous client (Amphp)
$asyncClient = new AsyncAnthropic(
    apiKey: $_ENV['ANTHROPIC_API_KEY'],
    timeout: 30.0,
);

// Async usage pattern
$promise = $asyncClient->messages->create([...]);
$message = \Amp\await($promise);
```

**Configuration Options**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claude-php/Claude-PHP-SDK](https://github.com/claude-php/Claude-PHP-SDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
