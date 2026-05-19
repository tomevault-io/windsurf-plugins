---
trigger: always_on
description: This is a **monorepo of Model Context Protocol (MCP) servers** providing media handling, information retrieval, and document creation capabilities. Each server is a separate, independently deployable process that communicates via stdio transport.
---

# Agentic AI Tool Suite - AI Coding Agent Instructions

## Project Overview

This is a **monorepo of Model Context Protocol (MCP) servers** providing media handling, information retrieval, and document creation capabilities. Each server is a separate, independently deployable process that communicates via stdio transport.

### Architecture Pattern: Multi-Language MCP Server Suite

- **TypeScript Servers** (`information-retrieval-server`, `media-tools-server`, `presentation-creator-server`): Use `@modelcontextprotocol/sdk`, Zod validation, stdio transport
- **Python Servers** (`pdf-creator-server`, `presentation-creator-server`): Use `fastmcp` library, async/await patterns
- **Dual Implementation**: `presentation-creator-server` exists in both TypeScript (wrapper) and Python (implementation) - Python version contains the actual logic

## Critical Developer Workflows

### Building TypeScript Servers

**Standard build process** (applies to all TS servers):
```bash
cd <server-directory>
npm install
npm run build  # Compiles TS → JS, bundles with esbuild, sets executable permissions
```

The `build` script in `package.json` performs three steps:
1. `tsc` - TypeScript compilation to `build/` directory
2. `esbuild` - Bundles into single file with external packages
3. `chmod 755` - Makes `build/index.js` executable for shebang execution

**Development workflow**:
```bash
npm run watch        # Auto-rebuild on changes
npm run inspector    # Debug with MCP Inspector (opens browser UI)
```

### Python Server Setup

**Virtual environment required**:
```bash
cd <python-server-directory>
python -m venv .venv
.\.venv\Scripts\activate  # Windows PowerShell
pip install -r requirements.txt
playwright install chromium  # Required for PDF/presentation generation
```

**Important**: MCP client configs must point to the venv's Python executable, not system Python:
```json
{
  "command": "C:/path/to/.venv/Scripts/python.exe",
  "args": ["C:/path/to/server_script.py"]
}
```

### Publishing Workflow

**TypeScript servers** are published to npm and can be run via `npx`:
- Published packages: `information-retrieval-mcp-server`, `media-tools-mcp-server`, `presentation-creator-mcp-server`
- Users can run with: `npx -y <package-name>` (no installation needed)
- The `prepare` script in package.json auto-builds before publishing

**Python servers** are packaged but not yet published to PyPI (see `PUBLISHED_PACKAGES.md`)

## Project-Specific Conventions

### Environment Variables Pattern

All servers follow this pattern:
1. Check for required API keys at startup (log warnings, don't exit)
2. Validate keys when tools are called (return user-friendly error messages)
3. Support fallback env var names (e.g., `GEMINI_API_KEY` OR `GOOGLE_API_KEY`)

Example from `information-retrieval-server/src/index.ts`:
```typescript
const GOOGLE_API_KEY = process.env.GOOGLE_API_KEY;
if (!GOOGLE_API_KEY) {
  console.error("WARN: GOOGLE_API_KEY not set. Tools will fail.");
}
```

### Tool Implementation Pattern

**TypeScript servers** use this structure:
```typescript
// 1. Define Zod schema for validation
const toolArgsSchema = z.object({
  param: z.string().describe('Description for LLM'),
});

// 2. Implement handler function
async function handleTool({ param }: { param: string }) {
  try {
    // Tool logic here
    return { content: [{ type: 'text', text: 'Result' }] };
  } catch (error: any) {
    return { isError: true, content: [{ type: 'text', text: error.message }] };
  }
}

// 3. Register in ListToolsRequestSchema handler
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{ name: 'tool_name', description: '...', inputSchema: {...} }]
}));

// 4. Route in CallToolRequestSchema handler
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const validatedArgs = toolArgsSchema.parse(request.params.arguments);
  return await handleTool(validatedArgs);
});
```

**Python servers** use FastMCP decorators:
```python
@mcp.tool()
async def tool_name(param: str, optional_param: str = "default") -> str:
    """Docstring becomes tool description for LLM."""
    try:
        # Tool logic
        return "Success message"
    except Exception as e:
        return f"Error: {e}"
```

### Output File Conventions

All servers save output files to `~/Downloads` folder:
- PDFs: `~/Downloads/<filename>.pdf`
- Presentations: `~/Downloads/<filename>.pptx`
- Downloaded images: User-specified path (validated for image extensions)

### Temporary File Handling

Python servers use this pattern:
```python
temp_dir = tempfile.mkdtemp(prefix="mcp_<server>_")
try:
    # Use temp_dir for intermediate files
    pass
finally:
    if temp_dir and os.path.exists(temp_dir):
        shutil.rmtree(temp_dir)
```

### Error Handling Philosophy

- **User-facing errors**: Return helpful messages explaining what went wrong and how to fix it
- **API errors**: Parse and format API error responses (code, message)
- **Never exit the server process** on API errors - return error content to the client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Godzilla675/agentic-ai-tool-suite](https://github.com/Godzilla675/agentic-ai-tool-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
