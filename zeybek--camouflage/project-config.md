---
trigger: always_on
description: Critical guidelines for AI assistants working with Camouflage codebase - security rules, code style, and best practices
---


# AI Assistant Guidelines

## Overview

These guidelines help AI assistants (like Claude, GPT-4, Copilot) work effectively with the Camouflage codebase.

## Priority Rules

### 1. Never Modify File Content

**CRITICAL**: Camouflage NEVER modifies `.env` file content.

```typescript
// ❌ NEVER suggest this
const edit = new vscode.WorkspaceEdit();
edit.replace(document.uri, range, maskedValue);
await vscode.workspace.applyEdit(edit);

// ✅ ALWAYS use decorations
editor.setDecorations(decorationType, ranges);
```

**If asked to modify files**: Politely explain that Camouflage is decoration-only by design.

### 2. Respect Existing Architecture

Before suggesting changes:

1. ✅ Read relevant rule files (especially `architecture.mdc`)
2. ✅ Understand module dependencies
3. ✅ Follow established patterns
4. ❌ Don't suggest architectural changes without justification

### 3. Security First

Always consider security implications:

- ✅ No network requests with sensitive data
- ✅ Validate all user input
- ✅ No logging of values
- ✅ Sanitize patterns before regex compilation

## Code Suggestions

### Before Suggesting Code

1. **Read context**: Check what files are open, what the user is working on
2. **Check existing implementation**: Look for similar functionality
3. **Follow conventions**: Use existing naming, structure, patterns
4. **Consider tests**: Think about how it will be tested

### Code Style

```typescript
// ✅ GOOD: Follow project conventions
export function matchPattern(key: string, pattern: string): boolean {
  const sanitized = sanitizePattern(pattern);
  return new RegExp(sanitized, 'i').test(key);
}

// ❌ BAD: Different style
export const matchPattern = (key, pattern) => {
  return new RegExp(pattern, 'i').test(key);
};
```

### Imports

Always use existing import structure:

```typescript
// ✅ GOOD: Project convention
import * as vscode from 'vscode';
import { generateHiddenText } from '../lib/text-generator';
import * as config from '../utils/config';

// ❌ BAD: Different convention
import vscode = require('vscode');
const { generateHiddenText } = require('../lib/text-generator');
```

## Responding to Requests

### Adding Features

When asked to add a feature:

1. **Clarify requirements**:

   - What problem does it solve?
   - How should it behave?
   - What edge cases exist?

2. **Propose approach**:

   - Where should code go?
   - What files need changes?
   - Any architectural concerns?

3. **Consider impacts**:

   - Breaking changes?
   - Performance impact?
   - Security implications?
   - Testing requirements?

4. **Provide implementation**:
   - Follow coding standards
   - Include tests
   - Update documentation

### Example: Adding New Hiding Style

**User**: "Add a rainbow hiding style"

**AI Response**:

```
I'll help you add a rainbow hiding style. Here's the approach:

1. **Update package.json**: Add 'rainbow' to hiddenTextStyle enum
2. **Update text-generator.ts**: Implement rainbow transformation
3. **Add tests**: Test the new style
4. **Update README**: Document new feature

Here's the implementation:

[Code for package.json]
[Code for text-generator.ts]
[Code for tests]
[Documentation update]

This follows the Strategy pattern already used for hiding styles.
Want me to proceed with implementation?
```

### Debugging

When helping debug:

1. **Understand the problem**:

   - What's the expected behavior?
   - What's actually happening?
   - When does it occur?

2. **Check common issues**:

   - Is extension activated?
   - Is file recognized as `.env`?
   - Are decorations being applied?
   - Any console errors?

3. **Suggest investigation**:

   - Add logging
   - Check configuration
   - Review event listeners

4. **Provide fix**:
   - Explain root cause
   - Show fix with explanation
   - Suggest test to prevent regression

### Refactoring

When suggesting refactoring:

1. **Explain why**: What's improved?
2. **Show before/after**: Clear comparison
3. **Preserve behavior**: No functional changes
4. **Update tests**: Ensure they still pass

```
I suggest extracting this pattern matching logic into a separate function:

Before:
[Current code]

After:
[Refactored code]

Benefits:
- More testable
- Reusable
- Clearer responsibility

This follows the Single Responsibility Principle outlined in patterns.mdc.
```

## Testing Guidance

### Always Consider Tests

When suggesting code changes:

```typescript
// Suggest implementation
export function newFeature(input: string): string {
  return transform(input);
}

// ALSO suggest test
describe('newFeature', () => {
  it('should transform input correctly', () => {
    expect(newFeature('test')).toBe('expected');
  });

  it('should handle empty string', () => {
    expect(newFeature('')).toBe('');
  });
});
```

### Test Patterns

Follow AAA pattern:

```typescript
it('should mask sensitive values', () => {
  // Arrange
  const input = 'API_KEY=secret';
  const style: HiddenTextStyle = 'stars';

  // Act
  const result = maskValue(input, style);

  // Assert
  expect(result).toBe('API_KEY=******');
});
```

## Documentation

### Update Documentation

When adding/changing features:

1. ✅ Update README if user-facing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
