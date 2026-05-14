---
trigger: always_on
description: Enables **fully automated scoped application development** with zero manual steps!
---

# ServiceNow MCP Server v2.0 - Development Guide

**For:** Claude Code (claude.ai/code)
**Version:** 2.0
**Instance:** Multi-instance support enabled

---

## 🚨 CRITICAL: MCP-FIRST POLICY

**ALWAYS use MCP tools as the PRIMARY method for ServiceNow operations.**

### Priority Order (MANDATORY):
1. **FIRST**: Use MCP tools (SN-Create-Record, SN-Update-Record, SN-Query-Table, etc.)
2. **SECOND**: If MCP fails, use SN-Execute-Background-Script
3. **LAST RESORT**: Document as manual step only if both fail

### When to Write Background Scripts:
- **NEVER** as first choice
- **ONLY** when MCP tools have documented limitations
- **ALWAYS** try MCP batch operations first (5-10+ parallel calls)

---

## 🎯 MCP Tools (40+ Total)

### Generic Tools (Work on ANY table)
**Core CRUD:**
- `SN-Query-Table` - Query records with filtering
- `SN-Create-Record` - Create records in any table
- `SN-Get-Record` - Get single record by sys_id
- `SN-Update-Record` - Update existing records
- `SN-Get-Table-Schema` - Get table structure
- `SN-Discover-Table-Schema` - Deep schema with relationships
- `SN-List-Available-Tables` - List all available tables

### Specialized Tables (Convenience)
**ITSM:**
- `SN-List-Incidents`, `SN-Create-Incident`, `SN-Get-Incident`
- `SN-List-ChangeRequests`
- `SN-List-Problems`

**Administration:**
- `SN-List-SysUsers`
- `SN-List-SysUserGroups`
- `SN-List-CmdbCis`

### Convenience Tools (Incident Operations)
- `SN-Update-Incident` - Update incident by sys_id
- `SN-Close-Incident` - Close incident with resolution
- `SN-Assign-Incident` - Assign incident to user/group
- `SN-Add-Work-Notes` - Add work notes to incident
- `SN-Search-Incidents` - Natural language incident search

### Update Set Management
- `SN-Set-Update-Set` - Set current update set (AUTOMATED!)
- `SN-Get-Current-Update-Set` - Get active update set
- `SN-List-Update-Sets` - List available update sets
- `SN-Move-Records-To-Update-Set` - Fix records in wrong set
- `SN-Clone-Update-Set` - Clone entire update set
- `SN-Inspect-Update-Set` - Inspect update set contents

### Application Scope
- `SN-Set-Current-Application` - Set current application scope (AUTOMATED!)

### Script Execution
- `SN-Execute-Background-Script` - Execute server-side JavaScript (via sys_trigger)
- `SN-Create-Fix-Script` - Generate script for manual execution

### Script Synchronization
- `SN-Sync-Script-To-Local` - Download script to local file
- `SN-Sync-Local-To-Script` - Upload local file to ServiceNow
- `SN-Watch-Script` - Watch local file and auto-sync changes

### Natural Language Interface
- `SN-NL-Search` - Search records using natural language
- `SN-NL-Query-Builder` - Convert NL to encoded query

### Batch Operations
- `SN-Batch-Create` - Create multiple records with relationships
- `SN-Batch-Update` - Update multiple records efficiently

### Workflow Operations
- `SN-Create-Workflow` - Create complete workflow with activities
- `SN-Create-Activity` - Add activity to workflow
- `SN-Create-Transition` - Link workflow activities
- `SN-Publish-Workflow` - Publish workflow version

### Advanced Tools
- `SN-Validate-Configuration` - Validate catalog item config
- `SN-Explain-Field` - Get field documentation

### MCP Resources (Read-Only Access)
- `servicenow://instances` - List available instances
- `servicenow://schema/{table}` - Table schema as resource
- `servicenow://scripts/{sys_id}` - Script include contents

**Complete API Reference:** `docs/API_REFERENCE.md`

---

## 🌐 Multi-Instance Support

All tools support the `instance` parameter for routing to specific ServiceNow instances.

**Configuration:** `config/servicenow-instances.json`

```json
{
  "instances": [
    {
      "name": "dev",
      "url": "https://dev123.service-now.com",
      "username": "admin",
      "password": "password",
      "default": true
    },
    {
      "name": "prod",
      "url": "https://prod456.service-now.com",
      "username": "integration",
      "password": "password"
    }
  ]
}
```

**Usage:**
```javascript
// Default instance (marked with "default": true)
SN-Query-Table({ table_name: "incident", limit: 10 })

// Specific instance
SN-Query-Table({ table_name: "incident", limit: 10, instance: "prod" })
```

---

## 🎉 BREAKTHROUGH: Automated Background Script Execution

**As of 2025-09-29**, discovered automated background script execution via `sys_trigger` table:

### SN-Execute-Background-Script (FULLY AUTOMATED!)
```javascript
SN-Execute-Background-Script({
  script: "gs.info('Hello from automated script');",
  description: "Test automated execution",
  execution_method: "trigger"  // Default, most reliable
});
```

**How it works:**
1. Creates scheduled job in `sys_trigger` table
2. Executes in ~1 second
3. Auto-deletes trigger after execution
4. No manual copy-paste required!

**Fallback methods:**
- `ui`: Direct UI endpoint execution
- `auto`: Try trigger → ui → create fix script

### SN-Set-Update-Set (FULLY AUTOMATED!)
```javascript
// Set current update set programmatically
SN-Set-Update-Set({ update_set_sys_id: "abc123..." });
```

Uses automated background script execution - takes ~2 seconds, fully scriptable!

### SN-Set-Current-Application (FULLY AUTOMATED!)
```javascript
// Set application scope for scoped app development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Happy-Technologies-LLC/happy-platform-mcp](https://github.com/Happy-Technologies-LLC/happy-platform-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
