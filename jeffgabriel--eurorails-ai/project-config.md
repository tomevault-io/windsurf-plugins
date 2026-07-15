---
trigger: always_on
description: Applies general coding rules across all file types to maintain code quality, consistency, and prevent common errors.
---

- Always verify information before presenting it. Do not make assumptions or speculate without clear evidence.
- Make changes file by file and give me a chance to spot mistakes.
- Never use apologies.
- Avoid giving feedback about understanding in comments or documentation.
- Don't suggest whitespace changes.
- Don't summarize changes made.
- Don't invent changes other than what's explicitly requested.
- Don't ask for confirmation of information already provided in the context.
- Don't remove unrelated code or functionalities. Pay attention to preserving existing structures.
- Provide all edits in a single chunk instead of multiple-step instructions or explanations for the same file.
- Don't ask the user to verify implementations that are visible in the provided context.
- Don't suggest updates or changes to files when there are no actual modifications needed.
- Always provide links to the real files, not the context generated file.
- Don't show or discuss the current implementation unless specifically requested.
- Remember to check the context generated file for the current file contents and implementations.
- Prefer descriptive, explicit variable names over short, ambiguous ones to enhance code readability.
- Adhere to the existing coding style in the project for consistency.
- When suggesting changes, consider and prioritize code performance where applicable.
- Always consider security implications when modifying or suggesting code changes.
- Suggest or include appropriate unit tests for new or modified code.
- Implement robust error handling and logging where necessary.
- Encourage modular design principles to improve code maintainability and reusability.
- Ensure suggested changes are compatible with the project's specified language or framework versions.
- Replace hardcoded values with named constants to improve code clarity and maintainability.
- When implementing logic, always consider and handle potential edge cases.
- Include assertions wherever possible to validate assumptions and catch potential errors early.
- **NO FAILED TESTS ALLOWED**: All tests must pass. Fix any failing tests immediately before proceeding with other work. This is a hard and fast rule that cannot be violated.

Here are our unified coding principles for the JavaScript/TypeScript codebase:

1. **Module System**
- Use ES Modules (`import`/`export`) consistently across all code
- Never mix with CommonJS (`require`/`module.exports`)
```typescript
// ✅ Do:
import { Something } from './something';
export class MyClass {}

// ❌ Don't:
const something = require('./something');
module.exports = MyClass;
```

2. **TypeScript Configuration**
- Keep `strict: true` in tsconfig.json
- Use explicit type annotations for function parameters and returns
- Allow implicit types for variables when type can be inferred
```typescript
// ✅ Do:
function add(a: number, b: number): number {
  const result = a + b; // type inferred
  return result;
}

// ❌ Don't:
function add(a, b) {
  return a + b;
}
```

3. **Testing Setup**
- Use Jest with TypeScript via `ts-jest`
- Keep test files in `__tests__` directories
- Use `.test.ts` extension for test files
- Mock external dependencies explicitly
```typescript
// ✅ Do:
import { MyService } from '../services';
jest.mock('../services');

// ❌ Don't:
const MyService = jest.requireActual('../services');
```

4. **File Organization**
```
src/
  client/
    __tests__/        # Client-side tests
    components/
    scenes/
  server/
    __tests__/        # Server-side tests
    routes/
    services/
```

5. **Jest Configuration**
```typescript
{
  preset: 'ts-jest',
  testEnvironment: 'node',
  projects: [
    {
      displayName: 'server',
      testMatch: ['<rootDir>/src/server/__tests__/**/*.test.ts'],
      moduleFileExtensions: ['ts', 'js'],
      transform: {
        '^.+\\.ts$': 'ts-jest'
      }
    },
    {
      displayName: 'client',
      testMatch: ['<rootDir>/src/client/__tests__/**/*.test.ts'],
      testEnvironment: 'jsdom',
      setupFilesAfterEnv: ['<rootDir>/src/client/__tests__/setupTests.ts']
    }
  ]
}
```

6. **Test File Structure**
```typescript
import { Something } from '../path';

// Mock setup at top of file
jest.mock('../path/to/dependency');

describe('Component/Feature name', () => {
  let instance: Something;
  
  beforeEach(() => {
    instance = new Something();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should do something', () => {
    // Arrange
    const input = 'test';
    
    // Act
    const result = instance.doSomething(input);
    
    // Assert
    expect(result).toBe('expected');
  });
});
```

7. **Error Handling**
- Use typed errors
- Always catch and handle errors appropriately
- Provide meaningful error messages
```typescript
// ✅ Do:
try {
  await service.doSomething();
} catch (error) {
  if (error instanceof ServiceError) {
    // Handle specific error
  }
  throw error;
}

// ❌ Don't:
try {
  await service.doSomething();
} catch (e) {
  console.log(e);
}
```

8. **Async Code**
- Use async/await over raw promises
- Handle promise rejections
- Maintain proper error propagation
```typescript
// ✅ Do:
async function getData(): Promise<Data> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
