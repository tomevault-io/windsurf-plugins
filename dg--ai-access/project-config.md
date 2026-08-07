---
trigger: always_on
description: It is my fervent wish that this file guide every AI coding agent working with code in this repository.
---

# To My Agents!

It is my fervent wish that this file guide every AI coding agent working with code in this repository.

## Documentation

Any distilled, agent-facing documentation for this package - how it works
internally and the rationale behind key design decisions - lives in `docs/`.
Consult it before non-trivial changes; it is the source of truth from which the
public manual is distilled.

## Project Overview

**AIAccess** is a unified PHP library providing a consistent interface for accessing multiple AI model providers (OpenAI, Anthropic Claude, Google Gemini, DeepSeek, and Grok/xAI). The library abstracts provider-specific differences, allowing developers to write AI integration code once and switch providers with minimal changes.

**Key Features:**
- Single unified API across multiple providers
- Support for Chat, Batch processing, and Embeddings
- Modern PHP 8.1+ with strict types throughout
- No vendor SDK dependencies (uses native HTTP client)

## Development Commands

### Testing
```bash
# Run all tests
composer run tester

# Run specific test file
vendor/bin/tester tests/Chat/Chat.phpt -s -C

# Run tests in specific directory
vendor/bin/tester tests/Chat/ -s -C
```

### Static Analysis
```bash
# Run PHPStan (level 8)
composer run phpstan
```

### Installation
```bash
# Install dependencies
composer install
```

## Architecture

### Provider Abstraction Pattern

The library uses **interface-based abstraction** to standardize behavior across different AI providers:

```
AIAccess\Chat\Service (interface)
  ├── OpenAI\Client
  ├── Claude\Client
  ├── Gemini\Client
  ├── DeepSeek\Client
  └── Grok\Client
```

Each provider implements core interfaces:
- `Chat\Service` - Creates chat sessions
- `Batch\Service` - Optional batch processing (OpenAI, Claude only)
- `Embedding\Service` - Optional embeddings (OpenAI, Gemini only)
- `Http\Client` - HTTP transport abstraction (default: CurlClient)

### Abstract Base Class Pattern

`AIAccess\Chat\Chat` is an abstract class that defines the conversation contract:
- Message history management via `addMessage()` and `getMessages()`
- System instructions via `setSystemInstruction()`
- Common `sendMessage()` flow with error recovery
- Abstract `generateResponse()` for provider-specific implementation

Each provider extends this: `OpenAI\Chat`, `Claude\Chat`, `Gemini\Chat`, etc.

### Directory Structure

```
src/
├── Batch/              # Batch processing abstractions
├── Chat/               # Core chat abstractions (Chat, Response, Message, Role, etc.)
├── Embedding/          # Text embeddings support
├── Http/               # HTTP transport layer (Client, CurlClient, Response)
├── Provider/           # Provider-specific implementations
│   ├── OpenAI/
│   ├── Claude/
│   ├── Gemini/
│   ├── DeepSeek/
│   └── Grok/
├── Helpers.php         # Internal utilities (JSON encoding/decoding)
└── exceptions.php      # Exception hierarchy
```

### Exception Hierarchy

Exception design focuses on **recovery strategy**:

```
ServiceException                    (Base for all service errors)
├── ApiException                   (API returned error response - may be retriable)
├── CommunicationException         (Network or parse errors - potentially retriable)
└── UnexpectedResponseException    (Response structure mismatch - not retriable)

LogicException                      (Programming errors - fix during development)
```

**Error Handling Pattern:**
- `ApiException`: API explicitly returned an error (rate limits, validation, etc.)
- `CommunicationException`: Network issues or invalid JSON - retry may help
- `UnexpectedResponseException`: Response doesn't match expected schema - log and investigate
- `LogicException`: Invalid parameters or wrong method call order - fix in development

## Code Organization Principles

### Flat Structure Within Categories
All entities in single namespace with clear suffixes (`ChatResponse`, `BatchResponse`, `Vector`). Avoids deep nesting; prefers descriptive class names.

### Provider Parallelism
Each provider directory (`OpenAI/`, `Claude/`, etc.) has identical structure:
- `Client.php` - API communication and service factory
- `Chat.php` - Conversation state management
- `ChatResponse.php` - Response parsing and transformation
- `Batch.php` - Batch request container (if supported)
- `BatchResponse.php` - Batch response parsing (if supported)

This makes adding new providers straightforward: copy and adapt.

### Single Responsibility
- `Client` - API communication, service factory, HTTP handling
- `Chat` - Conversation state, message history
- `ChatResponse` - Response parsing, data transformation
- Clean separation enables easier testing and maintenance

## Testing with Nette Tester

Tests use `.phpt` extension and follow this structure:

```php
<?php

declare(strict_types=1);

use Tester\Assert;

require __DIR__ . '/../bootstrap.php';

test('Description of what is being tested', function () {
    $object = new SomeClass();
    $result = $object->doSomething();

    Assert::same('expected value', $result);
});
```

**Key Points:**
- Use `test()` function for each test case
- First parameter is a clear description (no need for separate comments)
- Group related tests in same file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dg/ai-access](https://github.com/dg/ai-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
