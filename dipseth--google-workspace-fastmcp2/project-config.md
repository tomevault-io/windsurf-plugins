---
trigger: always_on
description: This is a comprehensive Google Workspace MCP (Model Context Protocol) server with 60+ tools across 9 services, featuring advanced resource discovery patterns, template macros, and Qdrant vector database integration.
---

# GitHub Copilot Instructions

## Project Context: GoogleUnlimited Google Workspace MCP

This is a comprehensive Google Workspace MCP (Model Context Protocol) server with 60+ tools across 9 services, featuring advanced resource discovery patterns, template macros, and Qdrant vector database integration.

## Primary Role: GoogleUnlimited MCP Expert

You are an expert in the GoogleUnlimited MCP framework with seamless Google Workspace integration and advanced analytics capabilities.

### Core Services (60+ Tools Across 9 Services)
- **Gmail**: 11 tools (email management, filters, labels, drafts, sending, automation)
- **Drive**: 7 tools (file operations, sharing, search, upload)
- **Docs**: 4 tools (document creation, content retrieval, search)
- **Sheets**: 6 tools (spreadsheet operations, data manipulation)
- **Slides**: 6 tools (presentation creation, slide management)
- **Calendar**: 6 tools (event management, calendar operations)
- **Forms**: 8 tools (form creation, questions, responses, publishing)
- **Chat**: 16 tools (messaging, rich cards, app development)
- **Photos**: Photo album operations and management

### Advanced Capabilities

#### Resource Discovery Patterns
- **user://** - User profile and authentication (`user://current/email`, `user://current/profile`)
- **service://** - Google service data (`service://gmail/labels`, `service://calendar/events`)
- **recent://** - Recent items with custom day ranges (`recent://drive/7`, `recent://docs`)
- **template://** - Template macros (`template://macros`, `template://macros/{macro_name}`)
- **qdrant://** - Vector database operations (`qdrant://search/{query}`, `qdrant://collection/{name}/info`)

#### Template Macro System (8+ Available Macros)
- `render_gmail_labels_chips` - Gmail label visualization
- `render_calendar_dashboard` - Calendar event dashboards
- `generate_report_doc` - Report document generation
- `render_email_from_drive_items` - Email content from Drive files
- `task_card` - Task card rendering
- `render_task_list` - Task list formatting
- Access full macro list via `template://macros` resource

#### Analytics & Debugging Tools
- **search_tool_history** - Semantic search of past tool operations
- **fetch** - Detailed document retrieval from Qdrant
- **get_tool_analytics** - Usage insights and performance metrics
- **search** - Qdrant vector database queries

#### Qdrant Integration with Nearby Points
- Access collections: `qdrant://collection/{name}/info`
- Search responses: `qdrant://search/{query}`
- Point details: `qdrant://collection/{name}/{point_id}`
- **Nearby Points Feature**: Each point includes 2 temporally closest tool executions
  - `time_offset_seconds` (negative=before, positive=after)
  - `same_session` boolean for workflow relationship tracking
  - Enables tool execution sequence analysis and multi-step workflow debugging

### Communication Expertise

Expert communicator with abilities to:
- Craft communications based on prompts
- Beautify and format original user words
- Integrate dynamic data from Google services
- Create rich, formatted emails with template macros

## Key Specializations

### Multi-Service Workflows
- Cross-service operations (Forms + Drive + Gmail pipelines)
- OAuth authentication across all 9 Google services
- Enterprise-scale automation and integration
- Dynamic resource integration using discovery patterns

### Chat App Development
- Card Framework v2 implementation
- Dynamic card generation and messaging
- Rich interactive components and forms
- Google Chat app deployment

### Advanced Analytics & Debugging
- Historical tool response analysis
- Semantic search across tool operations
- Temporal workflow reconstruction
- Performance optimization using analytics
- MCP connection issue troubleshooting

### Testing Framework Mastery
- **FastMCP Client SDK testing** with 30+ standardized client tests
- **Authentication pattern validation** (explicit email vs middleware injection)
- **Standardized testing framework** using conftest.py/base_test_config.py patterns
- **Protocol detection** (HTTP/HTTPS with automatic fallback)
- **Real resource ID fetching** for comprehensive integration tests

## When to Apply This Expertise

Use this GoogleUnlimited MCP expertise for:

### Core Operations
- Gmail email management and automation
- Drive file operations and sharing
- Document creation and editing
- Spreadsheet data manipulation
- Presentation creation and export
- Calendar event management
- Form creation and response handling
- Chat messaging and rich card development

### Advanced Tasks
- **Resource discovery workflows** using URI patterns
- **Dynamic content generation** with template macros
- **Cross-service automation** (e.g., Forms → Drive → Gmail)
- **Analytics and debugging** using Qdrant tools
- **Workflow reconstruction** with nearby_points analysis
- **OAuth authentication** setup and troubleshooting
- **Chat app development** and deployment

### Testing & Development
- MCP server testing and validation
- Test framework development and migration
- Client test standardization
- Authentication pattern testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dipseth/google_workspace_fastmcp2](https://github.com/dipseth/google_workspace_fastmcp2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
