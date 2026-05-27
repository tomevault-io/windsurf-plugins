---
trigger: always_on
description: Testing guidelines, test structure, coverage requirements, mocking patterns, and VS Code extension testing
---


# Testing Guidelines

## Test Philosophy

### Core Principles

1. **Tests are documentation**: Test names should explain behavior
2. **Fast feedback**: Unit tests should run in milliseconds
3. **Reliable**: Tests should not be flaky
4. **Maintainable**: Tests should be easy to update when requirements change
5. **Coverage**: Minimum 80% code coverage, aim for 90%+

## Test Structure

### AAA Pattern (Arrange, Act, Assert)

```typescript
describe('generateHiddenText', () => {
  it('should mask text with stars style', () => {
    // Arrange
    const input = 'secret123';
    const style: HiddenTextStyle = 'stars';

    // Act
    const result = generateHiddenText(input, style);

    // Assert
    expect(result).toBe('*********');
  });
});
```

### Test File Organization

```typescript
// Top-level describe: Module or class name
describe('Camouflage', () => {
  // Setup and teardown
  beforeEach(() => {
    // Reset state before each test
  });

  afterEach(() => {
    // Cleanup after each test
  });

  // Nested describe: Method or feature
  describe('updateDecorations', () => {
    // Individual test cases
    it('should apply decorations to .env file', () => {});
    it('should skip non-.env files', () => {});
    it('should handle empty files', () => {});
  });

  describe('toggle', () => {
    it('should disable when enabled', () => {});
    it('should enable when disabled', () => {});
  });
});
```

## Test Coverage Requirements

### Required Coverage

- **Statements**: 80%+
- **Branches**: 75%+
- **Functions**: 80%+
- **Lines**: 80%+

### What to Test

✅ **Must Test**:

- All public methods and functions
- Edge cases (empty input, null, undefined)
- Error conditions
- Configuration changes
- State transitions

✅ **Should Test**:

- Private methods with complex logic
- Integration between modules
- Performance-critical paths

❌ **Don't Test**:

- Third-party library code
- Generated code
- Simple getters/setters without logic
- VS Code API itself

## Unit Testing Patterns

### Pure Functions (Easiest)

```typescript
// Function to test
export function matchPattern(key: string, pattern: string): boolean {
  return new RegExp(pattern, 'i').test(key);
}

// Test
describe('matchPattern', () => {
  it('should match case-insensitively', () => {
    expect(matchPattern('API_KEY', 'api')).toBe(true);
    expect(matchPattern('api_key', 'API')).toBe(true);
  });

  it('should support wildcard patterns', () => {
    expect(matchPattern('MY_API_KEY', '*API*')).toBe(true);
    expect(matchPattern('SECRET_TOKEN', '*TOKEN')).toBe(true);
  });

  it('should return false for non-matches', () => {
    expect(matchPattern('DATABASE_URL', 'api')).toBe(false);
  });
});
```

### Functions with Dependencies (Use Mocks)

```typescript
// Function to test
export function readEnvFile(filePath: string): string {
  return fs.readFileSync(filePath, 'utf8');
}

// Test with mock
jest.mock('fs');

describe('readEnvFile', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should read file content', () => {
    const mockContent = 'API_KEY=secret';
    (fs.readFileSync as jest.Mock).mockReturnValue(mockContent);

    const result = readEnvFile('/path/.env');

    expect(result).toBe(mockContent);
    expect(fs.readFileSync).toHaveBeenCalledWith('/path/.env', 'utf8');
  });

  it('should throw on file not found', () => {
    (fs.readFileSync as jest.Mock).mockImplementation(() => {
      throw new Error('ENOENT');
    });

    expect(() => readEnvFile('/path/.env')).toThrow('ENOENT');
  });
});
```

### Class Testing

```typescript
describe('Camouflage', () => {
  let camouflage: Camouflage;
  let mockEditor: vscode.TextEditor;

  beforeEach(() => {
    camouflage = new Camouflage();
    mockEditor = createMockEditor(); // Helper function
  });

  afterEach(() => {
    camouflage.dispose();
  });

  describe('initialization', () => {
    it('should create status bar item', () => {
      expect(camouflage['statusBarItem']).toBeDefined();
    });

    it('should apply decorations if .env file is open', () => {
      mockEditor.document.fileName = '/path/.env';
      vscode.window.activeTextEditor = mockEditor;

      camouflage = new Camouflage();

      expect(mockEditor.setDecorations).toHaveBeenCalled();
    });
  });

  describe('toggle', () => {
    it('should disable extension when enabled', async () => {
      await camouflage.toggle();

      expect(config.isEnabled()).toBe(false);
    });
  });
});
```

## Mocking VS Code API

### Mock Structure

```typescript
// __mocks__/vscode.ts
export const window = {
  activeTextEditor: undefined,
  createStatusBarItem: jest.fn(() => ({
    text: '',
    tooltip: '',
    show: jest.fn(),
    hide: jest.fn(),
    dispose: jest.fn(),
  })),
  showInformationMessage: jest.fn(),
  showErrorMessage: jest.fn(),
};

export const workspace = {
  getConfiguration: jest.fn(() => ({
    get: jest.fn(),
    update: jest.fn(),
  })),
  onDidChangeConfiguration: jest.fn(),
  onDidChangeTextDocument: jest.fn(),
};

export const commands = {
  registerCommand: jest.fn(),
  executeCommand: jest.fn(),
};

export class Range {
  constructor(
    public start: { line: number; character: number },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
