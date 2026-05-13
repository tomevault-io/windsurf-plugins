---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Laravel N8N is a Laravel package that provides a fluent client for the n8n public REST API and webhook triggering. It enables Laravel applications to interact with n8n workflows, executions, credentials, users, projects, and other n8n resources.

- **Namespace**: `KayedSpace\N8n`
- **PHP Version**: >=8.2
- **Laravel Version**: >=10

## Project Structure

```
src/
├── Client/
│   ├── Api/               # API resource classes (Workflows, Executions, etc.)
│   │   └── AbstractApi.php  # Base class with logging, caching, events, metrics
│   ├── Webhook/
│   │   └── Webhooks.php   # Webhook triggering with async queue support
│   └── N8nClient.php      # Main client class
├── Concerns/
│   └── HasPagination.php  # Trait for auto-pagination (all(), listIterator())
├── Console/               # Artisan commands for n8n management
│   ├── HealthCheckCommand.php
│   ├── ListWorkflowsCommand.php
│   ├── ActivateWorkflowCommand.php
│   ├── DeactivateWorkflowCommand.php
│   ├── ExecutionStatusCommand.php
│   └── TestWebhookCommand.php
├── Events/                # Laravel events for observability
│   ├── N8nEvent.php       # Base event class
│   ├── WorkflowCreated.php, WorkflowUpdated.php, etc.
│   ├── ExecutionCompleted.php, ExecutionFailed.php
│   ├── WebhookTriggered.php
│   ├── ApiRequestCompleted.php
│   └── RateLimitEncountered.php
├── Exceptions/            # Domain-specific exceptions
│   ├── N8nException.php   # Base exception with response context
│   ├── WorkflowNotFoundException.php
│   ├── ExecutionFailedException.php
│   ├── RateLimitException.php
│   └── AuthenticationException.php
├── Http/
│   └── Middleware/
│       └── VerifyN8nWebhook.php  # Webhook signature verification middleware
├── Jobs/
│   └── TriggerN8nWebhook.php     # Queue job for async webhooks
├── Facades/
│   └── N8nClient.php      # Laravel facade
├── Enums/
│   └── RequestMethod.php  # HTTP method enum
└── N8nServiceProvider.php # Service provider

config/
└── n8n.php               # Comprehensive configuration file

tests/
├── Unit/                 # Unit tests organized by class
└── Architecture/         # Architecture tests
```

## Architecture

### Client Design

The package follows a resource-based architecture with extensive features:

1. **N8nClient**: Main entry point that instantiates resource classes
2. **AbstractApi**: Enhanced base class for API resources that handles:
   - Authentication via `X-N8N-API-KEY` header
   - Request preparation (query parameter cleaning, boolean conversion)
   - **Logging**: PSR-3 compatible logging with configurable channels
   - **Caching**: Response caching with automatic invalidation on mutations
   - **Events**: Dispatches Laravel events for all operations
   - **Metrics**: Tracks request counts, duration, and status codes
   - **Retry Strategy**: Exponential/linear backoff with configurable delays
   - **Rate Limiting**: Auto-wait functionality for 429 responses
   - **Middleware**: Request pipeline customization via `middleware()` method
   - **Macros**: Extensibility via Laravel's Macroable trait
   - **Debug Mode**: Verbose request/response dumping
3. **Resource Classes**: Each n8n API resource extends AbstractApi and includes:
   - **Pagination Helpers**: `all()` and `listIterator()` methods
   - **Batch Operations**: `activateMany()`, `deleteMany()`, etc.
   - **Event Dispatching**: Resource-specific events (WorkflowCreated, etc.)
4. **Webhooks**: Enhanced webhook class with:
   - Async queue support via `async()` method
   - Signature verification (HMAC SHA-256)
   - Event dispatching for webhook triggers
   - Middleware for route protection

### Response Format

**Backward Compatible**: All methods return `Collection|array` union types:
- Returns `Collection` when `n8n.return_type = 'collection'` (default)
- Returns `array` when `n8n.return_type = 'array'`
- **Collections implement `ArrayAccess`** so `$data['key']` still works!
- This means **zero breaking changes** - existing code continues to work

### HTTP Client

- Uses Laravel's HTTP client (`Illuminate\Http\Client\PendingRequest`)
- Comprehensive configuration via `config/n8n.php`:
  - Timeout, retry count, throw on errors
  - Smart retry strategies (exponential, linear, constant)
  - Rate limit auto-wait with configurable max wait time
  - Request/response logging with body inclusion control
- API resources automatically add API key authentication
- Webhooks support Basic Auth with per-request override

### Events System

All operations dispatch Laravel events for observability:
- **Workflow Events**: WorkflowCreated, WorkflowUpdated, WorkflowDeleted, WorkflowActivated, WorkflowDeactivated
- **Execution Events**: ExecutionCompleted, ExecutionFailed, ExecutionDeleted
- **Webhook Events**: WebhookTriggered
- **API Events**: ApiRequestCompleted (every request), RateLimitEncountered
- Events can be disabled via `n8n.events.enabled = false`

### Exception Hierarchy

Domain-specific exceptions with response context:
- `N8nException` - Base exception with response data and context
- `WorkflowNotFoundException` - 404 workflow errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kayedspace/laravel-n8n](https://github.com/kayedspace/laravel-n8n) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
