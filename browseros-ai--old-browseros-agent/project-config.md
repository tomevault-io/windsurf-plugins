---
trigger: always_on
description: - DO NOT automatically create or update, README.md file for changes.
---


## Rules
#  ALWAYS FOLLOW THESE RULES.
- DO NOT automatically create or update, README.md file for changes.
- DO NOT automatically generate example file to use the code unless asked.
- DO NOT automatically generate tests for the code unless asked.
- IMPORTANT: When asked a question or given a task, ALWAYS first generate a rough plan with pseudo code or design. DO NOT make changes without asking for approval first.
- IMPORTANT: Never use optional defaults like `|| "default-value"` in code. Always define constants at the top of the file for any default values (e.g., `const DEFAULT_MODEL = "gpt-4o-mini"`)

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

# Code Style & Formatting
- Use English for all code and documentation.
- Write concise, technical TypeScript. Follow Standard.js rules.
- Always declare the type of each variable and function (parameters and return value).
- Avoid using any.
- Create necessary types.
- Keep interfaces in the same file as their components rather than in a separate types directory.
- Use JSDoc to document public classes and methods.
- For interfaces, class properties, and smaller logic use inline comments, give two spaces and "// <comment>".
- DO NOT use JSDoc-style comments (`/** ... */`) for class properties or schema definitions, use inline comments instead.
- IMPORTANT: Follow this commenting style for clean, readable code:
  - Keep related code blocks compact and grouped together
  - Add brief comments only at the top of logical sections (e.g., "// Get the tool", "// Execute tool and handle result")
  - Avoid obvious comments - let the code be self-documenting
  - Comment spacing: Add comments to separate logical blocks within longer methods (5-10 lines per block)
  - Only comment where intent isn't immediately clear from the code
  - Examples of good comments:
    - Section separators: "// Check for task completion"
    - Non-obvious logic: "// Re-plan if execution failed"
    - TODO items or warnings about gotchas
  - Examples of bad comments to avoid:
    - "// Log tool call" (obvious from code)
    - "// Set result" (self-evident)
    - "// Call function" (redundant)
- Favor loops and small helper modules over duplicate code.
- Use descriptive names with auxiliary verbs (e.g. isLoading, hasError).
- File layout: exported component → subcomponents → hooks/helpers → static content.
- IMPORTANT: All imports must use path aliases like "@/lib" instead of relative paths like "./" or "../"
- IMPORTANT: Private methods must be prefixed with underscore (e.g., `_privateMethod()`)

# Naming Conventions
- Use PascalCase for classes.
- Use camelCase for variables, functions, and methods.
- Directories: Use kebab-case (e.g. components/auth-wizard).
- Files: 
  - Use PascalCase ONLY for files that export a class with the same name (e.g. BrowserContext.ts exports class BrowserContext)
  - Use lowercase for all other files: utilities, functions, interfaces, types, enums (e.g. profiler.ts, types.ts, tool.interface.ts)
  - Components (.tsx files) always use PascalCase (e.g. UserProfile.tsx)
- Use UPPERCASE for environment variables.
- Avoid magic numbers and define constants.
- File extensions:
  - Components → .tsx
  - Hooks/Utils → .ts
  - Style modules → .module.scss
- Prefer named exports for components
- Types/Interfaces in PascalCase (e.g. User, ButtonProps)
- OUR PRODUCT NAME IS Nxtscape (the "s" is small letter) -- so use that name correctly when naming things.

# Functions & Logic
- Keep functions short and single-purpose (<20 lines).
- Avoid deeply nested blocks by:
  - Using early returns.
  - Extracting logic into utility functions.
- Use higher-order functions (map, filter, reduce) to simplify logic.
- Use arrow functions for simple cases (<3 instructions), named functions otherwise.
- Use default parameter values instead of null/undefined checks.
- Use RO-RO (Receive Object, Return Object) for passing and returning multiple parameters.
- IMPORTANT: Order methods using "Operation-based grouping" (hybrid approach):
  ```typescript
  class Example {
    // 1. Constructor/Initialization
    constructor() {}
    init() {}
    
    // 2. Public getter methods
    getData() {}
    getStatus() {}
    
    // 3. Public creator/builder methods
    createItem() {}
    buildConfig() {}
    
    // 4. Public action/command methods
    save() {}
    delete() {}
    refresh() {}
    
    // 5. Public predicate methods
    isReady() {}
    hasData() {}
    
    // 6. Private helper methods (with _ prefix)
    private _validateData() {}
    private _formatOutput() {}
  }
  ```

# Data Handling
- Avoid excessive use of primitive types; encapsulate data in composite types.
- Avoid placing validation inside functions—use classes with internal validation instead.
- Prefer immutability for data:
  - Use readonly for immutable properties.
  - Use as const for literals that never change.

# TypeScript & Zod

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browseros-ai/old-browseros-agent](https://github.com/browseros-ai/old-browseros-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
