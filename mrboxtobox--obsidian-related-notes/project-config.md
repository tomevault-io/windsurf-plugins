---
trigger: always_on
description: - Use **Global English** for worldwide audience
---

# Claude Code Memory for obsidian-related-notes

## Style Guide & Documentation Standards

### Language & Terminology
- Use **Global English** for worldwide audience
- **Preferred terms:**
  - "keyboard shortcut" over "hotkey" (except when referring to the Hotkey feature)
  - "sync/syncing" over "synchronise/synchronising" 
  - "search term" over "search query"
  - "heading" over "header" (for text sections)
  - "maximum/minimum" over "max/min"
  - "select" over "tap/click"
  - "sidebar" over "side bar"
  - "perform" over "invoke/execute" (for commands)
  - "note" for Markdown files, "file" for other extensions
  - "folder" over "directory"
  - "active note" over "current note"

### UI & Formatting
- Use **sentence case** for headings, buttons, titles
- Use **bold** for button text in documentation
- Use → (U+2192) for UI interaction sequences: "Settings → Community plugins"
- Hyphenate directional terms as adjectives: "bottom-left corner"
- Prefer "upper-left/right" over "top-left/right"
- Use realistic examples over nonsense terms
- Include character in parentheses after key names: "hyphen (-)"

### Code & Documentation
- Use newlines between Markdown blocks
- Images should be .png or .svg format
- Use descriptive anchor tags: `#icon`, `#interface`, `#outline`
- Keep descriptions ≤150 characters for SEO
- Translate all content including filenames, aliases, links

## Plugin Development Fundamentals

### Basic Plugin Structure
```typescript
import { Plugin } from 'obsidian';

export default class ExamplePlugin extends Plugin {
  async onload() {
    // Configure resources needed by the plugin
  }
  
  async onunload() {
    // Release any resources configured by the plugin
  }
}
```

### Plugin Lifecycle
- **onload()**: Runs when plugin starts - configure capabilities here
- **onunload()**: Runs when plugin disabled - clean up resources

### Development Workflow
- Use Hot-Reload plugin for automatic reloading
- Access Developer Tools: `Ctrl+Shift+I` (Windows/Linux) or `Cmd-Option-I` (macOS)
- Reload manually: disable then re-enable in Community plugins

## Core APIs & Components

### Commands
```typescript
// Basic command
this.addCommand({
  id: 'example-command',
  name: 'Example command',
  callback: () => {
    console.log('Command executed');
  }
});

// Conditional command
this.addCommand({
  id: 'conditional-command',
  name: 'Conditional command',
  checkCallback: (checking: boolean) => {
    const canRun = someCondition();
    if (canRun && !checking) {
      executeAction();
    }
    return canRun;
  }
});

// Editor command
this.addCommand({
  id: 'editor-command',
  name: 'Editor command',
  editorCallback: (editor: Editor, view: MarkdownView) => {
    const selection = editor.getSelection();
    // Work with editor
  }
});
```

### Settings
```typescript
interface MyPluginSettings {
  setting1: string;
  setting2: boolean;
}

const DEFAULT_SETTINGS: Partial<MyPluginSettings> = {
  setting1: 'default value',
  setting2: true
};

// In plugin class
settings: MyPluginSettings;

async onload() {
  await this.loadSettings();
  this.addSettingTab(new MySettingTab(this.app, this));
}

async loadSettings() {
  this.settings = Object.assign({}, DEFAULT_SETTINGS, await this.loadData());
}

async saveSettings() {
  await this.saveData(this.settings);
}
```

### Views
```typescript
export const VIEW_TYPE_EXAMPLE = 'example-view';

export class ExampleView extends ItemView {
  constructor(leaf: WorkspaceLeaf) {
    super(leaf);
  }

  getViewType() {
    return VIEW_TYPE_EXAMPLE;
  }

  getDisplayText() {
    return 'Example view';
  }

  async onOpen() {
    const container = this.containerEl.children[1];
    container.empty();
    container.createEl('h4', { text: 'Example view' });
  }

  async onClose() {
    // Clean up resources
  }
}

// Register view
this.registerView(VIEW_TYPE_EXAMPLE, (leaf) => new ExampleView(leaf));
```

### Events
```typescript
// Vault events
this.registerEvent(this.app.vault.on('create', (file) => {
  console.log('File created:', file.path);
}));

// Workspace events
this.registerEvent(this.app.workspace.on('file-menu', (menu, file) => {
  menu.addItem((item) => {
    item.setTitle('Custom action').onClick(() => {
      // Handle click
    });
  });
}));

// Timing events
this.registerInterval(
  window.setInterval(() => this.updateStatusBar(), 1000)
);
```

### Vault Operations
```typescript
// Read files
const content = await this.app.vault.cachedRead(file); // For display
const content = await this.app.vault.read(file); // For modification

// Modify files
await this.app.vault.modify(file, newContent);

// Process files (atomic modification)
await this.app.vault.process(file, (data) => {
  return data.replace('old', 'new');
});

// Get files
const files = this.app.vault.getMarkdownFiles();
const file = this.app.vault.getFileByPath('path/to/file.md');
const folder = this.app.vault.getFolderByPath('path/to/folder');
```

## Best Practices

### Resource Management
- Use `registerEvent()`, `addCommand()`, etc. for automatic cleanup
- Clean up manually created resources in `onunload()`
- Don't detach leaves in `onunload()` (breaks user experience)

### Security
- Avoid `innerHTML`, `outerHTML`, `insertAdjacentHTML`
- Use `createEl()`, `createDiv()`, `createSpan()` instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrboxtobox/obsidian-related-notes](https://github.com/mrboxtobox/obsidian-related-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
