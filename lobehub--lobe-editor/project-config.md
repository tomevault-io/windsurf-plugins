---
trigger: always_on
description: Comprehensive plugin development, architecture, and refactoring guidelines for LobeHub Editor
---


# Plugin Development Guidelines

## Plugin Architecture Overview

LobeHub Editor follows a **dual-layer architecture** with framework-agnostic core and React-specific implementations.

### Plugin Structure Convention

Each plugin in [src/plugins/](mdc:src/plugins/) follows this structure:

```
plugin-name/
├── index.ts          # Main exports
├── index.md          # Documentation
├── plugin/           # Core plugin logic
│   ├── index.ts      # Plugin class
│   └── registry.ts   # Commands and hotkeys
├── react/            # React components (if applicable)
├── command/          # Editor commands
├── service/          # Business logic services
├── node/             # Custom Lexical nodes
├── utils/            # Utility functions
└── demos/            # Demo examples
```

## Plugin Implementation Pattern

### Base Plugin Class

All plugins extend [KernelPlugin](mdc:src/editor-kernel/plugin.ts):

```typescript
import { KernelPlugin } from '@/editor-kernel/plugin';
import { IEditorKernel, IEditorPlugin, IEditorPluginConstructor } from '@/types';
import { createDebugLogger } from '@/utils/debug';

export const MyPlugin: IEditorPluginConstructor<MyPluginOptions> = class
  extends KernelPlugin
  implements IEditorPlugin<MyPluginOptions>
{
  static pluginName = 'MyPluginName'; // Always follow naming convention
  private logger = createDebugLogger('plugin', 'my-plugin');

  constructor(
    protected kernel: IEditorKernel,
    public config?: MyPluginOptions, // Make config public for registry access
  ) {
    super();
    
    // Register nodes, services, themes
    kernel.registerNodes([MyCustomNode]);
    kernel.registerService(IMyService, new MyService());
    
    if (config?.theme) {
      kernel.registerThemes(config.theme);
    }
    
    // Register decorators for custom nodes
    this.registerDecorator(
      kernel,
      MyCustomNode.getType(),
      (node: DecoratorNode<any>, editor: LexicalEditor) => {
        return config?.decorator ? config.decorator(node as MyCustomNode, editor) : null;
      },
    );
  }

  onInit(editor: LexicalEditor): void {
    // Register commands, listeners, observers via registry
    this.register(registerMyCommands(editor, this.kernel, {
      enableHotkey: this.config?.enableHotkey,
      // other options from config
    }));
  }
};
```

## Registry Pattern for Commands and Hotkeys

### Registry Function Structure

Create `plugin/registry.ts` for centralized command and hotkey management:

```typescript
import { mergeRegister } from '@lexical/utils';
import { LexicalEditor } from 'lexical';
import { IEditorKernel } from '@/types';
import { HotkeyEnum } from '@/types/hotkey';

export interface PluginRegistryOptions {
  enableHotkey?: boolean;
  // Plugin-specific options in alphabetical order
}

export function registerPluginCommands(
  editor: LexicalEditor,
  kernel: IEditorKernel,
  options?: PluginRegistryOptions,
) {
  const { enableHotkey = true } = options || {};
  
  return mergeRegister(
    // Core commands (always active)
    editor.registerCommand(COMMAND, handler, PRIORITY),
    
    // Update listeners for state tracking
    editor.registerUpdateListener(() => {
      // State management logic
    }),
    
    // Hotkeys (conditionally enabled)
    kernel.registerHotkey(
      HotkeyEnum.Action,
      () => editor.dispatchCommand(COMMAND, payload),
      {
        enabled: enableHotkey,
        preventDefault: true,
        stopPropagation: true,
      },
    ),
  );
}
```

### What Goes in Registry vs React Components

**Move TO Registry:**

1. **Hotkey Registration** - `kernel.registerHotkey()`
2. **Core Editor Commands** - `editor.registerCommand()`
3. **Update Listeners for Business Logic** - State tracking, validation
4. **Keyboard Event Handlers** - Framework-agnostic keyboard logic

**Keep in React Components:**

1. **UI State Management** - React state and effects
2. **DOM Manipulation** - Positioning, floating UI
3. **React-Specific Commands** - Commands that update React state

## Service Pattern

### Service Definition

Services should implement interfaces in `service/`:

```typescript
import { genServiceId } from '@/editor-kernel/service';
import { IServiceID } from '@/types';

export interface IMyService {
  method1(param: Type): ReturnType;
  method2(param: Type): Promise<ReturnType>;
}

export const IMyService: IServiceID<IMyService> = 
  genServiceId<IMyService>('MyService');

export class MyService implements IMyService {
  private logger = createDebugLogger('service', 'my-service');
  
  method1(param: Type): ReturnType {
    this.logger.debug('Method1 called with:', param);
    // Implementation
  }
}
```

### Service Registration

Register services in plugin constructor:

```typescript
constructor(protected kernel: IEditorKernel, public config?: Options) {
  super();
  kernel.registerService(IMyService, new MyService());
}
```

## Command Pattern

### Command Definition

Define commands in `command/index.ts`:

```typescript
import { createCommand, LexicalEditor, COMMAND_PRIORITY_HIGH } from 'lexical';

export const MY_COMMAND = createCommand<PayloadType>('MY_COMMAND');

export function registerMyCommand(editor: LexicalEditor) {
  return editor.registerCommand(
    MY_COMMAND,
    (payload) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lobehub/lobe-editor](https://github.com/lobehub/lobe-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
