---
trigger: always_on
description: Design patterns (Decorator, Strategy, Facade) and anti-patterns used in Camouflage codebase
---


# Design Patterns and Anti-Patterns

## Recommended Patterns

### 1. Decorator Pattern (Method Decorators)

**Purpose**: Add cross-cutting concerns without modifying core logic.

**Implementation**:

```typescript
// lib/decorators.ts
export function Log(message: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      console.log(`[${message}] Executing ${propertyKey}`);
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

// Usage
export class Camouflage {
  @Log('Camouflage')
  @HandleErrors
  @Debounce(100)
  private updateDecorations(): void {
    // Core logic only, concerns handled by decorators
  }
}
```

**Benefits**:

- ✅ Separation of concerns
- ✅ Reusable across methods
- ✅ Easy to add/remove
- ✅ Clean core logic

**When to Use**:

- Logging
- Error handling
- Performance monitoring
- Debouncing
- Validation

### 2. Facade Pattern (Configuration)

**Purpose**: Simplify complex API into simple interface.

**Implementation**:

```typescript
// utils/config.ts - Facade
export function isEnabled(): boolean {
  return vscode.workspace.getConfiguration('camouflage').get<boolean>('enabled', true);
}

export function getHidingStyle(): HiddenTextStyle {
  return vscode.workspace
    .getConfiguration('camouflage')
    .get<HiddenTextStyle>('hiddenTextStyle', 'text');
}

export function getPatterns(): string[] {
  return vscode.workspace
    .getConfiguration('camouflage')
    .get<string[]>('selectiveHiding.patterns', []);
}

// Usage - Simple!
import * as config from '../utils/config';

if (config.isEnabled()) {
  const style = config.getHidingStyle();
}
```

**Benefits**:

- ✅ Hides VS Code API complexity
- ✅ Consistent access pattern
- ✅ Easy to mock in tests
- ✅ Single place for default values

**When to Use**:

- Configuration access
- API wrappers
- Complex subsystems

### 3. Strategy Pattern (Text Generation)

**Purpose**: Different algorithms for same task, selectable at runtime.

**Implementation**:

```typescript
// lib/text-generator.ts
type GeneratorStrategy = (text: string) => string;

const strategies: Record<HiddenTextStyle, GeneratorStrategy> = {
  text: (text) => 'camouflage'.repeat(Math.ceil(text.length / 10)),
  stars: (text) => '*'.repeat(text.length),
  dotted: (text) => '•'.repeat(text.length),
  scramble: (text) => scrambleText(text),
};

export function generateHiddenText(text: string, style: HiddenTextStyle): string {
  const strategy = strategies[style];
  if (!strategy) {
    throw new Error(`Unknown style: ${style}`);
  }
  return strategy(text);
}
```

**Benefits**:

- ✅ Easy to add new styles
- ✅ No if/else chains
- ✅ Each strategy independently testable
- ✅ Clear separation

**When to Use**:

- Multiple algorithms for same task
- Behavior selected at runtime
- Open/Closed principle (open for extension, closed for modification)

### 4. Observer Pattern (Event Handling)

**Purpose**: React to state changes.

**Implementation**:

```typescript
// VS Code already provides this
vscode.workspace.onDidChangeConfiguration((event) => {
  if (event.affectsConfiguration('camouflage')) {
    // React to configuration change
    this.reloadConfiguration();
  }
});

vscode.window.onDidChangeActiveTextEditor((editor) => {
  if (editor && isEnvFile(editor.document.fileName)) {
    // React to editor change
    this.updateDecorations();
  }
});
```

**Benefits**:

- ✅ Loose coupling
- ✅ Reactive architecture
- ✅ Easy to add listeners

**When to Use**:

- Event-driven architecture
- State synchronization
- Multiple components need updates

### 5. Singleton Pattern (Extension Instance)

**Purpose**: Single instance of extension core.

**Implementation**:

```typescript
// extension.ts
let camouflage: Camouflage | undefined;

export function activate(context: vscode.ExtensionContext): void {
  if (!camouflage) {
    camouflage = new Camouflage();
    camouflage.initialize(context);
  }
}

export function deactivate(): void {
  if (camouflage) {
    camouflage.dispose();
    camouflage = undefined;
  }
}
```

**Benefits**:

- ✅ Single source of truth
- ✅ Controlled access
- ✅ Resource management

**When to Use**:

- Extension activation
- Global state management
- Resource coordination

### 6. Factory Pattern (Decoration Creation)

**Purpose**: Create complex objects with consistent configuration.

**Implementation**:

```typescript
export class DecorationFactory {
  static createDecorationType(
    style: HiddenTextStyle,
    color: string
  ): vscode.TextEditorDecorationType {
    const baseConfig = {
      color,
      textDecoration: 'none',
    };

    const styleConfig = this.getStyleConfig(style);

    return vscode.window.createTextEditorDecorationType({
      ...baseConfig,
      ...styleConfig,
    });
  }

  private static getStyleConfig(style: HiddenTextStyle) {
    switch (style) {
      case 'text':
        return {
          letterSpacing: '-0.5em',
          opacity: '0',
        };
      case 'dotted':
        return {
          letterSpacing: '0.3em',
          before: { contentText: '•' },
        };
      // ... other styles
    }
  }
}
```

**Benefits**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
