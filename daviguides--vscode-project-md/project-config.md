---
trigger: always_on
description: **Project.md** is a VSCode extension that transforms Markdown files into interactive project navigation hubs. It provides intelligent file path detection, clickable navigation, and automatic file creation - specifically designed to enhance workflow for code assistants like Claude Code, Gemini CLI, and Codex CLI.
---

# Project.md - Technical Documentation for Claude Code

## Project Overview

**Project.md** is a VSCode extension that transforms Markdown files into interactive project navigation hubs. It provides intelligent file path detection, clickable navigation, and automatic file creation - specifically designed to enhance workflow for code assistants like Claude Code, Gemini CLI, and Codex CLI.

### Core Purpose

When working with code assistants, developers often reference project files in Markdown documentation. This extension makes those references **actionable**:
- File paths become clickable links
- Non-existent files are created automatically
- Navigation works seamlessly with VSCode's native features

## Architecture

### Extension Structure

```
project-md/
├── src/
│   ├── extension.ts          # Main extension logic
│   └── test/
│       └── extension.test.ts # Test suite (placeholder)
├── dist/                      # Compiled output (esbuild)
├── package.json              # Extension manifest
└── tsconfig.json             # TypeScript configuration
```

### Key Components

#### 1. **Activation Events**
```json
"activationEvents": ["onLanguage:markdown"]
```
- Extension activates when any Markdown file opens
- Lazy activation for performance optimization
- No activation on VSCode startup (reduces overhead)

#### 2. **Providers Registered**

**a) Document Link Provider**
- Scans document for file path patterns
- Converts paths to clickable VSCode links
- Uses command URI scheme for custom behavior
- Tooltip: "Open path"

**b) Definition Provider**
- Enables "Go to Definition" (F12) on file paths
- Only works for existing files
- Returns `vscode.Location` pointing to target file

**c) Command Handler**
- Command ID: `markdownLinks.open`
- Handles click events on document links
- Implements auto-creation logic
- Directory handling with fallback

### Path Detection Strategy

Three regex patterns capture different reference styles:

#### Pattern 1: Markdown Links
```typescript
const MD_LINK_RE = /\[[^\]]+\]\(\s*(@?(?:\.{1,2}\/|\/)[^)\s]+)\s*\)/g;
```

**Matches:**
- `[text](./path/file.md)`
- `[config](@./config/settings.ts)`
- `[root](/absolute/path.ts)`

**Captures:** Path inside parentheses, including optional `@` prefix

#### Pattern 2: Bare References
```typescript
const BARE_REF_RE = /(^|[\s(`])(@?(?:\.{1,2}\/|\/)[^\s`)\]]+)/g;
```

**Matches:**
- `./src/index.ts` (standalone)
- `Check ./config/app.ts for details`
- Line-beginning paths

**Captures:** Leading whitespace/delimiter + path

**Conflict Resolution:** Only adds if not already captured by MD_LINK_RE

#### Pattern 3: Inline Code References
```typescript
const INLINE_CODE_RE = /`(@?(?:\.{1,2}\/|\/)[^\s`)\]]+)`/g;
```

**Matches:**
- `` `./src/utils.ts` ``
- `` `@./config/database.ts` ``

**Captures:** Path inside backticks

**Conflict Resolution:** Checks for range intersection before adding

### Path Resolution Logic

```typescript
const abs = path.resolve(baseDir, rel.replace(/^@/, ""));
```

**Steps:**
1. Get document's directory (`path.dirname(doc.uri.fsPath)`)
2. Remove `@` prefix if present
3. Resolve relative path to absolute
4. Convert to filesystem path

**@ Prefix Behavior:**
- Stripped before resolution
- Allows aliasing convention (e.g., `@./` for project root contexts)
- No special mapping (yet) - treated as `./`

## Feature Implementation Details

### Feature 1: Clickable Links

**Implementation:**
```typescript
const linkProvider: vscode.DocumentLinkProvider = {
  provideDocumentLinks(doc) {
    return getRefRanges(doc).map(({ range, targetFsPath }) => {
      const cmdUri = vscode.Uri.parse(
        `command:${commandId}?${encodeURIComponent(JSON.stringify({ path: targetFsPath }))}`
      );
      const link = new vscode.DocumentLink(range, cmdUri);
      link.tooltip = "Open path";
      return link;
    });
  }
};
```

**Flow:**
1. User hovers over detected path
2. VSCode renders underline (document link)
3. Cmd/Ctrl+Click triggers command URI
4. Command handler (`markdownLinks.open`) executes
5. `openPathLike()` processes the path

### Feature 2: Auto-Creation

**Implementation:**
```typescript
async function openPathLike(targetFsPath: string) {
  const stat = await fs.promises.stat(targetFsPath).catch(() => undefined);

  if (!stat) {
    await fs.promises.mkdir(path.dirname(targetFsPath), { recursive: true });
    await fs.promises.writeFile(targetFsPath, "", "utf8");
  }

  const td = await vscode.workspace.openTextDocument(vscode.Uri.file(targetFsPath));
  await vscode.window.showTextDocument(td);
}
```

**Behavior:**
- **File doesn't exist**: Create parent directories + empty file
- **Directory**: Reveal in Explorer (or Finder/File Explorer as fallback)
- **File exists**: Open in editor
- **Error**: Show error message with path

**Safety:**
- Uses `recursive: true` for `mkdir` (safe if directories exist)
- Creates empty UTF-8 file (no data loss risk)
- Catch-all error handling prevents crashes

### Feature 3: Go to Definition

**Implementation:**
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daviguides) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
