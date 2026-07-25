---
trigger: always_on
description: Detailed step-by-step workflow instructions are available as prompt files in `.github/prompts/`. When asked to perform one of these tasks, **always read and follow the corresponding prompt file**:
---

# GitHub Copilot Instructions for Application Insights JavaScript SDK

## Workflow Prompts

Detailed step-by-step workflow instructions are available as prompt files in `.github/prompts/`. When asked to perform one of these tasks, **always read and follow the corresponding prompt file**:

- **Release PR**: `.github/prompts/release-pr.prompt.md` — Full workflow for creating a release PR (version bump, README updates, RELEASES.md, gruntfile, lint, commit, and build validation)

## Project Overview
This is the **Microsoft Application Insights JavaScript SDK** - a browser-based telemetry library for monitoring web applications. The SDK tracks page views, user interactions, performance metrics, exceptions, and custom events.

## Architecture & Structure

### Monorepo Structure
- **AISKU/**: Main Application Insights SDK package
- **AISKULight/**: Lightweight version of the SDK
- **shared/**: Core shared libraries (AppInsightsCore, AppInsightsCommon, 1ds-core-js)
- **extensions/**: Plugin-based extensions (analytics, dependencies, etc.)
- **channels/**: Data transmission channels (online, offline, tee)
- **tools/**: Build and development tools
- **examples/**: Sample implementations

### Key Technologies
- **TypeScript/JavaScript**: Primary languages (ES5 target for browser compatibility)
- **Rush**: Monorepo management tool
- **Rollup**: Module bundler
- **Grunt**: Task runner
- **Dynamic Proto**: Dynamic prototype pattern for performance

## Code Style & Patterns

### Required Before Each Commit
- Do not commit any changes that are only end-of-file whitespace changes
- Ensure all TypeScript files are formatted and imports are reordered correctly by running `npm run lint-fix` before committing
  - This will apply ESLint fixes to all TypeScript files
  - It will also reorder imports to maintain consistent style

### TypeScript/JavaScript Conventions
- Use **ES5-compatible** syntax for browser support and target ES5 for modern browsers
- Prefer `function` declarations over arrow functions for better IE compatibility
- Use `var` instead of `let/const` in JavaScript files for ES5 compatibility (use `let/const` in TypeScript files)
- Always use semicolons
- Use 4-space indentation
- Maximum line length: 140 characters

### Naming Conventions
- **Classes**: PascalCase (e.g., `PageViewManager`, `TelemetryContext`)
- **Interfaces**: PascalCase with `I` prefix (e.g., `ITelemetryItem`, `IPageViewTelemetry`)
- **Methods/Functions**: camelCase (e.g., `trackPageView`, `sendTelemetry`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_DURATION_ALLOWED`)
- **Private variables**: underscore prefix (e.g., `_logger`, `_hasInitialized`)
- **Enums**: PascalCase with `e` prefix (e.g., `eLoggingSeverity`, `eInternalMessageId`)
  - Must be const enums with integer values (not strings)
  - Use `createEnumStyle` helper for exported enums
  - All usage should reference the const enum directly

### Dynamic Proto Pattern
This project uses a unique `dynamicProto` pattern for performance optimization. This pattern should be used for all classes:

```typescript
export class MyClass {
    constructor() {
        dynamicProto(MyClass, this, (_self, _base) => {
            // Private variables should be included inside the constructor closure
            // They are not publicly visible on the class
            let _logger = _self._logger;
            let _hasInitialized = false;
            
            // Public methods need @DynamicProtoStub comment for TypeScript definitions
            _self.myMethod = () => {
                // Method implementation
            };
        });
    }
    
    /**
     * @DynamicProtoStub -- DO NOT add any code as this will be removed during packaging
     */
    public myMethod(): void {
        // This stub will be replaced by the dynamicProto implementation
    }
}
```

Key requirements:
- Use this pattern for all classes
- Private variables must be inside the constructor closure
- Public functions need `@DynamicProtoStub` comment for TypeScript definition generation
- Never add implementation code to the stub methods

### Error Handling
- Use `_throwInternal` for logging diagnostic errors
- Always include telemetry context in error messages
- Use appropriate logging severity levels: `CRITICAL`, `WARNING`, `INFORMATION`

```typescript
_throwInternal(_logger,
    eLoggingSeverity.WARNING,
    _eInternalMessageId.InvalidDurationValue,
    "Invalid page load duration value",
    { total, network, request, response, dom });
```

### Performance Considerations
- Minimize object allocations in hot paths
- Use `doPerf()` wrapper for performance tracking
- Avoid synchronous operations that could block the browser
- Implement lazy initialization where possible
- Use object pooling for frequently created objects
- Minimize the size of generated JavaScript by avoiding certain TypeScript features:
  - Do not use the spread `...` operator
  - Do not use optional chaining `?.` operator
  - Do not use the nullish coalescing `??` operator - use `||` instead
  - These restrictions will be removed once ES5 support is discontinued

## Browser Compatibility

### Target Support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/ApplicationInsights-JS](https://github.com/microsoft/ApplicationInsights-JS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
