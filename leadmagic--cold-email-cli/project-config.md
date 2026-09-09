---
trigger: always_on
description: The SmartLead CLI supports **Model Context Protocol (MCP)** integration for enhanced AI-assisted development and automation capabilities.
---

# MCP (Model Context Protocol) Configuration

## 🌐 Overview

The SmartLead CLI supports **Model Context Protocol (MCP)** integration for enhanced AI-assisted development and automation capabilities.

## ⚙️ MCP Server Configuration

### SmartLead MCP Server
Based on the original SmartLead MCP server implementation, this CLI provides comprehensive API access.

```json
{
  "mcpServers": {
    "smartlead": {
      "command": "npx",
      "args": [
        "smartlead-cli",
        "mcp-server"
      ],
      "env": {
        "SMARTLEAD_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### Instantly MCP Server (Coming Soon)
Future integration with Instantly platform:

```json
{
  "mcpServers": {
    "instantly": {
      "command": "npx",
      "args": [
        "smartlead-cli",
        "mcp-server",
        "--module=instantly"
      ],
      "env": {
        "INSTANTLY_API_KEY": "your-instantly-api-key"
      }
    }
  }
}
```

## 🔧 Configuration Files

### MCP Configuration Location
- **Global**: `~/.smartlead-cli/mcp.json`
- **Project**: `.smartlead-mcp.json`
- **Environment**: Via environment variables

### Example MCP Configuration
Create `.smartlead-mcp.json` in your project root:

```json
{
  "version": "1.0.0",
  "servers": {
    "smartlead": {
      "module": "smartlead",
      "apiKey": "${SMARTLEAD_API_KEY}",
      "baseUrl": "https://server.smartlead.ai/api/v1",
      "capabilities": [
        "campaigns",
        "leads", 
        "email-accounts",
        "analytics",
        "webhooks"
      ],
      "rateLimit": {
        "requests": 100,
        "window": "1m"
      }
    },
    "instantly": {
      "module": "instantly",
      "apiKey": "${INSTANTLY_API_KEY}",
      "baseUrl": "https://api.instantly.ai/api/v1",
      "capabilities": [
        "campaigns",
        "leads",
        "sequences",
        "analytics"
      ],
      "available": false,
      "comingSoon": "Q2 2024"
    }
  },
  "context": {
    "projectType": "email-marketing-automation",
    "modules": ["smartlead", "instantly"],
    "integrations": ["webhook", "api", "cli"],
    "documentation": [
      "docs/README.md",
      "docs/CONTRIBUTING.md", 
      "docs/ROADMAP.md"
    ]
  }
}
```

## 🚀 MCP Integration Features

### Available MCP Tools
The CLI provides MCP-compatible tools for:

#### SmartLead Tools
- **Campaign Management**: Create, update, start, pause, stop campaigns
- **Lead Operations**: Add, update, delete, search leads
- **Email Account Management**: Setup, warmup, health monitoring
- **Analytics**: Campaign performance, lead statistics, exports
- **Webhook Management**: Create, update, delete webhooks

#### Context Tools
- **Configuration**: API key management, module switching
- **Documentation**: Access to comprehensive docs and examples
- **Status Monitoring**: Real-time API status and health checks

### MCP Tool Examples

#### Campaign Tools
```typescript
// MCP Tool: smartlead_campaign_create
{
  "name": "smartlead_campaign_create",
  "description": "Create a new SmartLead campaign",
  "inputSchema": {
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "clientId": {"type": "number", "optional": true}
    },
    "required": ["name"]
  }
}

// MCP Tool: smartlead_campaign_analytics
{
  "name": "smartlead_campaign_analytics", 
  "description": "Get campaign analytics and performance metrics",
  "inputSchema": {
    "type": "object",
    "properties": {
      "campaignId": {"type": "number"},
      "startDate": {"type": "string", "optional": true},
      "endDate": {"type": "string", "optional": true}
    },
    "required": ["campaignId"]
  }
}
```

#### Lead Tools
```typescript
// MCP Tool: smartlead_lead_add
{
  "name": "smartlead_lead_add",
  "description": "Add leads to a SmartLead campaign",
  "inputSchema": {
    "type": "object", 
    "properties": {
      "campaignId": {"type": "number"},
      "leads": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "email": {"type": "string"},
            "firstName": {"type": "string", "optional": true},
            "lastName": {"type": "string", "optional": true},
            "companyName": {"type": "string", "optional": true}
          },
          "required": ["email"]
        }
      }
    },
    "required": ["campaignId", "leads"]
  }
}
```

## 🛠️ Development Integration

### MCP Server Implementation
The CLI includes an MCP server mode for AI integration:

```typescript
// src/mcp/server.ts
export class SmartLeadMCPServer {
  private modules: Map<ModuleName, CLIModule>;
  
  public async handleToolCall(tool: string, args: any): Promise<any> {
    const [moduleName, command] = tool.split('_');
    const module = this.modules.get(moduleName as ModuleName);
    
    if (module) {
      return await module.executeCommand(command, args);
    }
    
    throw new Error(`Unknown tool: ${tool}`);
  }
}
```

### Context7 Integration
For enhanced context management:

```json
{
  "context7": {
    "smartlead-cli": {
      "type": "typescript-cli",
      "architecture": "modular",
      "modules": {
        "smartlead": {
          "status": "available",
          "commands": 80,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
