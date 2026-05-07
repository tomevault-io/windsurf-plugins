---
trigger: always_on
description: **MarkBun** is a Typora-like markdown desktop editor built with:
---

# AGENTS.md - MarkBun Development Guide

## Project Overview

**MarkBun** is a Typora-like markdown desktop editor built with:
- **Milkdown**: WYSIWYG markdown editor framework
- **Electrobun**: Bun + WebView cross-platform desktop framework  
- **shadcn/ui**: React component library
- **TypeScript**: Type-safe development

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│                      MarkBun Application                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    IPC      ┌──────────────────────────────┐  │
│  │  Main Process│◄───────────►│     Renderer Process         │  │
│  │  (Electrobun)│   (JSON)    │     (WebView + React)        │  │
│  │  Bun Runtime │             │                              │  │
│  └──────────────┘             │  ┌────────────────────────┐   │  │
│         │                     │  │     Milkdown Editor    │   │  │
│         ▼                     │  │  ┌──────────────────┐  │   │  │
│  ┌──────────────┐             │  │  │   ProseMirror    │  │   │  │
│  │   OS APIs    │             │  │  │   Document Model │  │   │  │
│  │  File System │             │  │  └──────────────────┘  │   │  │
│  │    Menus     │             │  └────────────────────────┘   │  │
│  │   Windows    │             │                               │  │
│  └──────────────┘             │  ┌────────────────────────┐   │  │
│                               │  │    shadcn/ui Components│   │  │
│                               │  │  - Toolbar             │   │  │
│                               │  │  - Sidebar             │   │  │
│                               │  │  - Status Bar          │   │  │
│                               │  │  - Dialogs             │   │  │
│                               │  └────────────────────────┘   │  │
│                               └───────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Tech Stack Details

### 1. Milkdown

Milkdown is a plugin-driven WYSIWYG markdown editor built on ProseMirror.

**Key Concepts:**
- **Editor**: Core class that manages the editor lifecycle
- **Plugins**: Everything is a plugin (syntax, themes, UI features)
- **Ctx**: Context system for sharing data between plugins
- **ProseMirror**: Underlying document model and editing engine
- **Remark**: Markdown parser/serializer

**Usage Pattern for MarkBun:**
```typescript
import { Editor } from "@milkdown/kit/core";
import { commonmark } from "@milkdown/kit/preset/commonmark";
import { gfm } from "@milkdown/kit/preset/gfm";
import { history } from "@milkdown/kit/plugin/history";

Editor.make()
  .use(commonmark)
  .use(gfm)
  .use(history)
  .create();
```

**Required Plugins:**
- `@milkdown/kit/preset/commonmark` - Basic markdown
- `@milkdown/kit/preset/gfm` - GitHub Flavored Markdown
- `@milkdown/kit/plugin/history` - Undo/redo
- `@milkdown/kit/plugin/clipboard` - Copy/paste
- `@milkdown/plugin-math` - LaTeX math
- `@milkdown/plugin-slash` - Command palette

### 2. Electrobun

Electrobun is a desktop framework using Bun as the backend and native WebView as the renderer.

**Key Concepts:**
- **BrowserWindow**: Native window management
- **BrowserView**: WebView containers
- **Electroview**: Browser-side API to communicate with main process
- **views:// protocol**: Custom protocol for loading app files

**File Structure:**
```
src/
├── bun/              # Main process
│   └── index.ts      # Entry point, window creation
└── main-ui/          # Renderer process
    ├── index.html    # HTML entry
    └── index.ts      # React app entry
```

**Main Process Example:**
```typescript
import { BrowserWindow, ApplicationMenu } from "electrobun/bun";

// Create window
const win = new BrowserWindow({
  title: "MarkBun",
  url: "views://main-ui/index.html",
  width: 1200,
  height: 800,
});

// Set up menus
ApplicationMenu.setApplicationMenu([
  {
    label: "File",
    submenu: [
      { label: "New", action: "file-new" },
      { label: "Open", action: "file-open" },
      { label: "Save", action: "file-save" },
    ],
  },
]);
```

**Renderer Process Example:**
```typescript
import { Electroview } from "electrobun/view";

const electrobun = new Electroview({ rpc: null });

// Listen for menu actions
electrobun.on("menu-action", (action) => {
  if (action === "file-save") {
    saveDocument();
  }
});

// Call main process
electrobun.rpc("saveFile", { content: "..." });
```

### 3. shadcn/ui

Component library with excellent TypeScript support and customization.

**Key Principles:**
- Components are copied to your project (not dependencies)
- Use `npx shadcn@latest add <component>` to add components
- Built on Radix UI primitives
- Styled with Tailwind CSS

**Critical Rules:**
- Use semantic colors: `bg-primary`, `text-muted-foreground`
- Use `cn()` for conditional classes
- Use `flex gap-*` instead of `space-y-*`
- Use `size-*` for equal dimensions
- Forms use `FieldGroup` + `Field` structure

**Components for MarkBun:**
- `Button` - Toolbar buttons
- `Tooltip` - Button hints
- `Dialog` - Settings, modals
- `DropdownMenu` - File menu, insert menu

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaochong/markbun](https://github.com/xiaochong/markbun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
