---
trigger: always_on
description: This file provides context and instructions for AI coding agents working on the USAi API Node.js module - an open-source library for integrating with government AI services.
---

# AGENTS.md

This file provides context and instructions for AI coding agents working on the USAi API Node.js module - an open-source library for integrating with government AI services.

## Project Overview

The USAi API is a TypeScript/Node.js client library that provides an OpenAI-compatible interface for accessing government AI models (Claude, Llama, Gemini) through the USAi.gov platform. This is a **beta open-source project** designed for government agencies and contractors.

### Key Technologies
- TypeScript with strict mode
- Node.js (20.x, 22.x support)
- Jest for testing
- ESLint + Prettier for code quality
- OpenAI-compatible API design patterns

## Setup Commands

```bash
# Install dependencies
npm install

# Development build with watch mode
npm run dev

# Production build
npm run build

# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Lint code
npm run lint

# Format code
npm run format

# Type checking
npm run type-check
```

## Development Environment

### File Structure
- `src/` - TypeScript source code
  - `index.ts` - Main exports
  - `client.ts` - Core USAiAPI class
  - `http-client.ts` - HTTP client with retry logic
  - `types.ts` - Type definitions
  - `errors.ts` - Custom error classes
- `tests/` - Jest test files
- `examples/` - Usage examples including Jupyter notebook
- `dist/` - Built output (ESM + CommonJS)

### Build System
- TypeScript compilation to both ESM (`dist/esm/`) and CommonJS (`dist/cjs/`)
- Dual package.json approach for proper module resolution
- Source maps included for debugging

### Testing Strategy
- Unit tests for all core functionality
- Integration tests for API interactions
- Error handling tests for edge cases
- Enhanced features tests for government-specific functionality
- Target: **>90% code coverage**

## Code Style Guidelines

### TypeScript Standards
- Strict mode enabled - no `any` types
- Explicit return types for public methods
- Interface-based design over class inheritance
- Comprehensive JSDoc for all public APIs

### Formatting Rules
- Prettier configuration - 2 spaces, single quotes, no semicolons
- ESLint rules - TypeScript recommended + custom government rules
- Import organization - external imports first, then internal
- File naming - kebab-case for files, PascalCase for classes

### Code Patterns
```typescript
// Preferred: Interface-based design
interface USAiConfig {
  apiKey: string
  baseURL?: string
  timeout?: number
}

// Preferred: Explicit error handling
async function makeRequest(): Promise<Result<T, USAiError>> {
  try {
    // Implementation
  } catch (error) {
    return new USAiError('Specific error message', error)
  }
}

// Preferred: Government-focused documentation
/**
 * Processes government documents for AI analysis
 * @param document - Document content (PDF, DOCX, TXT)
 * @param options - Processing options for compliance
 * @returns Promise<DocumentAnalysis> - Analysis results
 * @government-use Approved for federal agency use
 */
```

## Testing Instructions

### Running Tests
```bash
# Run all tests
npm test

# Run specific test file
npm test -- client.test.ts

# Run tests in watch mode
npm run test:watch

# Generate coverage report
npm run test:coverage
```

### Test Requirements
- **All new features** must include tests
- **Edge cases** should be covered
- **Error scenarios** must be tested
- **Government compliance** features need specific tests
- **API compatibility** with OpenAI interface must be validated

### Test Categories
1. **Unit Tests** - Individual function/method testing
2. **Integration Tests** - API endpoint interactions
3. **Error Handling** - Network failures, invalid responses
4. **Government Features** - Document processing, enhanced embeddings
5. **Security Tests** - Input validation, sanitization

## Security Considerations

### Government Requirements
- **No hardcoded secrets** - use environment variables
- **Input validation** on all user data
- **Sanitization** of file uploads
- **Rate limiting** respect for government APIs
- **Audit logging** for compliance tracking

### Sensitive Data Handling
```typescript
// Correct: Environment-based configuration
const config = {
  apiKey: process.env.USAI_API_KEY,
  baseURL: process.env.USAI_BASE_URL || 'https://api.usai.gov'
}

// Wrong: Hardcoded credentials
const config = {
  apiKey: 'sk-123456789',  // NEVER do this
}
```

### Security Testing
- Run `npm audit` before commits
- Validate all file upload handling
- Test rate limiting behavior
- Ensure no sensitive data in logs

## Build and Deployment

### Build Process
```bash
# Clean previous builds
npm run clean

# Build for production
npm run build

# Verify build output
npm run build:verify

# Prepare for publishing
npm run prepublishOnly
```

### Package Publishing
- **Dual module support** (ESM + CommonJS)
- **TypeScript declarations** included
- **Source maps** for debugging
- **Government compliance** metadata in package.json

### CI/CD Integration
The project uses GitHub Actions for:
- **Multi-version testing** (Node 20, 22)
- **Security scanning** (npm audit, Snyk, CodeQL)
- **Government compliance** checks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adhocteam/usai-api](https://github.com/adhocteam/usai-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
