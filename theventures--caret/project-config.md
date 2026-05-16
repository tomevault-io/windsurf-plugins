---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

This is an unofficial Node.js API client for the Caret HTTP API (https://docs.caret.so/api-reference/introduction). Caret is an AI-powered meeting assistant that provides real-time guidance, coaching, and knowledge base capabilities during calls, along with automated transcription and meeting summaries.

## Development Commands

- **Install dependencies**: `bun install`
- **Run the application**: `bun run src/index.ts`
- **TypeScript compilation check**: `bun tsc --noEmit`
- **Run tests**: `bun test`
- **Format and lint code**: `bun format`
- **Build for distribution**: `bun run build`
- **Publish to npm**: `bun publish`

## API Client Architecture

The client should implement:

- **Base URL**: `https://api.caret.so/v1`
- **Authentication**: API key authentication (`X-API-Key` header)
- **API Scopes**: Permission-based access control (`users`, `notes`, `folders`, `templates`)
- **Rate limiting**: Standard endpoints (100 requests/min), Heavy operations (20 requests/min)
- **Core resources**:
  - Notes (list, get, update meeting notes; create from audio — Enterprise only)
  - Folders (create, list workspace folders — replaces deprecated Tags)
  - Workspace (get workspace details and settings)
  - Members (list workspace members with pagination)
  - Invites (list, create, delete workspace invites)
  - Zapier Webhooks (register/delete webhook URLs for Zapier integration)
- **MCP Server**: Caret also provides an MCP server at `https://api.caret.so/mcp` for AI assistant integration

## Design Patterns (Inspired by OpenAI Node.js Client)

### 1. Resource-Based Architecture
```typescript
// Main client class
class Caret {
  notes = new Notes(this);
  folders = new Folders(this);
  workspace = new Workspace(this);
  members = new Members(this);
  invites = new Invites(this);
  zapier = new Zapier(this);
}

// Base resource pattern
abstract class APIResource {
  protected _client: Caret;
  constructor(client: Caret) {
    this._client = client;
  }
}
```

### 2. Error Handling
- Hierarchical error system with specific error types
- HTTP status-based error generation
- Rich error context (status, headers, requestId)

### 3. Configuration
```typescript
const caret = new Caret({
  apiKey: process.env.CARET_API_KEY,
  baseURL: 'https://api.caret.so/v1',
  timeout: 30000,
  maxRetries: 3
});
```

### 4. Method Naming Conventions
- Consistent CRUD operations: `create()`, `get()`, `update()`, `list()`, `delete()`
- Resource-specific methods following REST patterns

## API Models

TypeScript interfaces for core models:

- **Note**: id, title, kind, status, createdAt, updatedAt, visibility (only-me/workspace/public), tags, participants, totalDurationSec, userWrittenNote, enhancedNote, summary, transcripts, calendarEvent, inputLanguage, translationLanguage, meetingApp
- **Folder**: id, name, icon, parentFolderId, createdAt, memberCount
- **Workspace**: id, name, createdAt, updatedAt, settings (company info, key terms, audio storage, data retention, meeting permissions, sensitive info masking), allowedEmailDomains, iconUrl
- **Member**: id, name, email, profileUrl, role (admin/member), createdAt, folders
- **Invite**: id, email, code, role, expiresAt, createdAt, isUrlInvite, folders

## Project Structure

```
src/
├── core/           # Core SDK functionality (errors, etc.)
├── resources/      # API endpoint implementations
│   ├── notes.ts
│   ├── folders.ts    # Workspace folders (replaces tags.ts)
│   ├── workspace.ts
│   ├── members.ts    # Workspace members
│   ├── invites.ts    # Workspace invites
│   └── zapier.ts     # Zapier webhook integration
├── types/          # TypeScript type definitions
├── webhooks/       # Webhook signature verification
├── client.ts       # Main Caret client class
└── index.ts        # Primary export point

tests/
├── __helpers__/    # Shared test utilities and mocks
├── core/           # Tests for core functionality
├── resources/      # Tests for API resources
├── webhooks/       # Tests for webhook verification
├── client.test.ts  # Core client tests
└── errors.test.ts  # Error handling tests
```

## Implementation Guidelines

- Use TypeScript with strict typing for API responses
- Implement proper error handling for API errors and rate limits
- Support async/await patterns
- Method overloading for different parameter combinations
- Rich parameter interfaces rather than primitive types
- Generate types from API documentation where possible
- **IMPORTANT: Always update README.md documentation when implementing new API endpoints**
  - Add usage examples for new endpoints
  - Update TypeScript type examples if new types are added
  - Ensure all new methods are documented with clear examples

## Testing

The project maintains comprehensive test coverage using Bun's built-in testing framework without additional dependencies.

### Test Coverage
- **Complete coverage** of client functionality, error handling, and API resources
- **Zero dependencies** - uses only `bun test` built-in capabilities

### Test Structure
- **Mock Strategy**: Uses Bun's native `mock()` function for HTTP request mocking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theventures/caret](https://github.com/theventures/caret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
