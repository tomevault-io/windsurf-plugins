---
trigger: always_on
description: Guidelines for Creating a New MCP Server Package
---


# Creating a New MCP Server Package

This guide provides a step-by-step process for creating a new MCP server package in the MCP DevTools monorepo.

@url https://docs.cursor.com/context/rules-for-ai
@file packages/jira/package.json
@file packages/jira/src/index.ts
@file .cursor/rules/repository-structure.mdc
@file .cursor/rules/mcp-server-implementation.mdc
@file .cursor/rules/core-libraries-usage.mdc

## Step 1: Set Up Package Structure

First, create the basic package structure:

```bash
# 1. Create package directory
mkdir -p packages/[service-name]/src/{tools,api,types,utils}

# 2. Create main files
touch packages/[service-name]/package.json
touch packages/[service-name]/tsconfig.json
touch packages/[service-name]/README.md
touch packages/[service-name]/src/index.ts
touch packages/[service-name]/src/types/index.ts
```

## Step 2: Configure Package Files

### package.json

```json
{
  "name": "@mcp-devtools/[service-name]",
  "version": "0.1.0",
  "description": "MCP server for [service] integration",
  "main": "build/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node build/index.js",
    "dev": "ts-node src/index.ts",
    "test": "jest",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write 'src/**/*.ts'"
  },
  "files": ["build"],
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/github-user/mcp-devtools.git"
  },
  "dependencies": {
    "@modelcontextprotocol/server": "^0.1.0",
    "@modelcontextprotocol/types": "^0.1.0",
    "@mcp-devtools/core": "^0.1.0"
  },
  "devDependencies": {
    "@types/jest": "^29.5.0",
    "@types/node": "^18.0.0",
    "eslint": "^8.0.0",
    "jest": "^29.5.0",
    "prettier": "^2.8.0",
    "ts-jest": "^29.1.0",
    "ts-node": "^10.9.0",
    "typescript": "^5.0.0"
  },
  "publishConfig": {
    "access": "public"
  }
}
```

### tsconfig.json

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "build",
    "rootDir": "src"
  },
  "include": ["src/**/*"],
  "exclude": ["**/*.test.ts", "node_modules"]
}
```

## Step 3: Implement Basic Server

Create your `src/index.ts` file:

```typescript
import { createMcpServer } from "@modelcontextprotocol/server";
import { loadConfig } from "@mcp-devtools/core/config";
import { logger } from "@mcp-devtools/core/logger";

// Import tool registration functions
import { registerTool1 } from "./tools/tool1";
import { registerTool2 } from "./tools/tool2";

// Define config type
interface ServiceConfig {
  SERVICE_URL: string;
  SERVICE_API_KEY: string;
  DEBUG?: string;
}

// Load and validate configuration
const getConfig = (): ServiceConfig => {
  return loadConfig({
    requiredVars: ["SERVICE_URL", "SERVICE_API_KEY"],
    optionalVars: {
      DEBUG: "false",
    },
  }) as ServiceConfig;
};

// Initialize API client
import { initApiClient } from "./api/client";

const initServer = async () => {
  try {
    // Get config
    const config = getConfig();

    // Initialize API client
    const apiClient = initApiClient({
      baseUrl: config.SERVICE_URL,
      apiKey: config.SERVICE_API_KEY,
    });

    // Create MCP server
    const server = createMcpServer();

    // Register all tools
    registerTool1(server, apiClient);
    registerTool2(server, apiClient);

    // Start the server
    server.start();
    logger.info("[service-name] MCP server started successfully");
  } catch (error) {
    logger.error("Failed to start [service-name] MCP server", { error });
    process.exit(1);
  }
};

// Start the server
initServer();
```

## Step 4: Create API Client

Create your `src/api/client.ts` file:

```typescript
import { httpClient } from "@mcp-devtools/core/http";
import { logger } from "@mcp-devtools/core/logger";

interface ApiClientConfig {
  baseUrl: string;
  apiKey: string;
}

export interface ApiClient {
  get: (path: string, options?: any) => Promise<any>;
  post: (path: string, data: any, options?: any) => Promise<any>;
  put: (path: string, data: any, options?: any) => Promise<any>;
  delete: (path: string, options?: any) => Promise<any>;
}

export const initApiClient = (config: ApiClientConfig): ApiClient => {
  const headers = {
    Authorization: `Bearer ${config.apiKey}`,
    "Content-Type": "application/json",
  };

  return {
    get: async (path, options = {}) => {
      try {
        const url = `${config.baseUrl}${path}`;
        return await httpClient.get(url, { ...options, headers });
      } catch (error) {
        logger.error(`API GET request failed: ${path}`, { error });
        throw error;
      }
    },

    post: async (path, data, options = {}) => {
      try {
        const url = `${config.baseUrl}${path}`;
        return await httpClient.post(url, data, { ...options, headers });
      } catch (error) {
        logger.error(`API POST request failed: ${path}`, { error });
        throw error;
      }
    },

    put: async (path, data, options = {}) => {
      try {
        const url = `${config.baseUrl}${path}`;
        return await httpClient.put(url, data, { ...options, headers });
      } catch (error) {
        logger.error(`API PUT request failed: ${path}`, { error });
        throw error;
      }
    },

    delete: async (path, options = {}) => {
      try {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DXHeroes/mcp-devtools](https://github.com/DXHeroes/mcp-devtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
