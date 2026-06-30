---
trigger: always_on
description: This file provides guidelines and commands for agentic coding agents operating in this repository.
---

# AGENTS.md - Autoflow Development Guide

This file provides guidelines and commands for agentic coding agents operating in this repository.

## Project Overview

Autoflow is a **monorepo** containing:
- **Backend**: Java 17, Spring Boot, Maven multi-module project
- **Frontend**: Vue 3, TypeScript, Vite, Vitest

## Project Structure

```
autoflow/
├── autoflow-app/          # Main Spring Boot application
├── autoflow-agent/        # Agent runtime (ReAct, streaming, memory)
├── autoflow-common/      # Common utilities
├── autoflow-core/         # Core workflow engine
├── autoflow-modules/      # Workflow modules (Flowable, LiteFlow)
├── autoflow-plugins/      # Plugin system (HTTP, SQL, LLM, etc.)
├── autoflow-spi/          # Service Provider Interface definitions
├── autoflow-fe/           # Vue 3 frontend application
└── pom.xml               # Root Maven configuration
```

---

## Build & Test Commands

### Backend (Maven)

```bash
# Build entire project
mvn clean install

# Build without tests
mvn clean install -DskipTests

# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=ReActIntegrationTest

# Run specific test method
mvn test -Dtest=ToolRegistryImplTest#shouldRegisterTool

# Run tests in specific module
mvn -pl autoflow-agent test

# Run checkstyle validation only
mvn checkstyle:check

# Skip checkstyle during build
mvn clean install -Dcheckstyle.skip=true

# Package specific module
mvn -pl autoflow-app clean package

# Build with Maven profile (if defined)
mvn clean install -P<profile-name>
```

### Frontend (autoflow-fe/)

```bash
cd autoflow-fe

# Install dependencies
npm install

# Start development server (http://localhost:5173)
npm run dev

# Production build
npm run build

# Type-check TypeScript
npm run type-check

# Lint with ESLint (auto-fix)
npm run lint

# Format with Prettier
npm run format

# Run tests (watch mode)
npm run test

# Run tests once
npm run test:run

# Run tests with coverage
npm run test:coverage

# Run single test file
npx vitest run src/stores/chat.test.ts

# Run tests matching pattern
npx vitest run --grep "modelId"
```

---

## Code Style Guidelines

### Java (Backend)

**Configuration:**
- Checkstyle rules in `checkstyle.xml` (root directory)
- Max line length: **200 characters**
- Max method length: **80 lines**
- Max method parameters: **8**
- Max nested for depth: **3**
- Max nested if depth: **3**

**Naming Conventions:**
| Element | Convention | Example |
|---------|-----------|---------|
| Package | lowercase | `io.autoflow.spi.model` |
| Class/Interface | PascalCase | `AgentEngine`, `ServiceData` |
| Method | camelCase | `executeTool`, `parseArgs` |
| Variable | camelCase | `sessionId`, `toolName` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Enum values | UPPER_SNAKE_CASE | `GET`, `POST` |

**Import Rules:**
- No redundant imports
- No unused imports
- No `import java.lang.*` (except in specific cases)
- Group imports: `java.*`, `javax.*`, third-party, internal

**Class Structure (recommended order):**
1. Fields (public static, public instance, private)
2. Constructors
3. Public methods
4. Private methods
5. Inner classes

**Error Handling:**
- Use specific exception types (`ExecuteException`, `InputValidateException`)
- Never swallow exceptions silently
- Always log at appropriate level (ERROR for failures, INFO for significant events, DEBUG for details)
- Use `log.error("message", exception)` for caught exceptions

**Annotations:**
- Use Lombok judiciously (`@Data`, `@Slf4j`, `@Service`)
- `@Override` required when overriding methods
- Javadoc required on public classes (Chinese comments acceptable)

**Code Patterns:**
- Use records for simple DTOs: `public record LlmResult(String text, List<ToolExecutionRequest> requests) {}`
- Use `Map.of()` and `List.of()` for immutable collections
- Prefer `switch` expressions over switch statements
- Use `Optional` for nullable return values

### TypeScript (Frontend)

**Prettier Configuration:**
```json
{
  "semi": false,
  "tabWidth": 2,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none"
}
```

**ESLint Configuration:**
- Extends: `plugin:vue/vue3-essential`, `eslint:recommended`, `@vue/eslint-config-typescript`, `@vue/eslint-config-prettier/skip-formatting`
- Vue 3 Composition API style

**Naming Conventions:**
| Element | Convention | Example |
|---------|-----------|---------|
| Variable | camelCase | `chatSession`, `activeSession` |
| Function | camelCase | `createChatSession`, `chatSSE` |
| Type/Interface | PascalCase | `ChatSession`, `ChatSSECallbacks` |
| Enum | PascalCase | `ContentTypeEnum` |
| Enum values | UPPER_SNAKE_CASE | `GET`, `POST` |
| File | kebab-case | `chat-session.ts`, `use-chat.ts` |

**Import Organization:**
1. Vue/core frameworks (`vue`, `vue-router`, `pinia`)
2. External libraries (`axios`, `@microsoft/fetch-event-source`)
3. Internal aliases (`@/api`, `@/stores`, `@/hooks`)
4. Relative imports (`./`, `../`)

**TypeScript Patterns:**
- Use explicit types for function parameters and return values
- Use `any` sparingly - prefer `unknown` for truly unknown types
- Use `interface` for object shapes, `type` for unions/primitives
- Use `Record<string, T>` instead of `{ [key: string]: T }`

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yiuman/autoflow](https://github.com/Yiuman/autoflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
