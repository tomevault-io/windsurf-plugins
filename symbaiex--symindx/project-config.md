---
trigger: always_on
description: ENFORCE comprehensive testing standards when writing or modifying test files
---

globs: **/*.test.ts, **/*.spec.ts, jest.config.js
alwaysApply: false
---
# Testing and Quality Standards

**Rule Priority:** Core Architecture  
**Activation:** Always Active  
**Scope:** Testing strategies, quality assurance, and code standards

## Testing Architecture Overview

SYMindX follows a **comprehensive testing strategy** that ensures reliability, performance, and maintainability across all system components through multiple testing layers.

### Testing Pyramid Structure
```
┌─────────────────────────────────────────────────────────────┐
│                     SYMindX Testing Pyramid                  │
├─────────────────────────────────────────────────────────────┤
│              E2E Tests (10%)                │              │
│         ┌─────────────────────────────────────┐              │
│         │  Integration Tests (20%)            │              │
│    ┌─────────────────────────────────────────────────┐       │
│    │           Unit Tests (70%)                      │       │
│    └─────────────────────────────────────────────────┘       │
│                                                               │
│  Coverage Requirements:                                       │
│  • Unit Tests: 90%+ coverage                                │
│  • Integration Tests: Critical paths                         │
│  • E2E Tests: User journeys                                 │
└─────────────────────────────────────────────────────────────┘
```

## Unit Testing Standards

### Jest Configuration
```typescript
// jest.config.ts
import type { Config } from '@jest/types';

const config: Config.InitialOptions = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: [
    '**/__tests__/**/*.ts',
    '**/?(*.)+(spec|test).ts'
  ],
  transform: {
    '^.+\\.ts$': ['ts-jest', {
      useESM: true,
      tsconfig: {
        module: 'esnext'
      }
    }]
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.types.ts',
    '!src/**/index.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  },
  setupFilesAfterEnv: [
    '<rootDir>/tests/setup.ts'
  ],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};

export default config;
```

### Test File Organization
```typescript
// Standard test file structure
describe('AIPortalManager', () => {
  let portalManager: AIPortalManager;
  let mockEventBus: jest.Mocked<EventBus>;
  let mockConfig: AIPortalConfig;
  
  beforeEach(() => {
    // Setup mocks and test instances
    mockEventBus = createMockEventBus();
    mockConfig = createTestConfig();
    portalManager = new AIPortalManager(mockEventBus, mockConfig);
  });
  
  afterEach(() => {
    // Cleanup resources
    jest.clearAllMocks();
  });
  
  describe('initialization', () => {
    it('should initialize all enabled portals', async () => {
      // Test setup and execution
    });
    
    it('should handle initialization failures gracefully', async () => {
      // Error condition testing
    });
  });
  
  describe('provider selection', () => {
    it('should select primary provider for standard requests', async () => {
      // Normal flow testing
    });
    
    it('should fallback to secondary provider when primary fails', async () => {
      // Fallback behavior testing
    });
  });
  
  describe('error handling', () => {
    it('should retry failed requests with exponential backoff', async () => {
      // Retry logic testing
    });
    
    it('should circuit break after repeated failures', async () => {
      // Circuit breaker testing
    });
  });
});
```

### Mock Factory Patterns
```typescript
// Mock factories for consistent test data
export class TestFactories {
  static createMockAgent(overrides: Partial<Agent> = {}): Agent {
    return {
      id: 'test-agent-id',
      name: 'Test Agent',
      characterId: 'nyx',
      status: 'inactive',
      config: {
        memoryProvider: 'sqlite',
        emotionModule: 'confident',
        cognitiveModule: 'reactive',
        aiPortal: 'openai'
      },
      metadata: {
        createdAt: new Date('2024-01-01'),
        lastActive: null,
        version: '1.0.0'
      },
      ...overrides
    };
  }
  
  static createMockMessage(overrides: Partial<Message> = {}): Message {
    return {
      id: `msg-${Date.now()}`,
      content: 'Test message content',
      sender: {
        id: 'test-user',
        name: 'Test User',
        platform: 'test'
      },
      channel: {
        id: 'test-channel',
        type: 'direct',
        platform: 'test'
      },
      timestamp: new Date(),
      ...overrides
    };
  }
  
  static createMockEventBus(): jest.Mocked<EventBus> {
    return {
      emit: jest.fn(),
      emitAndWait: jest.fn(),
      on: jest.fn(),
      off: jest.fn(),
      once: jest.fn(),
      removeAllListeners: jest.fn()
    } as jest.Mocked<EventBus>;
  }
}
```

### Testing AI Portal Integrations
```typescript
describe('OpenAIPortal', () => {
  let portal: OpenAIPortal;
  let mockOpenAI: jest.Mocked<OpenAI>;
  
  beforeEach(() => {
    mockOpenAI = {
      chat: {
        completions: {
          create: jest.fn()
        }
      }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
