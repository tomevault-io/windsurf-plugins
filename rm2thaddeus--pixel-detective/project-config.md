---
trigger: always_on
description: when asked to use MCP tools, follow the below guidelines
---


# MCP Interaction Rule for Pixel Detective

**Repository:** [Pixel_Detective](mdc:https:/github.com/rm2thaddeus/Pixel_**Branches:**
- main
- development

## Available MCP Servers & Capabilities

### 1. **GitHub MCP (`mcp_github_*`)**
- **Primary Use**: Version control, repository management, collaboration workflows
- **Capabilities**:
  - Search code, repositories, commits, issues, and PRs
  - Read, create, update, or delete files
  - Manage issues (create, list, comment, update) and pull requests (create, list, merge, update)
  - Manage branches and tags
- **When to Use**: All code commits, file operations, issue tracking, PR management

### 2. **Supabase MCP (`mcp_supabase_*`)**
- **Primary Use**: Database interactions, backend data management
- **Capabilities**:
  - Execute raw SQL queries
  - Manage database schemas, tables, and data (CRUD operations)
  - Seed databases and perform migrations/rollbacks
- **When to Use**: Database schema changes, data seeding, backend data operations

### 3. **Browser Tools MCP (`mcp_browser-tools_*`)**
- **Primary Use**: Frontend debugging, web analysis, performance optimization
- **Capabilities**:
  - Capture console logs (standard and errors) and network logs
  - Take screenshots for visual debugging
  - Run comprehensive audits (accessibility, performance, SEO, best practices, Next.js)
  - Wipe logs for clean debugging sessions
- **When to Use**: Debugging frontend issues, performance analysis, accessibility compliance

#### **Browser Tools MCP Setup & Troubleshooting**

**⚠️ CRITICAL SETUP REQUIREMENTS:**
Browser Tools MCP requires **three components** to function properly:

1. **Chrome Extension** - Captures browser data
2. **Browser Tools Server** - Node.js middleware (port 3025)
3. **Browser Tools MCP** - The MCP server itself

**Setup Verification Steps:**
```bash
# 1. Check Node.js version (requires v18+)
node --version

# 2. Start Browser Tools Server
npx @agentdeskai/browser-tools-server@latest

# 3. Verify server is running
netstat -an | findstr 3025  # Windows
# or
lsof -i :3025  # Mac/Linux
```

**Chrome Extension Installation:**
1. Clone repository: `git clone https://github.com/AgentDeskAI/browser-tools-mcp.git`
2. Open Chrome → `chrome://extensions/`
3. Enable "Developer mode"
4. Click "Load unpacked"
5. Select folder: `browser-tools-mcp/chrome-extension`
6. Verify "BrowserToolsMCP" appears in extensions

**Connection Verification:**
1. Open any webpage (e.g., https://example.com)
2. Open Chrome DevTools (F12)
3. Look for "BrowserTools" tab in DevTools
4. Verify connection status shows "Connected"

**Common Error Patterns:**
- `"Failed to discover browser connector server"` → Browser Tools Server not running
- `"Chrome extension not connected"` → Extension not installed or not connected
- `"Error taking screenshot"` → Extension installed but not connected to active tab

**Debugging Commands:**
```bash
# Test MCP connection
mcp_browser-tools_wipeLogs  # Should return "All logs cleared successfully"

# Test server connection  
mcp_browser-tools_getConsoleLogs  # Should return [] if no logs

# Test full functionality (requires extension)
mcp_browser-tools_takeScreenshot  # Should capture screenshot or show specific error
```

**Node.js Version Issues:**
- Browser Tools MCP requires Node.js v18+ (uses native fetch API)
- If using nvm: `nvm alias default 20` to set default version
- Verify with: `node --version` before starting server

### 4. **Browser Control MCP (`mcp_browser-control-*`)**
- **Primary Use**: Web navigation, content extraction, browser automation
- **Capabilities**:
  - Open/close tabs and manage browser sessions
  - List open tabs and browser history
  - Retrieve full web page content and extract links
  - Find and highlight specific text within web pages
- **When to Use**: Web scraping, content analysis, automated testing workflows

### 5. **Context7 MCP (`mcp_context7_*`)**
- **Primary Use**: Documentation lookup, API reference, library guidance
- **Capabilities**:
  - Resolve library/package names to documentation IDs
  - Fetch up-to-date library documentation and API references
- **When to Use**: When implementing new libraries, troubleshooting API usage, code examples

### 6. **Docker MCP (`docker-mcp`)**
- **Primary Use**: Container management, Docker Compose operations, containerized development
- **Capabilities**:
  - Create and manage Docker containers
  - Deploy and manage Docker Compose stacks
  - Retrieve container logs for debugging
  - List and monitor container status
- **When to Use**: Containerized applications, microservices, development environment setup

### 7. **Mindmap MCP (`mindmap`)**
- **Primary Use**: Visual planning, architecture documentation, sprint planning
- **Capabilities**:
  - Create structured mind maps for complex features and requirements
  - Generate visual representations of system architecture
  - Support sprint planning and PRD (Product Requirement Document) generation
  - Create project roadmaps and feature breakdowns
- **When to Use**: Sprint planning, complex feature planning, architecture documentation, PRD generation

## Best Practices for MCP Interactions

### **Pre-Action Context Gathering**
1. **Always establish context first**:
   - Verify current working directory (`pwd`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
