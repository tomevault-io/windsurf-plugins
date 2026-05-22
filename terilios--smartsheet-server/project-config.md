---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Smartsheet MCP (Model Context Protocol) server that enables automated operations on Smartsheet documents through a standardized interface. It bridges AI-powered automation tools with Smartsheet's collaboration platform, with specialized healthcare analytics capabilities including clinical research analytics, hospital operations, and healthcare innovation scoring.

**Current Version**: v0.3.0 - Now includes 34 tools with comprehensive cross-sheet reference management capabilities.

## Architecture

### Server Structure
- **TypeScript MCP Server** (`src/index.ts`): Main server implementing MCP protocol with both STDIO and HTTP transport modes
- **Python Operations Layer** (`smartsheet_ops/`): Python package handling Smartsheet API operations and batch analysis
- **Dual Transport Support**: STDIO for local CLI usage, HTTP for web-based clients

### Key Components
- **MCP Server** (`src/index.ts`): Handles tool registration, request routing, and error handling
- **CLI Operations** (`smartsheet_ops/smartsheet_ops/cli.py`): Python CLI for Smartsheet operations
- **Batch Analysis** (`smartsheet_ops/smartsheet_ops/batch_analysis.py`): Healthcare analytics with Azure OpenAI integration
- **Operations Core** (`smartsheet_ops/smartsheet_ops/__init__.py`): Core Smartsheet API operations

## Common Development Tasks

### Build and Run

```bash
# Install dependencies
npm install
cd smartsheet_ops && pip install -e . && cd ..

# Build TypeScript server
npm run build

# Watch mode for development
npm run watch

# Run MCP inspector
npm run inspector

# Start server with STDIO transport (default)
PYTHON_PATH=/path/to/python SMARTSHEET_API_KEY=your-key node build/index.js

# Start server with HTTP transport
PYTHON_PATH=/path/to/python SMARTSHEET_API_KEY=your-key node build/index.js --transport http --port 3000
```

### Environment Setup

Required environment variables:
- `SMARTSHEET_API_KEY`: Smartsheet API access token
- `PYTHON_PATH`: Path to Python executable with smartsheet_ops installed
- `AZURE_OPENAI_API_KEY`: Azure OpenAI key (for batch analysis features)
- `AZURE_OPENAI_API_BASE`: Azure OpenAI endpoint
- `AZURE_OPENAI_API_VERSION`: API version
- `AZURE_OPENAI_DEPLOYMENT`: Deployment name

### Working with Tools

The server provides 19 tools for Smartsheet operations. When implementing new tools:

1. Add tool definition in `setupToolHandlersForServer()` method in `src/index.ts`
2. Implement corresponding Python operation in `smartsheet_ops/smartsheet_ops/cli.py`
3. Add operation to choices list in `parse_args()` function
4. Handle the operation in the main() function

### Key Implementation Patterns

#### Tool Registration Pattern
```typescript
tools.push({
  name: 'tool_name',
  description: 'Tool description',
  inputSchema: {
    type: 'object',
    properties: {
      // Define parameters
    },
    required: ['required_params']
  }
});
```

#### Python CLI Operation Pattern
```python
elif args.operation == 'operation_name':
    # Parse input data
    data = json.loads(args.data)
    # Execute operation
    result = ops.operation_method(args.sheet_id, data)
    # Return JSON response
    print(json.dumps(result, indent=2))
```

## Phase 2 Features: Comments, Discussions & Cell History

### Discussion Management
- **Create Discussion**: Start threaded conversations on sheets or rows
- **Add Comments**: Reply to existing discussions with text and attachments
- **List Discussions**: Retrieve all discussions with optional comment inclusion
- **Get Comments**: Fetch all comments in a thread with attachment details
- **Delete Comments**: Remove specific comments (with proper permissions)

### Cell History & Audit Tracking  
- **Cell History**: Complete modification history for individual cells
- **Row History**: Timeline view of all changes across row columns
- **User Attribution**: Track who made changes and when
- **Formula Tracking**: History includes formula changes and calculations

### Cross-Sheet References (Phase 3 - v0.3.0)
- **Reference Analysis**: Identify and analyze all cross-sheet formulas in sheets
- **Dependency Mapping**: Find all sheets that reference a specific target sheet
- **Link Validation**: Check for broken cross-sheet references and suggest fixes
- **Formula Generation**: Create INDEX_MATCH, VLOOKUP, SUMIF, COUNTIF formulas programmatically
- **Impact Analysis**: Understand cross-sheet dependencies across workspaces
- **Custom Templates**: Support for custom formula patterns and advanced references

### Healthcare Use Cases Enabled
- **Clinical Notes**: Threaded discussions on patient records with full audit trails
- **Case Reviews**: Team collaboration with file attachments and change tracking
- **Compliance**: Complete audit history for regulatory requirements (HIPAA, FDA)
- **Research**: Collaborative discussions on research data with version history
- **Multi-Center Studies**: Cross-sheet references for linking patient data across hospital systems
- **Protocol Tracking**: Reference master protocol sheets from multiple study sites

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [terilios/smartsheet-server](https://github.com/terilios/smartsheet-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
