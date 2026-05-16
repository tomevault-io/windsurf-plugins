---
trigger: always_on
description: **Rule Priority:** Advanced Integration
---

# Model Context Protocol (MCP) Integration

**Rule Priority:** Advanced Integration  
**Activation:** External tool integration and service connections  
**Scope:** All MCP-compatible tools and service integrations

## Overview

Model Context Protocol (MCP) in Cursor v1.2+ enables seamless integration with external tools and services through standardized interfaces. This allows one-click installation of tools, OAuth authentication flows, and powerful service integrations without complex setup.

## MCP Architecture

### Core MCP Components

```typescript
// MCP Server Implementation
interface MCPServer {
  name: string;
  version: string;
  capabilities: MCPCapabilities;
  tools: MCPTool[];
  resources: MCPResource[];
  prompts: MCPPrompt[];
}

interface MCPCapabilities {
  logging?: boolean;
  notifications?: boolean;
  resources?: {
    subscribe?: boolean;
    listChanged?: boolean;
  };
  tools?: {
    listChanged?: boolean;
  };
  prompts?: {
    listChanged?: boolean;
  };
}

interface MCPTool {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  handler: (params: any) => Promise<MCPResult>;
}

interface MCPResource {
  uri: string;
  name: string;
  description?: string;
  mimeType?: string;
}

interface MCPPrompt {
  name: string;
  description: string;
  arguments?: MCPPromptArgument[];
}
```

### MCP Client Configuration

```json
// .cursor/mcp-config.json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-supabase"],
      "env": {
        "SUPABASE_URL": "process.env.SUPABASE_URL",
        "SUPABASE_ANON_KEY": "process.env.SUPABASE_ANON_KEY"
      }
    },
    "github": {
      "command": "npx", 
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "process.env.GITHUB_TOKEN"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-memory"],
      "env": {
        "MEMORY_STORE_PATH": "./data/mcp-memory"
      }
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-sequential-thinking"]
    },
    "playwright": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-playwright"]
    },
    "context7": {
      "command": "npx", 
      "args": ["@context7/mcp-server"]
    }
  },
  "allowedOrigins": [
    "cursor://",
    "vscode://",
    "localhost"
  ],
  "enableOAuth": true,
  "oauthConfig": {
    "providers": ["github", "google", "microsoft"],
    "redirectUrl": "cursor://oauth/callback"
  }
}
```

## One-Click Tool Installation

### Built-in MCP Tools

```typescript
// Available MCP tools for one-click installation
const availableMCPTools = {
  // Database and Storage
  supabase: {
    name: '@modelcontextprotocol/server-supabase',
    description: 'Supabase database operations',
    category: 'database',
    requiresAuth: true,
    capabilities: ['read', 'write', 'schema', 'realtime']
  },
  
  // Code and Documentation
  github: {
    name: '@modelcontextprotocol/server-github',
    description: 'GitHub repository operations',
    category: 'development',
    requiresAuth: true,
    capabilities: ['repos', 'issues', 'prs', 'files']
  },
  
  // AI and ML
  context7: {
    name: '@context7/mcp-server',
    description: 'Context7 library documentation',
    category: 'ai',
    requiresAuth: false,
    capabilities: ['search', 'documentation', 'examples']
  },
  
  // Web and Testing
  playwright: {
    name: '@modelcontextprotocol/server-playwright',
    description: 'Web browser automation',
    category: 'testing',
    requiresAuth: false,
    capabilities: ['navigate', 'interact', 'screenshot', 'test']
  },
  
  // Productivity
  memory: {
    name: '@modelcontextprotocol/server-memory',
    description: 'Persistent knowledge graphs',
    category: 'productivity',
    requiresAuth: false,
    capabilities: ['store', 'retrieve', 'search', 'graph']
  },
  
  // AI Reasoning
  'sequential-thinking': {
    name: '@modelcontextprotocol/server-sequential-thinking',
    description: 'Step-by-step problem solving',
    category: 'ai',
    requiresAuth: false,
    capabilities: ['reasoning', 'analysis', 'planning']
  }
};

// Installation workflow
class MCPInstaller {
  async installTool(toolName: string): Promise<InstallResult> {
    const tool = availableMCPTools[toolName];
    if (!tool) throw new Error(`Tool ${toolName} not found`);
    
    // 1. Install package
    await this.installPackage(tool.name);
    
    // 2. Configure MCP server
    await this.configureMCPServer(toolName, tool);
    
    // 3. Setup authentication if required
    if (tool.requiresAuth) {
      await this.setupAuthentication(toolName);
    }
    
    // 4. Verify installation
    return this.verifyInstallation(toolName);
  }
  
  private async setupAuthentication(toolName: string): Promise<void> {
    // Launch OAuth flow or prompt for API keys
    const authMethod = await this.detectAuthMethod(toolName);
    
    if (authMethod === 'oauth') {
      await this.launchOAuthFlow(toolName);
    } else {
      await this.promptForAPIKey(toolName);
    }
  }
}
```

### SYMindX-Specific MCP Integrations

```typescript
// Custom MCP server for SYMindX
export class SYMindXMCPServer implements MCPServer {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
