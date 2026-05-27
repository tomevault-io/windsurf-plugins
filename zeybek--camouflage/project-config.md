---
trigger: always_on
description: Quick reference for commands, key files, common tasks, testing patterns, and debugging tips
---


# Quick Reference

## Essential Commands

```bash
# Development
npm install          # Install dependencies
npm run compile      # Compile TypeScript
npm run watch        # Watch mode compilation
npm test             # Run tests
npm test -- --watch  # Watch mode tests
npm run lint         # Run ESLint
npm run format       # Format with Prettier

# Extension Keyboard Shortcuts
F5                   # Launch Extension Development Host
Ctrl+Shift+H         # Hide values (Windows/Linux)
Cmd+Shift+H          # Hide values (macOS)
Ctrl+Shift+R         # Reveal values (Windows/Linux)
Cmd+Shift+R          # Reveal values (macOS)
Ctrl+Shift+T         # Toggle value at cursor (Windows/Linux)
Cmd+Shift+T          # Toggle value at cursor (macOS)
Ctrl+Shift+S         # Toggle selective mode (Windows/Linux)
Cmd+Shift+S          # Toggle selective mode (macOS)

# Release (automated)
git commit -m "feat: new feature"  # Semantic commit
git push origin main               # Auto-release on merge
```

## Key Files

| File                       | Purpose                              |
| -------------------------- | ------------------------------------ |
| `src/extension.ts`         | Extension entry point & commands     |
| `src/core/camouflage.ts`   | Main business logic & decorations    |
| `src/parsers/index.ts`     | Parser registry & factory            |
| `src/parsers/base-parser.ts` | Abstract parser base               |
| `src/utils/config.ts`      | Configuration facade & file exclusion|
| `src/utils/file.ts`        | File utilities & format detection    |
| `package.json`             | Extension manifest & settings schema |
| `tsconfig.json`            | TypeScript config                    |
| `jest.config.js`           | Test config                          |
| `examples/`                | Example config files for all formats |

## Supported File Formats

| Format     | Extensions                          | Parser Class        |
| ---------- | ----------------------------------- | ------------------- |
| ENV        | `.env`, `.env.*`, `*.env`, `.envrc` | `EnvParser`         |
| Shell      | `.sh`                               | `EnvParser`         |
| JSON       | `.json`                             | `JsonParser`        |
| YAML       | `.yaml`, `.yml`                     | `YamlParser`        |
| Properties | `.properties`, `.ini`, `.conf`      | `PropertiesParser`  |
| TOML       | `.toml`                             | `TomlParser`        |

## Dependency Flow

```
extension.ts
    ↓
core/camouflage.ts
    ↓
parsers/ + lib/ + utils/
```

**Rules**:

- `parsers/` → File format parsing (Strategy Pattern)
- `lib/` → Pure functions only
- `utils/` → Can use VS Code API
- `core/` → Orchestrates everything
- No circular dependencies

## Common Tasks

### Add New Parser

1. Create parser file in `src/parsers/`:

```typescript
import { BaseParser } from './base-parser';
import type { ParsedVariable } from './types';

export class NewParser extends BaseParser {
  readonly name = 'new';
  readonly supportedExtensions = ['.new'];

  parse(content: string): ParsedVariable[] {
    const variables: ParsedVariable[] = [];
    // Parse content and populate variables
    return variables;
  }
}
```

2. Register in `src/parsers/index.ts`:

```typescript
import { NewParser } from './new-parser';

// In ParserRegistry constructor:
this.registerParser(new NewParser());
```

3. Update `package.json` (enabledParsers enum):

```json
"camouflage.files.enabledParsers": {
  "items": {
    "enum": ["env", "json", "yaml", "properties", "toml", "new"]
  }
}
```

4. Add tests in `src/parsers/__tests__/new-parser.test.ts`
5. Update README with examples

### Add New Hiding Style

1. Update `package.json`:

```json
"enum": ["text", "dotted", "stars", "scramble", "newStyle"]
```

2. Update `text-generator.ts`:

```typescript
const strategies: Record<HiddenTextStyle, GeneratorStrategy> = {
  // ... existing
  newStyle: (text) => transformText(text),
};
```

3. Add tests:

```typescript
it('should generate newStyle text', () => {
  expect(generateHiddenText('test', 'newStyle')).toBe('expected');
});
```

4. Update README with example

### Add New Command

1. Update `package.json`:

```json
"commands": [{
  "command": "camouflage.newCommand",
  "title": "Camouflage: New Command"
}]
```

2. Register in `extension.ts`:

```typescript
const command = vscode.commands.registerCommand('camouflage.newCommand', () =>
  camouflage.newCommand()
);
context.subscriptions.push(command);
```

3. Implement in `camouflage.ts`:

```typescript
@Log('New Command')
@HandleErrors
public newCommand(): void {
  // Implementation
}
```

### Add New Configuration

1. Update `package.json`:

```json
"camouflage.newSetting": {
  "type": "boolean",
  "default": true,
  "description": "New setting description"
}
```

2. Add getter in `config.ts`:

```typescript
export function getNewSetting(): boolean {
  return vscode.workspace.getConfiguration('camouflage').get<boolean>('newSetting', true);
}
```

3. Use in code:

```typescript
import * as config from '../utils/config';

if (config.getNewSetting()) {
  // Do something
}
```

## Testing Patterns

### Unit Test Template

```typescript
describe('functionName', () => {
  beforeEach(() => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
