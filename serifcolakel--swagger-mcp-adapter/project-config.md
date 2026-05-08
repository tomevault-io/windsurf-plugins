---
trigger: always_on
description: This is a TypeScript-based MCP (Model Context Protocol) server that integrates with Swagger/OpenAPI specifications to expose API endpoints as tools for Large Language Models (LLMs). The project provides a seamless way to connect existing REST APIs to LLMs, allowing them to make requests and receive responses through standardized MCP commands.
---

# GitHub Copilot Instructions for Swagger MCP Adapter

## Project Overview

This is a TypeScript-based MCP (Model Context Protocol) server that integrates with Swagger/OpenAPI specifications to expose API endpoints as tools for Large Language Models (LLMs). The project provides a seamless way to connect existing REST APIs to LLMs, allowing them to make requests and receive responses through standardized MCP commands.

The server reads Swagger/OpenAPI files, parses the endpoints, and exposes them as MCP tools that LLMs can invoke. This enables LLMs to interact with your APIs in a structured, validated manner.

## Architecture & Structure

Based on best practices for TypeScript/Node.js projects, this project follows a modular structure optimized for maintainability and scalability:

```
swagger-mcp-adapter/
├─ src/
│  ├─ index.ts              # MCP server entrypoint
│  ├─ server.ts             # MCP server setup (SDK integration)
│  ├─ swagger/
│  │   ├─ loader.ts         # Load & validate swagger/openapi files
│  │   ├─ parser.ts         # Parse specifications into normalized endpoints
│  │   ├─ cache.ts          # Memory state management for loaded specs
│  │   └─ types.ts          # TypeScript type definitions from OpenAPI
│  ├─ commands/
│  │   ├─ listServices.ts   # MCP command to list available services
│  │   ├─ callService.ts    # MCP command to call a specific service
│  │   └─ loadSpec.ts       # MCP command to load a new swagger spec
│  ├─ http/
│  │   ├─ client.ts         # HTTP client wrapper (axios/fetch)
│  │   └─ validator.ts      # Request/response validation with Zod
│  ├─ utils/
│  │   └─ logger.ts         # Structured logging utility
│  └─ config.ts             # Configuration management
├─ test/
│  ├─ swagger.mock.json     # Mock OpenAPI specification for testing
│  └─ server.test.ts        # Unit and integration tests
├─ package.json             # Project dependencies and scripts
├─ biome.json               # Biome configuration
├─ tsconfig.json            # TypeScript configuration
├─ README.md                # Project documentation
```

### Directory Guidelines

#### `/src` - Source Code

- Contains all TypeScript source files
- Organized by feature/domain for better maintainability
- Entry point is `index.ts` for the MCP server

#### `/src/swagger` - OpenAPI/Swagger Handling

- `loader.ts`: Handles loading OpenAPI specs from files or URLs
- `parser.ts`: Parses specifications into normalized data structures
- `cache.ts`: Memory state management for loaded specs and endpoints
- `types.ts`: Generated TypeScript interfaces from OpenAPI schemas

#### `/src/commands` - MCP Commands

- Implements MCP protocol commands as tools for LLMs
- `listServices.ts`: Returns available API endpoints with clean markdown formatting
- `getServiceInformation.ts`: Provides detailed information about a specific service including parameters, schemas, and examples
- `getAllServiceInformation.ts`: Provides comprehensive information about all services from a specification
- `getCacheInformation.ts`: Monitors cache status, expiration times, and performance metrics

#### `/src/http` - HTTP Client & Validation

- `client.ts`: Generic HTTP client for making API requests
- `validator.ts`: Schema validation using Zod based on OpenAPI specs

#### `/src/utils` - Utilities

- Shared utility functions and helpers
- Logging, error handling, and common operations

#### `/test` - Test Files

- Unit tests for individual modules
- Integration tests for MCP server functionality
- Mock data for testing without external dependencies

## TypeScript Modules & Dependencies

- Use npm for dependency management (`package.json`)
- Module should use ESM (`"type": "module"` in package.json)
- Keep dependencies minimal and regularly updated
- Use `npm audit` and `npm update` for security and updates
- Export main functionality through `package.json` exports field

## TypeScript Development Standards

### Code Style & Formatting

- Always use `biome` for code formatting and linting
- Follow TypeScript naming conventions: camelCase for variables/functions, PascalCase for classes/interfaces
- Use meaningful variable names; avoid abbreviations
- Keep functions small and focused on a single responsibility
- Prefer `const` over `let`, use `let` only when reassignment is necessary

### Error Handling

- **Always handle errors appropriately** after async operations
- Use `try/catch` blocks for synchronous errors
- Return errors as rejected Promises for async operations
- Create custom error classes for domain-specific errors
- Use structured logging for error details with context

```typescript
// Good
try {
  const result = await service.callEndpoint(params);
  return result;
} catch (error) {
  logger.error("Failed to call endpoint", { error, params });
  throw new APIError("Service call failed", { cause: error });
}
```

### Memory State Management

- **Spec Cache**: Maintain in-memory cache of loaded Swagger/OpenAPI specifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serifcolakel/swagger-mcp-adapter](https://github.com/serifcolakel/swagger-mcp-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
