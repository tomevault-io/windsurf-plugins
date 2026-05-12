---
trigger: always_on
description: This repository contains a TypeScript client library for the OpenHands Agent Server. It provides a complete, type-safe interface for interacting with the OpenHands Agent Server API, enabling developers to build applications that can create and manage AI agent conversations, workspaces, and real-time event streams.
---

# Agent Server TypeScript Client

## General Purpose

This repository contains a TypeScript client library for the OpenHands Agent Server. It provides a complete, type-safe interface for interacting with the OpenHands Agent Server API, enabling developers to build applications that can create and manage AI agent conversations, workspaces, and real-time event streams.

The client is designed to mirror the structure and functionality of the Python SDK (`software-agent-sdk`) while providing idiomatic TypeScript/JavaScript APIs with full type safety and modern development tooling.

## Key Features

- **Complete API Coverage**: Implements all endpoints from the OpenHands Agent Server OpenAPI specification
- **Type Safety**: Full TypeScript support with comprehensive interfaces and type definitions
- **Real-time Events**: WebSocket client for streaming conversation events and agent status updates
- **Workspace Management**: File operations, uploads, downloads, and workspace state management
- **Conversation Lifecycle**: Create, start, stop, and manage AI agent conversations
- **Error Handling**: Robust error handling with custom exception classes and retry logic
- **Modern Tooling**: ESLint, Prettier, Jest testing framework, and GitHub Actions CI/CD

## Browser Compatibility Requirement

**All code in this library must be browser-compatible.** This is a hard constraint — the SDK is designed to run in browser environments and must never depend on Node.js-specific APIs.

**Do NOT use:**

- `fs`, `fs/promises`, or any filesystem APIs
- `child_process`, `spawn`, `exec`, or any process execution APIs
- `path` (Node.js module — use string manipulation or URL APIs instead)
- `os`, `net`, `http`, `https`, `stream`, `buffer` (Node.js built-in), `crypto` (Node.js built-in), or any other Node.js built-in modules
- Any npm package that depends on Node.js built-in modules

**DO use:**

- `fetch` for HTTP requests
- `WebSocket` for real-time communication
- Web-standard APIs (`URL`, `Blob`, `File`, `FormData`, `TextEncoder`/`TextDecoder`, etc.)
- Browser-compatible npm packages only

This applies to all source code under `src/`. Test files (`src/__tests__/`) are an exception since they run in Node.js via Jest.

## Source Material

This TypeScript client is based on the following source materials:

### 1. OpenAPI Specification

- **Source**: [OpenHands Docs - Agent SDK OpenAPI](https://github.com/OpenHands/docs/blob/main/openapi/agent-sdk.json)
- **Purpose**: Defines the complete REST API specification for the OpenHands Agent Server
- **Usage**: Used to generate TypeScript interfaces, API client methods, and ensure complete endpoint coverage

### 2. Python SDK Reference Implementation

- **Source**: [OpenHands Software Agent SDK](https://github.com/OpenHands/software-agent-sdk)
- **Key Components**:
  - `RemoteConversation` class - Main conversation management
  - `RemoteWorkspace` class - Workspace file operations
  - `RemoteState` class - Agent state and configuration management
- **Purpose**: Provides the architectural blueprint and API design patterns
- **Usage**: Ensures consistent class names, method signatures, and behavior across language implementations

### 3. Agent Server Implementation

- **Source**: Located within the `software-agent-sdk` repository as `agent-server`
- **Purpose**: The actual server implementation that this client communicates with
- **Usage**: Reference for understanding expected request/response formats and WebSocket event structures

## Architecture Alignment

The TypeScript client maintains architectural consistency with the Python SDK:

```typescript
// TypeScript Client (this repo)
const conversation = new RemoteConversation(config);
await conversation.start();
await conversation.workspace.write_file('/path/file.txt', 'content');
const state = conversation.state;
```

```python
# Python SDK (reference implementation)
conversation = RemoteConversation(config)
await conversation.start()
await conversation.workspace.write_file('/path/file.txt', 'content')
state = conversation.state
```

### Workspace Architecture

The workspace module follows the Python SDK's architecture with a common interface and multiple implementations:

```
src/workspace/
├── base.ts           # IWorkspace interface defining the contract
├── remote-workspace.ts  # RemoteWorkspace - connects to remote agent server
├── local-workspace.ts   # LocalWorkspace - stub (throws errors, directs to RemoteWorkspace)
└── workspace.ts      # Factory functions and Workspace class (backwards compatible)
```

**IWorkspace Interface**: Defines the common contract for all workspace implementations:

- `executeCommand()` - Execute bash commands
- `fileUpload()` / `fileDownload()` - File operations
- `gitChanges()` / `gitDiff()` - Git operations
- `close()` - Cleanup resources

**RemoteWorkspace**: Fully implemented class that connects to a remote OpenHands agent server via HTTP API.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenHands/typescript-client](https://github.com/OpenHands/typescript-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
