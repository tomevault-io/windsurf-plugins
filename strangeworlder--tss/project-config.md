---
trigger: always_on
description: - "fish shell, be vary of extra endline characters from Windows"
---

general:
  - shell:
      - "WSL2"
      - "fish shell, be vary of extra endline characters from Windows"
      - "Check correct node version by running `nvm use` in the terminal"
  - editor:
      - "latest version of Cursor"
      - "latest version of VS Code"
  - frontend:
      - "latest stable version of Vue"
      - "Typescript with strict mode enabled"
      - "TypeScript configuration:"
          - "verbatimModuleSyntax is enabled in tsconfig.json"
          - "strict mode is enabled"
          - "isolatedModules is enabled"
          - "esModuleInterop is enabled"
      - "TypeScript best practices:"
          - "Always use type-only imports for interfaces and types: `import type { IMyInterface } from './file'`"
          - "Never mix type imports with value imports in the same import statement"
          - "Use explicit return types for functions and methods"
          - "Prefer interfaces over type aliases for object shapes"
          - "Annotate exported constants and functions with explicit types"
      - "File organization and imports:"
          - "When moving files between directories, ALWAYS update all import paths in other files that reference the moved file"
          - "Use a systematic approach to file moves:"
              - "1. Identify all files that import the file being moved"
              - "2. Update import paths in those files before or immediately after the move"
              - "3. Verify that all imports use the correct path after the move"
              - "4. Run the application to check for import errors"
          - "Use absolute imports with @/ prefix for project files"
          - "Use relative imports for files in the same directory or for closely related files"
          - "When in doubt, search the entire codebase for references to the file being moved"
      - "Testing requirements:"
          - "Testing framework: Vitest for unit tests, Cypress for E2E tests"
          - "Test file organization:"
              - "Unit tests must be co-located with components in __tests__ directory"
              - "Test files must be named *.test.ts or *.test.tsx"
              - "E2E tests must be in cypress/e2e directory"
          - "Component testing requirements:"
              - "Every component MUST have unit tests"
              - "Tests must cover all props, events, and slots"
              - "Tests must verify accessibility features"
              - "Tests must include error states and edge cases"
              - "Tests must verify responsive behavior"
              - "100% coverage required for critical components"
          - "Test documentation:"
              - "Each test file must have a description of what is being tested"
              - "Test cases must have clear, descriptive names"
              - "Complex test setups must be documented"
          - "Testing best practices:"
              - "Use vi.mock for mocking dependencies"
              - "Mock external dependencies and API calls"
              - "Test user interactions and events"
              - "Verify component state changes"
              - "Test error handling and edge cases"
              - "Include accessibility testing"
          - "Testing structure:"
              - "Follow atomic design testing pattern for all components"
              - "Create separate test utility files (*.test-utils.ts)"
              - "Create fixture files for mock data in __fixtures__ directory"
              - "Organize tests by category (rendering, props, events, etc.)"
              - "Avoid script execution issues by creating files directly"
          - "Pinia store testing:"
              - "Use vi.mock to mock Pinia stores"
              - "Mock store state, getters, and actions"
              - "Test store mutations and state changes"
              - "Test store action error handling"
              - "Test store getter computations"
              - "Test store plugin functionality"
              - "Test store persistence (if used)"
              - "Example store mock:"
                  ```typescript
                  vi.mock('@/stores/myStore', () => ({
                    useMyStore: vi.fn(() => ({
                      state: {},
                      getters: {},
                      actions: vi.fn(),
                    })),
                  }));
                  ```
      - "Semantic variables are mandatory and must be maintained in front/src/assets/vars.css"
      - "All styling must use semantic variables - no hard-coded values allowed"
      - "Units must use rem as the base unit - avoid px completely"
      - "Semantic variable usage guidelines:"
          - "Always use semantic variables whenever possible"
          - "Never use base design tokens directly in components"
          - "When creating new semantic variables:"
              - "Must be generic enough to be used across multiple components"
              - "Must represent a meaningful design concept or purpose"
              - "Never create component-specific semantic variables"
              - "If a new semantic concept is identified, create a new semantic variable in vars.css"
      - "Color system consists of two layers:"
          - "Base design tokens (100-900 scale):"
              - "--color-primary-{variant}"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/strangeworlder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
