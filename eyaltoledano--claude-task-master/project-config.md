---
trigger: always_on
description: Before implementing the TDD workflow, ensure your project has a proper testing framework configured. This section covers setup for different technology stacks.
---

# Test Workflow & Development Process

## **Initial Testing Framework Setup**

Before implementing the TDD workflow, ensure your project has a proper testing framework configured. This section covers setup for different technology stacks.

### **Detecting Project Type & Framework Needs**

**AI Agent Assessment Checklist:**
1. **Language Detection**: Check for `package.json` (Node.js/JavaScript), `requirements.txt` (Python), `Cargo.toml` (Rust), etc.
2. **Existing Tests**: Look for test files (`.test.`, `.spec.`, `_test.`) or test directories
3. **Framework Detection**: Check for existing test runners in dependencies
4. **Project Structure**: Analyze directory structure for testing patterns

### **JavaScript/Node.js Projects (Jest Setup)**

#### **Prerequisites Check**
```bash
# Verify Node.js project
ls package.json  # Should exist

# Check for existing testing setup
ls jest.config.js jest.config.ts  # Check for Jest config
grep -E "(jest|vitest|mocha)" package.json  # Check for test runners
```

#### **Jest Installation & Configuration**

**Step 1: Install Dependencies**
```bash
# Core Jest dependencies
npm install --save-dev jest

# TypeScript support (if using TypeScript)
npm install --save-dev ts-jest @types/jest

# Additional useful packages
npm install --save-dev supertest @types/supertest  # For API testing
npm install --save-dev jest-watch-typeahead  # Enhanced watch mode
```

**Step 2: Create Jest Configuration**

Create `jest.config.js` with the following production-ready configuration:

```javascript
/** @type {import('jest').Config} */
module.exports = {
  // Use ts-jest preset for TypeScript support
  preset: 'ts-jest',

  // Test environment
  testEnvironment: 'node',

  // Roots for test discovery
  roots: ['<rootDir>/src', '<rootDir>/tests'],

  // Test file patterns
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],

  // Transform files
  transform: {
    '^.+\\.ts$': [
      'ts-jest',
      {
        tsconfig: {
          target: 'es2020',
          module: 'commonjs',
          esModuleInterop: true,
          allowSyntheticDefaultImports: true,
          skipLibCheck: true,
          strict: false,
          noImplicitAny: false,
        },
      },
    ],
    '^.+\\.js$': [
      'ts-jest',
      {
        useESM: false,
        tsconfig: {
          target: 'es2020',
          module: 'commonjs',
          esModuleInterop: true,
          allowSyntheticDefaultImports: true,
          allowJs: true,
        },
      },
    ],
  },

  // Module file extensions
  moduleFileExtensions: ['ts', 'tsx', 'js', 'jsx', 'json', 'node'],

  // Transform ignore patterns - adjust for ES modules
  transformIgnorePatterns: ['node_modules/(?!(your-es-module-deps|.*\\.mjs$))'],

  // Coverage configuration
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: [
    'text', // Console output
    'text-summary', // Brief summary
    'lcov', // For IDE integration
    'html', // Detailed HTML report
  ],

  // Files to collect coverage from
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/**/index.ts', // Often just exports
    '!src/generated/**', // Generated code
    '!src/config/database.ts', // Database config (tested via integration)
  ],

  // Coverage thresholds - TaskMaster standards
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Higher standards for critical business logic
    './src/utils/': {
      branches: 85,
      functions: 90,
      lines: 90,
      statements: 90,
    },
    './src/middleware/': {
      branches: 80,
      functions: 85,
      lines: 85,
      statements: 85,
    },
  },

  // Setup files
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],

  // Global teardown to prevent worker process leaks
  globalTeardown: '<rootDir>/tests/teardown.ts',

  // Module path mapping (if needed)
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },

  // Clear mocks between tests
  clearMocks: true,

  // Restore mocks after each test
  restoreMocks: true,

  // Global test timeout
  testTimeout: 10000,

  // Projects for different test types
  projects: [
    // Unit tests - for pure functions only
    {
      displayName: 'unit',
      testMatch: ['<rootDir>/src/**/*.test.ts'],
      testPathIgnorePatterns: ['.*\\.integration\\.test\\.ts$', '/tests/'],
      preset: 'ts-jest',
      testEnvironment: 'node',
      collectCoverageFrom: [
        'src/**/*.ts',
        '!src/**/*.d.ts',
        '!src/**/*.test.ts',
        '!src/**/*.integration.test.ts',
      ],
      coverageThreshold: {
        global: {
          branches: 70,
          functions: 80,
          lines: 80,
          statements: 80,
        },
      },
    },
    // Integration tests - real database/services
    {
      displayName: 'integration',
      testMatch: [
        '<rootDir>/src/**/*.integration.test.ts',
        '<rootDir>/tests/integration/**/*.test.ts',
      ],
      preset: 'ts-jest',
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/integration.ts'],
      testTimeout: 10000,
    },

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
