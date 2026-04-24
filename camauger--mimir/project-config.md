---
trigger: always_on
description: **Mimir** is a VS Code/Cursor extension that converts DOCX documents to JSON and HTML formats. It's specifically designed to process pedagogical components, making it ideal for educational content management and documentation workflows.
---

# CLAUDE.md - Mimir Extension Documentation

## Project Overview

**Mimir** is a VS Code/Cursor extension that converts DOCX documents to JSON and HTML formats. It's specifically designed to process pedagogical components, making it ideal for educational content management and documentation workflows.

### Key Features
- Convert DOCX files to HTML with pedagogical component support
- Convert DOCX files to JSON for downstream processing
- Preview DOCX files as HTML directly in the editor
- Flexible image handling (save to disk or base64 encoding)

---

## Architecture

### Technology Stack
- **Runtime**: Node.js (VS Code Extension Host)
- **Language**: JavaScript (extension.js)
- **Backend**: Python script (external process)
- **VS Code API**: ^1.96.0

### Extension Structure

```
mimir/
├── extension.js         # Main extension logic
├── package.json         # Extension manifest and configuration
├── README.md           # User documentation (French)
├── LICENCE.md          # MIT License
├── CLAUDE.md           # This file (AI assistant documentation)
└── mimir-0.1.0.vsix    # Packaged extension
```

### Command Registration

The extension registers three commands:

1. **mimir.convertToHtml** - Convert DOCX to HTML
2. **mimir.convertToJson** - Convert DOCX to JSON
3. **mimir.previewHtml** - Preview DOCX as HTML in webview

All commands are available via right-click context menu on `.docx` files in the explorer.

---

## How It Works

### Conversion Flow

```
User Action → VSCode Command → exec() Python Script → Output File → User Notification
```

1. **File Selection**: User right-clicks a DOCX file or triggers command without selection
2. **Configuration Loading**: Extension reads user settings for Python path, script path, and image handling
3. **Command Execution**: Spawns Python process with appropriate flags
4. **Progress Notification**: Shows VS Code progress notification during conversion
5. **Result Handling**:
   - Success: Shows notification with "Open" button
   - Error: Shows error message with details

### Python Script Integration

The extension calls an external Python script with the following pattern:

```bash
python <scriptPath> <docxPath> --output-dir <dir> [--save-images|--base64-images] [--html|--json]
```

**Expected Script Interface:**
- Accepts DOCX file path as first argument
- `--output-dir`: Specifies output directory
- `--save-images`: Saves images to disk
- `--base64-images`: Encodes images as base64
- `--html`: Outputs HTML format
- `--json`: Outputs JSON format

---

## Configuration

### User Settings

Three configuration options are exposed via `package.json`:

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `mimir.pythonPath` | string | `"python"` | Path to Python executable |
| `mimir.scriptPath` | string | `""` | Path to conversion script |
| `mimir.saveImagesToDisk` | boolean | `true` | Save images vs base64 encoding |

### Setting Locations
- User Settings: `~/.config/Code/User/settings.json`
- Workspace Settings: `.vscode/settings.json`

Example configuration:
```json
{
  "mimir.pythonPath": "/usr/bin/python3",
  "mimir.scriptPath": "/path/to/converter/script.py",
  "mimir.saveImagesToDisk": true
}
```

---

## Development

### Prerequisites
- Node.js (v14 or higher)
- VS Code (v1.96.0 or higher)
- Python 3.x with required conversion libraries

### Setup

1. Clone the repository:
```bash
git clone https://github.com/camauger/mimir.git
cd mimir
```

2. Install dependencies:
```bash
npm install
```

3. Open in VS Code:
```bash
code .
```

4. Press F5 to launch Extension Development Host

### Testing

The extension includes ESLint for code quality:

```bash
npm run lint          # Run linter
npm run pretest       # Lint before testing
npm test              # Run tests
```

### Building

To package the extension:

```bash
vsce package
```

This creates a `.vsix` file that can be installed manually or published to the marketplace.

---

## Code Structure

### extension.js Analysis

The main extension file contains three similar command handlers:

1. **File Selection Logic** (lines 12-32, 85-105, 157-178)
   - Checks if file URI is provided
   - If not, searches workspace for all `.docx` files
   - Shows quick pick dialog for user selection

2. **Configuration Retrieval** (lines 41-44, 114-117, 193-195)
   - Loads user/workspace settings
   - Retrieves Python path and script path
   - Determines image handling mode

3. **Command Execution** (lines 48-80, 121-153, 198-242)
   - Constructs shell command with proper escaping
   - Shows progress notification
   - Executes Python script via `child_process.exec`
   - Handles success/error cases

### Key Implementation Details

**Progress Feedback**: Uses `vscode.window.withProgress` for non-blocking notifications

**Error Handling**: Catches execution errors and displays via `vscode.window.showErrorMessage`

**Preview Mode**:
- Creates temporary `.preview` directory
- Always uses base64 encoding to avoid path issues
- Renders HTML in webview panel (column 2)

**Security Considerations**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/camauger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
