---
trigger: always_on
description: This document defines the coding conventions, architecture standards, and quality rules for this Laravel package. All agents (human or AI) must follow these rules — non-compliant contributions will be rejected.
---

# AGENTS

This document defines the coding conventions, architecture standards, and quality rules for this Laravel package. All agents (human or AI) must follow these rules — non-compliant contributions will be rejected.

For workflow, task management, and Claude Code-specific behavioral rules, see `CLAUDE.md`.

---

## Critical Rules

1. **Read documentation first** – Before working on any module, read the relevant documentation
2. **Documentation is the source of truth** – Documentation overrides all assumptions
3. **Run `composer check` before submitting code changes** – All lint, analyse, and test checks must pass (not required for documentation-only changes)
4. **Update documentation when code changes** – Keep docs in sync with implementation
5. **No function-level namespace imports** – Never use `use function json_decode;`
6. **Include PHPDoc on every class** – Document purpose and usage
7. **Respect layer boundaries** – Never violate the layer responsibility model
8. **Use dependency injection** – Never directly instantiate services; use contracts where appropriate
9. **Earn your abstractions** – Do not introduce indirection without concrete justification

---

## Atlas v3 Architecture

Atlas v3 is a unified AI SDK for Laravel applications. It owns its own provider layer — no external AI package dependency.

### Layer Model

```
Consumer API (Facade, fluent builders)
         ↓
Executor (tool loop, steps, orchestration events)
         ↓
Driver (routes modality calls to handlers)
         ↓
Handlers + Resolvers (build HTTP payloads, parse responses)
         ↓
HttpClient (sends HTTP, fires transport events)
```

### Key Principles

1. **Own the provider layer** — Atlas talks directly to AI provider APIs
2. **Drivers are thin coordinators** — they route to modality handlers, never build HTTP payloads
3. **Handlers compose resolvers** — MessageFactory, MediaResolver, ToolMapper, ResponseParser
4. **Stateless drivers** — one request → one response; the executor handles loops
5. **Shared HttpClient** — all providers use the same transport with consistent event dispatching

---

## Core Principles

1. Follow **PSR-12** and **Laravel Pint** formatting
2. Use **strict types** and modern **PHP 8.2+** syntax
3. All code must be **stateless**, **framework-aware**, and **application-agnostic**
4. Keep everything **self-contained** with no hard dependency on a consuming app
5. Always reference **documentation** for functional requirements and naming accuracy
6. Write clear, testable, deterministic code
7. Every class must include a **PHPDoc block** summarizing its purpose
8. **Program to interfaces, not implementations**, when multiple implementations or testing seams are required
9. **Single responsibility** – each class does one thing well
10. **Dependencies flow downward** – higher layers depend on lower layers only
11. **Earn your abstractions** – every layer must provide real value

**Example PHPDoc:**
```php
/**
 * Class UserWebhookService
 *
 * Handles webhook registration, processing, and retry logic for user-related events.
 */
```

---

## Package Structure

```
package-root/
├── docs/                 # VitePress documentation
├── src/
│   ├── Concerns/         # Cross-domain traits
│   ├── Console/          # Artisan commands
│   ├── Embeddings/       # Vector embeddings and similarity search
│   ├── Enums/            # Shared enums (Provider, Role, Modality, FinishReason, etc.)
│   ├── Events/           # Transport and orchestration events
│   ├── Exceptions/       # Exception hierarchy
│   ├── Executor/         # Agent executor, tool loop, steps
│   ├── Facades/          # Atlas facade
│   ├── Input/            # Media input types (Image, Audio, Video, Document)
│   ├── Messages/         # Typed conversation messages
│   ├── Middleware/        # MiddlewareStack and context objects
│   ├── Pending/          # Fluent request builders + Concerns/
│   ├── Persistence/      # Models/, Services/, Middleware/, Enums/, Concerns/
│   ├── Providers/        # Contracts/, Concerns/, Handlers/, Tools/, {Provider}/
│   ├── Queue/            # Queue dispatch infrastructure + Jobs/
│   ├── Requests/         # Immutable request DTOs
│   ├── Responses/        # Response objects and StorableContract
│   ├── Schema/           # JSON schema builder + Fields/
│   ├── Support/          # Pure utilities
│   ├── Testing/          # Fakes for testing
│   ├── Tools/            # Tool infrastructure (definition, serialization)
│   └── Voice/            # Voice session HTTP controllers
├── config/
├── tests/
│   ├── Unit/
│   └── Feature/
└── sandbox/
```

### Directory Rules

1. **Domain-organized** – Each top-level `src/` directory represents a domain concern, not a generic pattern
2. **Namespacing follows structure** – e.g., `Atlasphp\Atlas\Messages\UserMessage`
3. **Cross-domain references are allowed** – Domains may import types from other domains
4. **Domain-local contracts** – Interfaces live with their domain (e.g., `Providers/Contracts/`), not in a shared `Contracts/` directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atlas-php/atlas](https://github.com/atlas-php/atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
