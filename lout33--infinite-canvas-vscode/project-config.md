---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Infinite Canvas VS Code Extension** - a visual canvas editor with full Obsidian Canvas compatibility. It enables users to create infinite mind maps, diagrams, and visual workflows with AI-powered content generation and markdown support.

**Key Components:**
- **VS Code Extension** (`src/extension.ts`): Custom editor for `.canvas` files with webview integration
- **Canvas Web App** (`webview/`): Browser-based canvas implementation with AI features
- **OpenRouter AI Integration**: Multi-model AI content generation via OpenRouter API

## Development Commands

### Build & Compilation
```bash
# Install dependencies
npm install

# Compile TypeScript (one-time)
npm run compile

# Watch mode for development
npm run watch
```

### Testing & Development
```bash
# Launch Extension Development Host
# Use Command Palette: Cmd+Shift+P → "Debug: Start Debugging"
# OR Run > Start Debugging

# Reload extension after changes
# Cmd+R in Extension Development Host window
```

### Package Extension
```bash
# Install packaging tool
npm install -g @vscode/vsce

# Package extension
vsce package
```

## Architecture Overview

### Extension Architecture
```
VS Code Extension Host (Node.js)
├── src/extension.ts              # Main extension logic
│   ├── CanvasEditorProvider      # Custom text editor for .canvas files
│   ├── File I/O operations       # Load/save .canvas files
│   └── Webview management        # Create and manage webview panels
│
└── Webview (Browser Environment)
    ├── webview/main.js           # VS Code integration bridge
    ├── webview/style.css         # VS Code themed styles
    └── webview/src/
        ├── InfiniteCanvasSimple.js  # Core canvas implementation
        ├── AIManager.js             # AI content generation
        ├── aiService.js             # OpenRouter API integration
        ├── markdownRenderer.js      # Markdown rendering
        └── markdownParser.js        # Markdown parsing
```

### Data Flow
```
.canvas file ↔ Extension Host ↔ Webview ↔ Canvas App
                    ↕
               File System Operations
```

### File Format
Uses standard Obsidian Canvas format (.canvas files):
```json
{
  "nodes": [
    {
      "id": "unique-id",
      "x": 100, "y": 100,
      "width": 250, "height": 60,
      "type": "text|file",
      "text": "content",
      "file": "path/to/file.md"  // for file nodes
    }
  ],
  "edges": [
    {
      "id": "edge-id",
      "fromNode": "node-1", "fromSide": "bottom",
      "toNode": "node-2", "toSide": "top"
    }
  ]
}
```

## Key Implementation Details

### Path Handling
- **Critical**: All file paths are normalized to relative paths from workspace root
- `normalizeToRelativePath()` method handles path conversion for compatibility
- Supports legacy absolute paths by converting them to relative paths

### VS Code Integration
- Custom editor for `.canvas` files via `registerCustomEditorProvider`
- Webview messaging system for bidirectional communication:
  - `loadContent`: Extension → Webview (file loading)
  - `save`: Webview → Extension (content saving)
  - `loadFile`/`saveFile`: File node content operations

### AI Integration
- OpenRouter API integration for multiple AI models (GPT-4, Claude, Gemini)
- Configuration via `infinite-canvas.groqApiKey` setting
- Fallback to mock responses when API key not provided
- Context-aware generation using connected nodes as conversation history

### Canvas Interaction Model
- **Double-click empty space**: Create new text node
- **Double-click node**: Edit node content
- **Shift + drag**: Create connections between nodes
- **Drag**: Move nodes or pan canvas
- **Mouse wheel**: Zoom in/out
- **Delete key**: Remove selected nodes/connections

## Configuration & Settings

### VS Code Settings
```json
{
  "infinite-canvas.groqApiKey": "your-openrouter-api-key"
}
```

### Environment Variables (Fallback)
- `GROQ_API_KEY`
- `VITE_GROQ_API_KEY`

## Testing Strategy

### Manual Testing Workflow
1. Create test `.canvas` files in workspace
2. Test basic operations: create, edit, delete, save, load
3. Verify Obsidian compatibility with existing canvas files
4. Test AI generation features with different node types
5. Validate file node operations (markdown editing)

### Canvas Debugging
Enable debug logging in `InfiniteCanvasSimple.js` by setting `DEBUG_MODE = true`

Look for key debug messages:
- `🎨 Initializing VS Code Canvas App`
- `📍 Canvas element found`
- `🖱️ Double click detected!`
- `➕ Creating new node at`

## Common Issues & Solutions

### Extension Development
- **TypeScript errors**: Run `npm run compile` to check compilation
- **Extension not loading**: Check VS Code developer console
- **Canvas not rendering**: Check webview developer tools (right-click → inspect)
- **File operations failing**: Check workspace folder configuration and file paths

### AI Features
- **No API key**: Extension falls back to mock responses
- **AI generation fails**: Check network connectivity and API key validity
- **File content not loading**: Verify file paths are relative to workspace root

### Mac-Specific Development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lout33/infinite_canvas_vscode](https://github.com/lout33/infinite_canvas_vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
