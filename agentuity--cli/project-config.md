---
trigger: always_on
description: Code Generation Development Rules for CLI and SDK
---


# Code Generation Development Rules

> **⚠️ IMPORTANT**: These rules apply to both CLI and SDK development. Keep both codebases in sync when making changes.

## Core Principles

### Never Modify Generated Content in Source Files
- ❌ NEVER edit files in `sdk-js/src/` that contain generated content
- ❌ NEVER hardcode generated content like `copyWriter` prompts in source files
- ✅ ALWAYS generate content into `node_modules/@agentuity/sdk/dist/` or `src/` directories
- ✅ Use dynamic loading patterns for generated content

### Code Generation Workflow
1. **Modify CLI generation logic** (e.g., `internal/bundler/prompts.go`)
2. **Update SDK to handle generated content dynamically** (e.g., `src/apis/prompt/index.ts`)
3. **Build and test the full pipeline**: CLI generation → SDK loading → Agent usage

### Multi-File Support
- **Directory Scanning**: CLI scans `src/prompts/` and `prompts/` directories for all YAML files
- **File Processing**: Processes all `.yaml` and `.yml` files found in the prompts directory
- **Combined Output**: Merges prompts from multiple files into a single generated output
- **Legacy Support**: Still supports single `prompts.yaml` file in various locations

### JSDoc Documentation
- **Function Comments**: Generated `system` and `prompt` functions include JSDoc comments with actual prompt content
- **IDE Support**: JSDoc comments provide better IDE hover tooltips and documentation
- **Content Preservation**: Original prompt templates are preserved exactly as written in YAML
- **Line Wrapping**: Long lines are automatically wrapped at 80 characters for readability

### Optional Field Handling
- ✅ Generated code should NEVER require optional chaining (`?.`)
- ✅ Always generate both `system` and `prompt` fields, even if empty
- ✅ Empty fields should return empty strings, not undefined
- ❌ Never generate partial objects that require optional chaining

## Architecture Overview

### File Structure
```
sdk-js/src/apis/prompt/
├── generic_types.ts          # Simple utility types for CLI to use
├── generated/
│   ├── index.d.ts           # Shell TypeScript definitions (replaced by CLI)
│   ├── index.js             # Shell JavaScript (replaced by CLI)
│   └── _index.js            # Actual generated JavaScript (created by CLI)
├── index.ts                 # Main API with dynamic loading
└── signature.ts             # Signature function factory
```

### Key Learnings from Implementation

#### 1. Simplified Architecture
- **Use shell files**: `index.d.ts` and `index.js` are placeholders that get completely replaced
- **Less complex generics**: Avoid overly complex TypeScript generics that cause compilation issues
- **Rely on code generation**: Let the CLI do the heavy lifting instead of complex type manipulation

#### 2. Slug-Based Naming
- **Use slugs directly**: Generate code using the original slug names (e.g., `'simple-helper'`)
- **Quote property names**: Use `'slug-name'` syntax in TypeScript interfaces
- **Bracket notation**: Use `prompts['slug-name']` in JavaScript for property access
- **CamelCase variables**: Use `strcase.ToLowerCamel(slug)` for JavaScript variable names

#### 3. Type Safety Without Complexity
```typescript
// ✅ Simple, working approach
export type PromptsCollection = Record<string, any>;
export type PromptSignature<T> = (params: any) => string;

// ❌ Avoid overly complex generics that cause compilation issues
export type ComplexGeneric<T extends Prompt<infer A, infer B, infer C, infer D, infer E>> = ...
```

## CLI-Specific Rules

### Generation Target Locations
```go
// ✅ Correct: Generate into installed SDK
sdkPath := filepath.Join(root, "node_modules", "@agentuity", "sdk", "dist", "generated")

// ❌ Wrong: Generate into source SDK
sdkPath := filepath.Join(root, "src", "generated")
```

### Slug Handling in Code Generation
```go
// ✅ Correct: Use slugs directly with proper quoting
const %s = {
    slug: "%s",
    // ... fields
};`, strcase.ToLowerCamel(prompt.Slug), prompt.Slug

// In TypeScript interfaces
exports = append(exports, fmt.Sprintf("  '%s': %s;", prompt.Slug, strcase.ToCamel(prompt.Slug)))

// In JavaScript property access
bodyParts = append(bodyParts, fmt.Sprintf("const result = prompts['%s'].system(params)", prompt.Slug))
```

### File Generation Pattern
```go
func FindSDKGeneratedDir(ctx BundleContext, projectDir string) (string, error) {
    possibleRoots := []string{
        findWorkspaceInstallDir(ctx.Logger, projectDir),
        projectDir,
    }

    for _, root := range possibleRoots {
        // Try dist directory first (production)
        sdkPath := filepath.Join(root, "node_modules", "@agentuity", "sdk", "dist", "generated")
        if _, err := os.Stat(filepath.Join(root, "node_modules", "@agentuity", "sdk")); err == nil {
            if err := os.MkdirAll(sdkPath, 0755); err == nil {
                return sdkPath, nil
            }
        }
        // Fallback to src directory (development)
        sdkPath = filepath.Join(root, "node_modules", "@agentuity", "sdk", "src", "generated")
        if _, err := os.Stat(filepath.Join(root, "node_modules", "@agentuity", "sdk", "src")); err == nil {
            if err := os.MkdirAll(sdkPath, 0755); err == nil {
                return sdkPath, nil
            }
        }
    }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentuity/cli](https://github.com/agentuity/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
