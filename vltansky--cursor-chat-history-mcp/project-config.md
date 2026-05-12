---
trigger: always_on
description: This project creates an MCP (Model Context Protocol) server that **exposes your Cursor conversation history** to AI assistants like Claude. It acts as a bridge between your local Cursor chat database and AI tools.
---

# Cursor Conversations MCP Server - Project Overview

## Core Concept
This project creates an MCP (Model Context Protocol) server that **exposes your Cursor conversation history** to AI assistants like Claude. It acts as a bridge between your local Cursor chat database and AI tools.

## Primary Goal
**Enable AI assistants to analyze your actual coding conversations** to provide personalized, context-aware assistance based on your real development patterns and history.

## Key Value Proposition
- **Personalized Coding Rules**: Generate coding standards from your actual conversations, not generic best practices
- **Pattern Recognition**: Identify recurring themes and solutions in your development work
- **Context-Aware Help**: Get assistance informed by your specific projects and coding style
- **Historical Insights**: Learn from past problem-solving sessions and apply proven solutions

## How It Works
1. **Local Access**: Reads your Cursor conversation database directly (no external services)
2. **MCP Protocol**: Exposes conversation data through standardized MCP tools
3. **AI Integration**: AI assistants can query, analyze, and extract insights from your chat history
4. **Privacy First**: All data stays local - no external API calls or data sharing

## Common Use Cases
- `"Analyze my React conversations and create component guidelines"`
- `"Generate TypeScript rules based on my actual coding patterns"`
- `"Find similar debugging sessions and extract solutions"`
- `"Create project documentation from relevant conversations"`

## Technical Architecture
- **Database**: SQLite access to Cursor's conversation storage at `src/database/`
- **MCP Server**: TypeScript implementation at `src/server.ts`
- **Tools**: Conversation analysis tools at `src/tools/`
- **Entry Point**: Main configuration in `package.json` and `README.md`

---

# General Project Guidelines
Use npm for package management

## Code Organization
- Follow the established directory structure in `src/`
- Group related functionality in modules
- Use barrel exports for clean public APIs

## Development Workflow
- Build with `npm run build` before testing
- Use TypeScript for all new code
- Follow patterns established in existing codebase

## File Extensions
- Use `.js` extensions for local imports (required for ESM)
- TypeScript files should be `.ts`

---

# MCP Server Development Guide

This guide covers best practices for developing Model Context Protocol (MCP) servers using the TypeScript SDK.

## Tool Registration Pattern

- **Use McpServer with Proper Description Format**
  - Tool descriptions are provided as a separate string parameter, not within the schema object
  - Pass the Zod schema shape directly as the third parameter
  - Import `z` from 'zod' for schema definitions

```typescript
// ✅ DO: Correct tool registration with description
server.tool(
  'my_tool',
  'Clear description of what this tool does and its purpose',
  {
    param1: z.string().min(1),
    param2: z.number().optional().default(10),
    param3: z.array(z.string()).optional()
  },
  async (input) => {
    // input is properly typed and validated
    return {
      content: [{ type: 'text', text: 'result' }]
    };
  }
);

// ❌ DON'T: Include description in schema object
server.tool(
  'my_tool',
  {
    description: 'Tool description', // This causes TypeScript errors
    param1: z.string().min(1),
    param2: z.number().optional().default(10)
  },
  async (input) => {
    // This pattern is incorrect
  }
);

// ❌ DON'T: Missing description entirely
server.tool(
  'my_tool',
  {
    param1: z.string().min(1),
    param2: z.number().optional().default(10)
  },
  async (input) => {
    // Tools without descriptions are less discoverable
  }
);
```

## Parameter Access Pattern

- **Direct Parameter Access**
  - Tool handlers receive validated parameters directly as the first argument
  - No need to access `request.params.arguments`
  - Parameters are automatically validated against the Zod schema

```typescript
// ✅ DO: Direct parameter access
async (input) => {
  // input is typed and validated
  const { conversationId, includeMetadata } = input;
}

// ❌ DON'T: Manual parameter extraction
async (request) => {
  const input = request.params.arguments as MyInputType;
}
```

## MCP-Specific Error Handling

- **MCP Tool Response Format**
  - Always return content in the expected MCP format
  - Use consistent error response structure
  - Include meaningful error messages for debugging

```typescript
async (input) => {
  try {
    const result = await myOperation(input);
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(result, null, 2)
      }]
    };
  } catch (error) {
    return {
      content: [{
        type: 'text',
        text: `Error: ${error instanceof Error ? error.message : 'Unknown error occurred'}`
      }]
    };
  }
}
```

## Server Setup Pattern

- **Standard Server Initialization**
  - Use `McpServer` from the official SDK
  - Connect with `StdioServerTransport` for CLI tools
  - Await the connection to ensure proper setup

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vltansky/cursor-chat-history-mcp](https://github.com/vltansky/cursor-chat-history-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
