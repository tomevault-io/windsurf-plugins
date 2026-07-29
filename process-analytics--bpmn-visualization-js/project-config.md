---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`bpmn-visualization` is a TypeScript library for visualizing process execution data on BPMN diagrams. It parses BPMN 2.0 XML, builds an internal model, and renders interactive diagrams using mxGraph with extensive customization capabilities.

## Common Development Commands

### Setup and Development
```bash
npm install                    # Install dependencies
npm run dev                    # Start dev server at http://localhost:10001/dev/public/index.html
npm run all                    # Full check: clean, lint, build, test (run before PRs)
```

### Building
```bash
npm run build                  # TypeScript compilation check (no output)
npm run build-bundles          # Create distribution bundles (ESM, IIFE)
npm run prepack                # Full build: generate types + bundles (runs before npm pack)
```

### Testing
```bash
npm test                       # Run all tests (unit + integration + e2e)
npm run test:unit              # Jest unit tests (parsers, converters, utilities)
npm run test:unit:coverage     # Unit tests with coverage
npm run test:unit:watch        # Watch mode for unit tests
npm run test:integration       # Integration tests (parser → renderer)
npm run test:e2e               # Playwright E2E with visual regression
npm run test:e2e:verbose       # E2E tests with debug output
npm run test:perf              # Performance benchmarks
npm run test:bundles           # Validate distribution bundles
```

To run a single test file:
```bash
# Unit test
npx jest test/unit/path/to/test-file.test.ts --config=./test/unit/jest.config.cjs

# Integration test
npx jest test/integration/path/to/test-file.test.ts --config=./test/integration/jest.config.cjs

# E2E test
npx jest test/e2e/path/to/test-file.test.ts --config=./test/e2e/jest.config.cjs
```

### Linting and Code Quality
```bash
npm run lint                   # Auto-fix linting issues
npm run lint-check             # Check linting without fixing
```

### JSDoc Guidelines

When writing or updating code, follow these JSDoc practices:

**Public API Documentation:**
- **REQUIRED**: All public classes, methods, interfaces, and types must have JSDoc comments
- **REQUIRED**: All new public API elements must include the `@since` tag with the version number
  - Infer the next version from `package.json` (current: `0.47.0-post` → next: `0.48.0`)
  - Always confirm the target version with the user once per session before using it
- Include description, `@param` for parameters, `@return` for return values
- Document exceptions with `@throws` when applicable
- Use `@example` for complex APIs to show usage patterns
- Mark experimental APIs with `@experimental` tag
- Mark deprecated APIs with `@deprecated` and provide migration guidance

**Internal Code Documentation:**
- **OPTIONAL**: Internal/private code may have JSDoc but it's not required
- Prefer self-documenting code with clear naming over excessive comments
- Add JSDoc for complex internal logic where the "why" isn't obvious
- Use inline comments for non-trivial implementation details

**JSDoc Best Practices:**
- Keep descriptions concise but complete
- Use TypeScript types; avoid redundant type information in JSDoc (rely on `@param {Type}` when TypeScript inference isn't sufficient)
- Link to related types/methods using `{@link ClassName}` or `{@link methodName}`
- Document parameter constraints and valid ranges
- Specify units for numeric parameters (e.g., "duration in milliseconds")

**Example:**
```typescript
/**
 * Loads and renders a BPMN diagram from XML.
 *
 * @param xml The BPMN 2.0 XML content to parse and render
 * @param options Optional configuration for rendering behavior
 * @returns The loaded model information
 * @throws {Error} If the XML is invalid or cannot be parsed
 * @since 0.48.0
 * @example
 * const bpmnVisualization = new BpmnVisualization({ container: 'diagram' });
 * bpmnVisualization.load('<definitions>...</definitions>');
 */
public load(xml: string, options?: LoadOptions): LoadResult {
  // implementation
}
```

### Documentation
```bash
npm run docs                   # Generate all documentation
npm run docs:user              # Generate user documentation
npm run docs:api               # Generate API documentation with typedoc
```

## Architecture Overview

The library follows a **3-layer pipeline architecture** with clear separation of concerns:

```
BpmnVisualization.load(xml)
  ├─ 1. Parse:    XML → BpmnModel (internal representation)
  ├─ 2. Register: BpmnModel → RenderedModel (prepare for rendering)
  └─ 3. Render:   RenderedModel → mxGraph cells (visual display)
```

### 1. Parsing Layer (`src/component/parser/`)

**Two-stage parsing:**
- **Stage 1:** `BpmnXmlParser` converts BPMN XML to JSON using `fast-xml-parser`
- **Stage 2:** `BpmnJsonParser` orchestrates converters to build `BpmnModel`

**Key Converters** (in `src/component/parser/json/converter/`):
- `ProcessConverter` - Activities, events, gateways, sequence flows
- `CollaborationConverter` - Pools, lanes, message flows
- `DiagramConverter` - Visual information (shapes, edges, bounds, labels)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [process-analytics/bpmn-visualization-js](https://github.com/process-analytics/bpmn-visualization-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
