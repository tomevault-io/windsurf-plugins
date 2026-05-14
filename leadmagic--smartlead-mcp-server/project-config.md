---
trigger: always_on
description: name: SmartLead MCP API Development Patterns
---

---
name: SmartLead MCP API Development Patterns
description: Development patterns for API clients and MCP tools in the SmartLead server
author: LeadMagic Team
version: 1.0.0
# 🏗️ SmartLead API Development Patterns & Best Practices

> **Developer Guide**: Master the patterns and practices used throughout the SmartLead MCP Server. This guide will help you write consistent, maintainable code that follows our established conventions.

## 🎯 **Quick Reference**

### **🔥 Most Important Patterns**
1. **Base Client Extension** → All API modules extend `BaseSmartLeadClient`
2. **MCP Tool Registration** → Use the standard registration pattern
3. **Zod Schema Validation** → All inputs validated with Zod
4. **Error Handling** → Use `SmartLeadError` for API errors
5. **Type Safety** → Avoid `any`, use proper TypeScript types

## 🏗️ **SmartLead API Client Architecture**

### **🛠️ Base Client Pattern** 
> **File**: [`src/client/base.ts`](mdc:src/client/base.ts)

The foundation of all API interactions:

```typescript
export abstract class BaseSmartLeadClient {
  protected config: SmartLeadConfig;
  
  constructor(config: SmartLeadConfig) {
    this.config = config;
  }
  
  // 🔄 Automatic retry with exponential backoff
  protected async withRetry<T>(operation: () => Promise<T>): Promise<T> {
    // Handles transient failures automatically
  }
  
  // 🌐 HTTP methods with proper typing
  protected async get<T>(url: string): Promise<T> { }
  protected async post<T>(url: string, data: unknown): Promise<T> { }
  protected async put<T>(url: string, data: unknown): Promise<T> { }
  protected async delete<T>(url: string): Promise<T> { }
}
```

**Key Features:**
- ✅ **HTTP request handling** with Axios
- ✅ **Automatic retry logic** with exponential backoff  
- ✅ **Rate limiting** and request queuing
- ✅ **Error handling** with SmartLeadError class
- ✅ **Request/response logging** for debugging

### **🔌 Module Structure Pattern**
> **Example**: [`src/modules/campaigns/client.ts`](mdc:src/modules/campaigns/client.ts)

Each API module follows this pattern:

```typescript
export class CampaignManagementClient extends BaseSmartLeadClient {
  /**
   * 🎯 Create a new campaign
   * @param params - Campaign creation parameters
   * @returns Promise resolving to campaign data
   */
  async createCampaign(params: CreateCampaignRequest): Promise<CampaignResponse> {
    return this.withRetry(() => 
      this.post<CampaignResponse>('/campaigns', params)
    );
  }
  
  /**
   * 📋 List campaigns with filtering
   * @param filters - Optional filtering parameters
   * @returns Promise resolving to campaign list
   */
  async listCampaigns(filters?: CampaignFilters): Promise<CampaignListResponse> {
    const params = new URLSearchParams();
    if (filters?.status) params.append('status', filters.status);
    if (filters?.limit) params.append('limit', filters.limit.toString());
    
    return this.withRetry(() => 
      this.get<CampaignListResponse>(`/campaigns?${params}`)
    );
  }
}
```

## 🛠️ **MCP Tool Development Patterns**

### **🎯 Tool Registration Pattern**
> **Example**: [`src/tools/campaigns.ts`](mdc:src/tools/campaigns.ts)

Standard structure for all MCP tools:

```typescript
export function registerCampaignTools(
  server: McpServer,
  client: SmartLeadClient,
  formatSuccessResponse: (message: string, data: unknown, summary?: string) => MCPToolResponse,
  handleError: (error: unknown) => MCPToolResponse
): void {
  
  // 🎯 Tool Registration
  server.registerTool(
    'smartlead_create_campaign',
    {
      title: 'Create SmartLead Campaign',
      description: 'Create a new email campaign with sequences and settings. Perfect for starting new outreach efforts.',
      inputSchema: CreateCampaignSchema.shape,
    },
    async (params) => {
      try {
        // ✅ 1. Validate input with Zod
        const validatedParams = CreateCampaignSchema.parse(params);
        
        // ✅ 2. Call API method  
        const result = await client.campaigns.createCampaign(validatedParams);
        
        // ✅ 3. Format success response with helpful summary
        const summary = `Created campaign "${validatedParams.name}" with ${validatedParams.sequences?.length || 0} sequences`;
        
        return formatSuccessResponse(
          'Campaign created successfully',
          result,
          summary
        );
      } catch (error) {
        // ✅ 4. Handle errors gracefully
        return handleError(error);
      }
    }
  );
}
```

### **🏷️ Tool Naming Convention**

| **Pattern** | **Example** | **Usage** |
|-------------|-------------|-----------|
| `smartlead_create_*` | `smartlead_create_campaign` | Creating new resources |
| `smartlead_get_*` | `smartlead_get_campaign` | Fetching single items |
| `smartlead_list_*` | `smartlead_list_campaigns` | Fetching multiple items |
| `smartlead_update_*` | `smartlead_update_campaign` | Modifying existing resources |
| `smartlead_delete_*` | `smartlead_delete_campaign` | Removing resources |
| `smartlead_fetch_*` | `smartlead_fetch_analytics` | Getting computed data |

**Specificity Rules:**
- Add `_by_*` for filtering: `smartlead_list_leads_by_campaign`
- Add `_with_*` for includes: `smartlead_get_campaigns_with_analytics`
- Add context when needed: `smartlead_fetch_warmup_stats_by_email_account`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadMagic/smartlead-mcp-server](https://github.com/LeadMagic/smartlead-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
