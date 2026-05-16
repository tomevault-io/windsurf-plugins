---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

LacyLights MCP is a Model Context Protocol (MCP) server that enables AI-powered theatrical lighting design. It acts as a bridge between AI assistants (like Claude) and the LacyLights lighting control system, providing natural language control of DMX fixtures, scene generation, and cue management.

**Role in LacyLights Ecosystem:**
- AI integration layer for the LacyLights system
- Provides 80+ MCP tools for lighting control
- Connects AI assistants to the lacylights-go backend
- Enables natural language scene generation and script analysis

## Development Commands

### Development
```bash
npm run dev              # Start dev server with auto-reload (tsx watch)
npm run build            # Compile TypeScript to dist/
npm start                # Run production build from dist/
```

### Testing
```bash
npm test                 # Run all tests
npm run test:watch       # Run tests in watch mode
npm run test:coverage    # Generate coverage report
```

### Code Quality
```bash
npm run lint             # Check for linting errors
npm run lint:fix         # Fix auto-fixable linting errors
```

## Architecture

### Directory Structure

```
lacylights-mcp/
├── src/
│   ├── index.ts              # Main entry point, MCP protocol handler
│   ├── tools/                # MCP tool implementations
│   │   ├── project-tools.ts  # Project CRUD operations
│   │   ├── fixture-tools.ts  # Fixture management
│   │   ├── look-tools.ts     # Look generation and control
│   │   └── cue-tools.ts      # Cue sequence and playback
│   ├── services/             # Core services
│   │   ├── graphql-client-simple.ts  # Apollo client for backend
│   │   ├── ai-lighting.ts    # OpenAI integration
│   │   └── rag-service-simple.ts     # Pattern matching service
│   └── types/
│       └── lighting.ts       # TypeScript interfaces
├── tests/                    # Test files (mirrors src/)
├── run-mcp.js               # MCP client integration wrapper
└── dist/                    # Compiled output
```

### Key Technologies

- **TypeScript**: CommonJS modules (target ES2018)
- **MCP SDK**: @modelcontextprotocol/sdk for AI integration
- **Apollo GraphQL**: Client for backend communication
- **OpenAI API**: AI-powered scene generation
- **Jest**: Testing with ts-jest

### Data Flow

1. AI client (Claude) calls MCP tool via stdio
2. `index.ts` routes to appropriate tool handler
3. Tool handler calls GraphQL client to lacylights-go backend
4. For AI operations, uses OpenAI API + RAG patterns
5. Response JSON returned to AI client

## Important Patterns

### MCP Tool Development
When adding new MCP tools:

1. **Define tool** in `ListToolsRequestSchema` handler with input schema
2. **Add handler case** in `CallToolRequestSchema` handler
3. **Implement method** in appropriate tool class
4. **Write tests** in `tests/tools/*.test.ts`
5. **Document** in README.md function reference

```typescript
// Tool definition pattern
{
  name: "tool_name",
  description: "What this tool does",
  inputSchema: {
    type: "object",
    properties: { ... },
    required: [...]
  }
}
```

### Safe Look Updates
Use "safe" look update functions to preserve existing fixtures:
- `addFixturesToLook`: Add without affecting others
- `removeFixturesFromLook`: Remove specific fixtures only
- `ensureFixturesInLook`: Add only if missing
- `updateLookPartial`: Merge updates instead of replacing

**Always prefer these over `updateLook` to avoid data loss.**

### GraphQL Client Usage
All backend communication through `LacyLightsGraphQLClient`:
- Methods are strongly typed with TypeScript
- Errors automatically thrown from GraphQL responses
- Fixture definitions flattened into instances

### Error Handling
- Catch GraphQL errors and return structured responses
- Validate inputs before calling backend
- Provide helpful error messages for AI context

## Testing Guidelines

### Test Structure
- Tests mirror `src/` structure in `tests/`
- Use Jest with ts-jest preset
- Mock fetch calls with `jest.mock('cross-fetch')`

### Common Test Patterns
```typescript
// Mock GraphQL responses
const mockResponse = {
  json: jest.fn().mockResolvedValue({ data: { projects: [] } })
};
mockFetch.mockResolvedValue(mockResponse as any);

// Test error handling
mockFetch.mockResolvedValue({
  json: jest.fn().mockResolvedValue({ errors: [{ message: 'Error' }] })
});
```

### Coverage Requirements
- Lines: 70%
- Functions: 65%
- Branches: 50%
- Test timeout: 10 seconds

## CI/CD

| Workflow | File | Purpose |
|----------|------|---------|
| CI | `ci.yml` | Tests, lint on PRs |
| Release | `release.yml` | Build and publish releases |

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LACYLIGHTS_GRAPHQL_ENDPOINT` | Yes | Backend GraphQL URL (default: http://localhost:4000/graphql) |
| `OPENAI_API_KEY` | For AI features | OpenAI API key for scene generation |
| `CHROMA_HOST` | No | ChromaDB host for persistent RAG |
| `CHROMA_PORT` | No | ChromaDB port (default: 8000) |

### Module System
- Uses CommonJS (`module: "commonjs"`) not ESM
- Main entry: `dist/index.js` after build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbernstein/lacylights-mcp](https://github.com/bbernstein/lacylights-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
