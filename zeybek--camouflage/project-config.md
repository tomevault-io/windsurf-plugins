---
trigger: always_on
description: VS Code extension development rules - lifecycle, API usage, decorations, commands, and configuration
---


# VS Code Extension Development Rules

## Extension Lifecycle

### Activation

The extension activates when specified events occur (defined in `package.json`):

```json
"activationEvents": [
  "onLanguage:dotenv",
  "onLanguage:properties",
  "onLanguage:plaintext",
  "onLanguage:json",
  "onLanguage:yaml",
  "onLanguage:toml",
  "onStartupFinished"
]
```

This enables the extension for all supported configuration file formats.

**Rules**:

- ✅ Activate only when needed (lazy loading)
- ✅ Keep activation time < 200ms
- ❌ Don't activate on `*` (every file)
- ❌ Don't do heavy work in `activate()`

### Activation Function

```typescript
export function activate(context: vscode.ExtensionContext): void {
  // 1. Initialize lightweight components first
  const camouflage = new Camouflage();

  // 2. Register commands
  const toggleCommand = vscode.commands.registerCommand('camouflage.toggle', () =>
    camouflage.toggle()
  );

  // 3. Register all disposables
  context.subscriptions.push(toggleCommand, camouflage);

  // 4. Start extension
  camouflage.initialize(context);
}
```

### Deactivation

```typescript
export function deactivate(): void {
  // Dispose resources not in context.subscriptions
  // Usually not needed if everything is properly registered
}
```

## VS Code API Usage

### Window API

```typescript
// ✅ GOOD: Check for active editor
const editor = vscode.window.activeTextEditor;
if (editor) {
  // Work with editor
}

// ❌ BAD: Assume editor exists
const editor = vscode.window.activeTextEditor!;
editor.document.getText(); // Crash if no editor!

// Status bar
const statusBar = vscode.window.createStatusBarItem(
  vscode.StatusBarAlignment.Right,
  100 // Priority
);
context.subscriptions.push(statusBar); // Always dispose!

// Messages
vscode.window.showInformationMessage('Success!');
vscode.window.showWarningMessage('Warning!');
vscode.window.showErrorMessage('Error!');
```

### Workspace API

```typescript
// Configuration
const config = vscode.workspace.getConfiguration('camouflage');
const isEnabled = config.get<boolean>('enabled', true);

// Update configuration
await config.update('enabled', false, vscode.ConfigurationTarget.Global);

// File system
const files = await vscode.workspace.findFiles('**/.env', '**/node_modules/**');

// Text documents
vscode.workspace.textDocuments.forEach((doc) => {
  // Process each open document
});
```

### Commands API

```typescript
// Register command
const disposable = vscode.commands.registerCommand('camouflage.toggle', async () => {
  // Command implementation
  await toggle();
});

// Execute existing command
await vscode.commands.executeCommand('workbench.action.reloadWindow');

// Pass arguments
await vscode.commands.executeCommand('editor.action.insertSnippet', {
  snippet: 'API_KEY=${1:value}',
});
```

### TextEditor Decorations

```typescript
// Create decoration type
const decorationType = vscode.window.createTextEditorDecorationType({
  color: '#FF0000',
  backgroundColor: '#00000000',
  letterSpacing: '-0.5em',
  opacity: '0',
  textDecoration: 'none; font-size: 0;',
});

// Apply decorations
const ranges = [new vscode.Range(0, 10, 0, 20)];
editor.setDecorations(decorationType, ranges);

// Clear decorations
editor.setDecorations(decorationType, []);

// Dispose when done
decorationType.dispose();
```

## Event Handling

### Document Events

```typescript
// Text document changes (debounce this!)
vscode.workspace.onDidChangeTextDocument((event) => {
  if (event.document === vscode.window.activeTextEditor?.document) {
    updateDecorations();
  }
});

// Editor selection changes
vscode.window.onDidChangeActiveTextEditor((editor) => {
  if (editor && isEnvFile(editor.document.fileName)) {
    updateDecorations();
  }
});

// Configuration changes
vscode.workspace.onDidChangeConfiguration((event) => {
  if (event.affectsConfiguration('camouflage')) {
    reloadConfiguration();
  }
});
```

### Event Disposal

```typescript
// ✅ GOOD: Register for automatic disposal
const disposable = vscode.workspace.onDidChangeTextDocument(handler);
context.subscriptions.push(disposable);

// ❌ BAD: Never disposed
vscode.workspace.onDidChangeTextDocument(handler); // Memory leak!
```

## Performance Best Practices

### Debounce Frequent Events

```typescript
// ✅ GOOD: Debounce text changes
@Debounce(100)
private updateDecorations(): void {
  // Called max once per 100ms
}

vscode.workspace.onDidChangeTextDocument(() => {
  this.updateDecorations();
});

// ❌ BAD: No debouncing
vscode.workspace.onDidChangeTextDocument(() => {
  this.updateDecorations(); // Called on EVERY keystroke!
});
```

### Cache Expensive Operations

```typescript
// ✅ GOOD: Cache decoration type
private decorationType?: vscode.TextEditorDecorationType;

private getDecorationType(): vscode.TextEditorDecorationType {
  if (!this.decorationType) {
    this.decorationType = vscode.window.createTextEditorDecorationType({
      // ... decoration config
    });
  }
  return this.decorationType;
}

// ❌ BAD: Create new decoration type every time
private getDecorationType(): vscode.TextEditorDecorationType {
  return vscode.window.createTextEditorDecorationType({
    // ... decoration config
  }); // Creates new object every call!
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
