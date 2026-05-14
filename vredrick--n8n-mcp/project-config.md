---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

n8n-mcp is a comprehensive documentation and knowledge server that provides AI assistants with complete access to n8n node information through the Model Context Protocol (MCP). It serves as a bridge between n8n's workflow automation platform and AI models, enabling them to understand and work with n8n nodes effectively.

## ✅ Latest Updates (v2.7.4)

### Update (v2.7.4) - Self-Documenting MCP Tools:
- ✅ **RENAMED: start_here_workflow_guide → tools_documentation** - More descriptive name
- ✅ **NEW: Depth parameter** - Control documentation detail level with "essentials" or "full"
- ✅ **NEW: Per-tool documentation** - Get help for any specific tool by name
- ✅ **Concise by default** - Essential info only, unless full depth requested
- ✅ **LLM-friendly format** - Plain text, not JSON for better readability
- ✅ **Two-tier documentation**:
  - **Essentials**: Brief description, key parameters, example, performance, 2-3 tips
  - **Full**: Complete documentation with all parameters, examples, use cases, best practices, pitfalls
- ✅ **Quick reference** - Call without parameters for immediate help
- ✅ **8 documented tools** - Comprehensive docs for most commonly used tools
- ✅ **Performance guidance** - Clear indication of which tools are fast vs slow
- ✅ **Error prevention** - Common pitfalls documented upfront

### Update (v2.7.0) - Diff-Based Workflow Editing with Transactional Updates:
- ✅ **NEW: n8n_update_partial_workflow tool** - Update workflows using diff operations for precise, incremental changes
- ✅ **RENAMED: n8n_update_workflow → n8n_update_full_workflow** - Clarifies that it replaces the entire workflow
- ✅ **NEW: WorkflowDiffEngine** - Applies targeted edits without sending full workflow JSON
- ✅ **80-90% token savings** - Only send the changes, not the entire workflow
- ✅ **13 diff operations** - addNode, removeNode, updateNode, moveNode, enableNode, disableNode, addConnection, removeConnection, updateConnection, updateSettings, updateName, addTag, removeTag
- ✅ **Smart node references** - Use either node ID or name for operations
- ✅ **Transaction safety** - Validates all operations before applying any changes
- ✅ **Validation-only mode** - Test your diff operations without applying them
- ✅ **Comprehensive test coverage** - All operations and edge cases tested
- ✅ **Example guide** - See [workflow-diff-examples.md](./docs/workflow-diff-examples.md) for usage patterns
- ✅ **FIXED: MCP validation error** - Simplified schema to fix "additional properties" error in Claude Desktop
- ✅ **FIXED: n8n API validation** - Updated cleanWorkflowForUpdate to remove all read-only fields
- ✅ **FIXED: Claude Desktop compatibility** - Added additionalProperties: true to handle extra metadata from Claude Desktop
- ✅ **NEW: Transactional Updates** - Two-pass processing allows adding nodes and connections in any order
- ✅ **Operation Limit** - Maximum 5 operations per request ensures reliability
- ✅ **Order Independence** - Add connections before nodes - engine handles dependencies automatically

### Update (v2.6.3) - n8n Instance Workflow Validation:
- ✅ **NEW: n8n_validate_workflow tool** - Validate workflows directly from n8n instance by ID
- ✅ **Fetches and validates** - Retrieves workflow from n8n API and runs comprehensive validation
- ✅ **Same validation logic** - Uses existing WorkflowValidator for consistency
- ✅ **Full validation options** - Supports all validation profiles and options
- ✅ **Integrated workflow** - Part of complete lifecycle: discover → build → validate → deploy → execute
- ✅ **No JSON needed** - AI agents can validate by just providing workflow ID

### Update (v2.6.2) - Enhanced Workflow Creation Validation:
- ✅ **NEW: Node type validation** - Verifies node types actually exist in n8n
- ✅ **FIXED: nodes-base prefix detection** - Now catches `nodes-base.webhook` BEFORE database lookup
- ✅ **NEW: Smart suggestions** - Detects `nodes-base.webhook` and suggests `n8n-nodes-base.webhook`
- ✅ **NEW: Common mistake detection** - Catches missing package prefixes (e.g., `webhook` → `n8n-nodes-base.webhook`)
- ✅ **NEW: Minimum viable workflow validation** - Prevents single-node workflows (except webhooks)
- ✅ **NEW: Empty connection detection** - Catches multi-node workflows with no connections
- ✅ **Enhanced error messages** - Clear guidance on proper workflow structure
- ✅ **Connection examples** - Shows correct format: `connections: { "Node Name": { "main": [[{ "node": "Target", "type": "main", "index": 0 }]] } }`
- ✅ **Helper functions** - `getWorkflowStructureExample()` and `getWorkflowFixSuggestions()`
- ✅ **Prevents broken workflows** - Like single webhook nodes with empty connections that show as question marks
- ✅ **Reinforces best practices** - Use node NAMES (not IDs) in connections

### Update (v2.6.1) - Enhanced typeVersion Validation:
- ✅ **NEW: typeVersion validation** - Workflow validator now enforces typeVersion on all versioned nodes
- ✅ **Catches missing typeVersion** - Returns error with correct version to use
- ✅ **Warns on outdated versions** - Alerts when using older node versions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vredrick/n8n-mcp](https://github.com/vredrick/n8n-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
