---
trigger: always_on
description: This is a Node.js library for controlling DYNAMIXEL servo motors via U2D2 interface with Protocol 2.0 support. The library provides both ESM and CommonJS builds, TypeScript definitions, and comprehensive Electron integration support.
---

# Cursor Rules for node-dynamixel

## Project Overview
This is a Node.js library for controlling DYNAMIXEL servo motors via U2D2 interface with Protocol 2.0 support. The library provides both ESM and CommonJS builds, TypeScript definitions, and comprehensive Electron integration support.

## Architecture & Code Organization

### Directory Structure
- `src/` - Source code (ES modules)
  - `dynamixel/` - Core DYNAMIXEL classes (Protocol2, DynamixelDevice, etc.)
  - `transport/` - Connection classes (SerialConnection, U2D2Connection, WebSerialConnection)
  - `utils/` - Utility classes (Logger)
  - `DynamixelController.js` - Main controller class
- `examples/` - Example scripts and usage demonstrations
- `tests/` - Test suite (unit and integration tests)
- `dist/` - Built files (generated, not in git)

### Key Classes & Patterns
- **DynamixelController**: Main entry point, handles device discovery and management
- **DynamixelDevice**: Individual motor control and monitoring
- **Protocol2**: DYNAMIXEL Protocol 2.0 implementation with CRC calculation
- **Transport Classes**: Connection abstractions (Serial, USB, WebSerial)
- **Event-driven**: Use EventEmitter patterns for connection state and errors

## Coding Standards

### JavaScript/ES Modules
- Use ES6+ features and ES modules (`import`/`export`)
- Prefer `const` over `let`, avoid `var`
- Use arrow functions for callbacks and short functions
- Use template literals for string interpolation
- Always use semicolons
- Use JSDoc comments for all public methods and classes

### Error Handling
- Always use try/catch for async operations
- Emit 'error' events on EventEmitter classes
- Provide descriptive error messages with context
- Use Protocol2.getErrorDescription() for DYNAMIXEL error codes

### Async Patterns
- Use async/await over Promise chains
- Always handle Promise rejections
- Use timeouts for hardware communication
- Clean up resources in finally blocks

### Buffer & Protocol Handling
- Always validate buffer lengths before parsing
- Use Protocol2.parseStatusPacket() for parsing raw responses
- Calculate CRC using Protocol2.calculateCRC()
- Handle both little-endian and big-endian data properly

## Development Workflow & Quality Assurance

### **MANDATORY Before Any Commit or Pull Request**
**Always run these commands in sequence - NO EXCEPTIONS:**

1. **Linting**: `npm run lint` (or `npm run lint:fix` to auto-fix)
2. **Building**: `npm run build` (verify TypeScript generation)
3. **Testing**: `npm test` (full test suite with --detectOpenHandles)
4. **Package Verification**: `npm pack --dry-run` (ensure package is valid)

### **Development Commands Reference**
```bash
# Primary workflow (run ALL of these before committing):
npm run lint          # Check code style and catch errors
npm run lint:fix       # Auto-fix linting issues where possible
npm run build          # Build ESM/CommonJS + TypeScript definitions
npm test               # Run all 284 tests with handle detection
npm pack --dry-run     # Verify package contents

# Individual test commands:
npm run test:unit      # Unit tests only
npm run test:integration  # Integration tests only
npm run test:coverage  # Tests with coverage report
npm run test:watch     # Watch mode for development

# Development helpers:
npm run build:watch    # Watch mode for building
npm run discovery      # Test device discovery
npm run diagnostics    # USB diagnostics
```

### **Code Quality Standards**
- **Zero linting errors** - Fix ALL ESLint warnings and errors
- **All tests must pass** - No exceptions, including integration tests
- **Clean builds** - TypeScript definitions must generate without errors
- **No hanging handles** - Jest must exit cleanly (--detectOpenHandles)
- **Package integrity** - npm pack must succeed without warnings

### **Pre-Commit Checklist**
- [ ] `npm run lint` passes with zero errors
- [ ] `npm run build` completes successfully
- [ ] `npm test` shows "284 passed" with no hanging handles
- [ ] `npm pack --dry-run` shows expected package contents
- [ ] `node examples/separated-discovery.js` works with real hardware (if available)
- [ ] Git status is clean (no untracked build artifacts)

## Testing Guidelines

### Test Structure
- Unit tests in `tests/unit/` for individual classes
- Integration tests in `tests/integration/` for real hardware scenarios
- Use Jest with ES modules support
- Mock hardware connections for unit tests

### **Testing + Linting Workflow**
**CRITICAL**: Always run linting before testing to catch code quality issues early:
```bash
# Correct workflow:
npm run lint           # Fix any linting errors first
npm run build          # Ensure build works
npm test               # Then run full test suite

# NEVER skip linting - it catches critical issues that tests might miss
```

### Test Patterns
- Use `createStatusPacketBuffer()` helper for mock DYNAMIXEL responses
- Always clean up connections in `afterEach` blocks
- Use `--detectOpenHandles` to ensure no resource leaks
- Test both success and error conditions

### Mock Guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andyshinn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
