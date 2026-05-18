---
trigger: always_on
description: Guidelines for implementing and maintaining tests for Task Master CLI
---


# Testing Guidelines for Task Master CLI

## Test Organization Structure

- **Unit Tests**
  - Located in `tests/unit/`
  - Test individual functions and utilities in isolation
  - Mock all external dependencies
  - Keep tests small, focused, and fast
  - Example naming: `utils.test.js`, `task-manager.test.js`

- **Integration Tests**
  - Located in `tests/integration/`
  - Test interactions between modules
  - Focus on component interfaces rather than implementation details
  - Use more realistic but still controlled test environments
  - Example naming: `task-workflow.test.js`, `command-integration.test.js`

- **End-to-End Tests**
  - Located in `tests/e2e/`
  - Test complete workflows from a user perspective
  - Focus on CLI commands as they would be used by users
  - Example naming: `create-task.e2e.test.js`, `expand-task.e2e.test.js`

- **Test Fixtures**
  - Located in `tests/fixtures/`
  - Provide reusable test data
  - Keep fixtures small and representative
  - Export fixtures as named exports for reuse

## Test File Organization

```javascript
// 1. Imports
import { jest } from '@jest/globals';

// 2. Mock setup (MUST come before importing the modules under test)
jest.mock('fs');
jest.mock('@anthropic-ai/sdk');
jest.mock('../../scripts/modules/utils.js', () => ({
  CONFIG: {
    projectVersion: '1.5.0'
  },
  log: jest.fn()
}));

// 3. Import modules AFTER all mocks are defined
import { functionToTest } from '../../scripts/modules/module-name.js';
import { testFixture } from '../fixtures/fixture-name.js';
import fs from 'fs';

// 4. Set up spies on mocked modules (if needed)
const mockReadFileSync = jest.spyOn(fs, 'readFileSync');

// 5. Test suite with descriptive name
describe('Feature or Function Name', () => {
  // 6. Setup and teardown (if needed)
  beforeEach(() => {
    jest.clearAllMocks();
    // Additional setup code
  });
  
  afterEach(() => {
    // Cleanup code
  });
  
  // 7. Grouped tests for related functionality
  describe('specific functionality', () => {
    // 8. Individual test cases with clear descriptions
    test('should behave in expected way when given specific input', () => {
      // Arrange - set up test data
      const input = testFixture.sampleInput;
      mockReadFileSync.mockReturnValue('mocked content');
      
      // Act - call the function being tested
      const result = functionToTest(input);
      
      // Assert - verify the result
      expect(result).toBe(expectedOutput);
      expect(mockReadFileSync).toHaveBeenCalledWith(expect.stringContaining('path'));
    });
  });
});
```

## Jest Module Mocking Best Practices

- **Mock Hoisting Behavior**
  - Jest hoists `jest.mock()` calls to the top of the file, even above imports
  - Always declare mocks before importing the modules being tested
  - Use the factory pattern for complex mocks that need access to other variables

  ```javascript
  // ✅ DO: Place mocks before imports
  jest.mock('commander');
  import { program } from 'commander';
  
  // ❌ DON'T: Define variables and then try to use them in mocks
  const mockFn = jest.fn();
  jest.mock('module', () => ({
    func: mockFn // This won't work due to hoisting!
  }));
  ```

- **Mocking Modules with Function References**
  - Use `jest.spyOn()` after imports to create spies on mock functions
  - Reference these spies in test assertions
  
  ```javascript
  // Mock the module first
  jest.mock('fs');
  
  // Import the mocked module
  import fs from 'fs';
  
  // Create spies on the mock functions
  const mockExistsSync = jest.spyOn(fs, 'existsSync').mockReturnValue(true);
  
  test('should call existsSync', () => {
    // Call function that uses fs.existsSync
    const result = functionUnderTest();
    
    // Verify the mock was called correctly
    expect(mockExistsSync).toHaveBeenCalled();
  });
  ```

- **Testing Functions with Callbacks**
  - Get the callback from your mock's call arguments
  - Execute it directly with test inputs
  - Verify the results match expectations
  
  ```javascript
  jest.mock('commander');
  import { program } from 'commander';
  import { setupCLI } from '../../scripts/modules/commands.js';
  
  const mockVersion = jest.spyOn(program, 'version').mockReturnValue(program);
  
  test('version callback should return correct version', () => {
    // Call the function that registers the callback
    setupCLI();
    
    // Extract the callback function
    const versionCallback = mockVersion.mock.calls[0][0];
    expect(typeof versionCallback).toBe('function');
    
    // Execute the callback and verify results
    const result = versionCallback();
    expect(result).toBe('1.5.0');
  });
  ```

## ES Module Testing Strategies

When testing ES modules (`"type": "module"` in package.json), traditional mocking approaches require special handling to avoid reference and scoping issues.

- **Module Import Challenges**
  - Functions imported from ES modules may still reference internal module-scoped variables
  - Imported functions may not use your mocked dependencies even with proper jest.mock() setup
  - ES module exports are read-only properties (cannot be reassigned during tests)

- **Mocking Entire Modules**
  ```javascript
  // Mock the entire module with custom implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skindhu/AI-TASK-MANAGER](https://github.com/skindhu/AI-TASK-MANAGER) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
