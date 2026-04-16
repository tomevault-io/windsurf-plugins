---
trigger: always_on
description: This document establishes development standards and best practices for the HeyReach Model Context Protocol (MCP) server, ensuring >75% success rate and production-ready quality.
---

# HeyReach MCP Server Development Standards

## Overview
This document establishes development standards and best practices for the HeyReach Model Context Protocol (MCP) server, ensuring >75% success rate and production-ready quality.

## Core Development Principles

### 1. API-First Development
- **Test API endpoints directly** before implementing MCP tools
- **Validate authentication** with raw HTTP calls first
- **Document working endpoints** in API_ENDPOINT_STATUS.md
- **Remove broken tools** rather than shipping unreliable functionality

### 2. Systematic Error Handling
- **Categorize errors** by HTTP status codes (401, 400, 404, 500+)
- **Provide actionable guidance** in error messages
- **Use consistent error response format** across all tools
- **Test error scenarios** as thoroughly as success cases

### 3. Incremental Tool Building
- **Start with core functionality** (authentication, basic operations)
- **Add tools incrementally** after validating dependencies
- **Maintain >75% success rate** before adding new features
- **Test each tool independently** and in combination

## MCP Server Architecture Standards

### Server Structure
```typescript
export class HeyReachMcpServer {
  private server: McpServer;
  private heyReachClient: HeyReachClient;
  
  constructor(config: HeyReachConfig) {
    this.server = new McpServer({
      name: 'heyreach-mcp-server',
      version: '1.1.8'
    }, {
      capabilities: { tools: {} }
    });
    
    this.heyReachClient = new HeyReachClient(config);
    this.setupTools();
  }
}
```

### Tool Implementation Pattern
```typescript
this.server.tool(
  'tool-name',
  {
    param: z.string().describe('Clear parameter description')
  },
  async ({ param }) => {
    try {
      // 1. Validate parameters
      validateRequiredParams({ param }, ['param'], 'tool-name');
      
      // 2. Call API client
      const result = await this.heyReachClient.methodName(param);
      
      // 3. Return success response
      return createSuccessResponse(result.data, 'Success message');
    } catch (error) {
      // 4. Handle errors consistently
      return createErrorResponse(error instanceof Error ? error.message : 'Unknown error');
    }
  }
);
```

## Authentication Standards

### HeyReach API Authentication
- **Use X-API-KEY header** (not Authorization Bearer)
- **Base URL**: `https://api.heyreach.io/api/public`
- **Timeout**: 30 seconds minimum for API responses
- **Handle empty responses** (HeyReach returns content-length: 0 for auth)

### API Client Configuration
```typescript
this.client = axios.create({
  baseURL: config.baseUrl || 'https://api.heyreach.io/api/public',
  headers: {
    'X-API-KEY': this.apiKey,
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
  timeout: 30000,
});
```

## Error Handling Standards

### MCP Error Codes
- **-32602**: Invalid params (validation errors)
- **-32603**: Internal error (server errors)
- **-32601**: Method not found
- **-32600**: Invalid request

### Error Response Pattern
```typescript
// Use consistent error response format
return createErrorResponse(error instanceof Error ? error.message : 'Unknown error');

// For validation errors, include helpful guidance
throw new McpError(
  ErrorCode.InvalidParams,
  `${toolName}: Parameter '${param}' is required. Use get-campaigns to get valid IDs.`
);
```

### API Error Categorization
```typescript
// Categorize HeyReach API errors properly
if (status === 401 || status === 403) {
  throw new McpError(ErrorCode.InvalidParams, 'Invalid API key');
}
if (status === 404) {
  throw new McpError(ErrorCode.InvalidParams, 'Resource not found');
}
if (message.includes('timeout')) {
  throw new McpError(ErrorCode.InternalError, 'Network timeout - retry in a few moments');
}
```

## Testing Standards

### Testing Hierarchy
1. **Direct API calls** (axios/curl) - Must pass first
2. **API client methods** - Test HeyReachClient directly
3. **MCP tool execution** - Test through MCP protocol
4. **End-to-end integration** - Test with Claude/Cursor

### Success Rate Requirements
- **>75% success rate** required for production deployment
- **100% authentication success** required for core functionality
- **Comprehensive error handling** for all failure scenarios
- **Real API testing** with valid credentials

### Test File Naming
- `test-api-*.js` - Direct API endpoint tests
- `test-client-*.js` - HeyReachClient method tests  
- `test-mcp-*.js` - MCP server protocol tests
- `test-integration-*.js` - End-to-end integration tests

## Development Workflow

### 1. Pre-Development
- [ ] Validate API endpoints with direct HTTP calls
- [ ] Document working endpoints and parameters
- [ ] Test authentication thoroughly
- [ ] Plan tool dependencies and prerequisites

### 2. Implementation
- [ ] Implement API client methods first
- [ ] Add comprehensive error handling
- [ ] Create MCP tool wrappers
- [ ] Test each tool independently

### 3. Quality Assurance
- [ ] Achieve >75% tool success rate
- [ ] Test error scenarios and edge cases
- [ ] Validate with real API credentials
- [ ] Document tool usage and examples

### 4. Deployment
- [ ] Update version number

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bcharleson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
