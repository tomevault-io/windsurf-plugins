---
trigger: always_on
description: Enforces strict TypeScript configuration for Next.js 16. Type safety, proper compiler options, and IDE integration for catching errors at development time.
---


# Next.js 16 TypeScript - Strict Configuration

This rule enforces **strict TypeScript settings** that catch errors during development, not production.

**Philosophy**: If it compiles with strict TypeScript, it's probably correct.

---

## 🎯 Core Principle

TypeScript is your **first line of defense** against bugs. Configure it strictly to catch:
- Type errors
- Null/undefined issues
- Missing return types
- Unused variables
- Import errors

**We enforce**: Strict mode, proper paths, typed routes, typed environment variables.

---

## ✅ REQUIRED: Base tsconfig.json

### Minimal Strict Configuration

Every Next.js 16 + TypeScript project MUST have this baseline:

```json filename="tsconfig.json"
{
  "compilerOptions": {
    // ===== STRICT TYPE CHECKING =====
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,

    // ===== MODULE RESOLUTION =====
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",

    // ===== EMIT =====
    "noEmit": true,
    "incremental": true,

    // ===== NEXT.JS SPECIFIC =====
    "target": "ES2020",
    "plugins": [
      {
        "name": "next"
      }
    ],

    // ===== PATH ALIASES =====
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": [
    "next-env.d.ts",
    ".next/types/**/*.ts",
    "**/*.ts",
    "**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

---

## 🔒 REQUIRED: Strict Mode Flags

### What Each Flag Does

```json
{
  "compilerOptions": {
    // ===== STRICT MODE (enables all strict checks) =====
    "strict": true,
    // This enables:
    // - strictNullChecks: null and undefined must be explicit
    // - strictFunctionTypes: function params are contravariant
    // - strictBindCallApply: bind/call/apply are correctly typed
    // - strictPropertyInitialization: class properties must be initialized
    // - noImplicitThis: 'this' must have explicit type
    // - alwaysStrict: emit "use strict"
    // - noImplicitAny: no implicit 'any' types

    // ===== ADDITIONAL STRICT CHECKS =====
    "noUncheckedIndexedAccess": true,
    // Array/object access returns 'T | undefined'
    // Catches: array[999] might be undefined

    "noImplicitOverride": true,
    // Must use 'override' keyword when overriding methods
    // Catches: accidental method name typos in subclasses

    "noFallthroughCasesInSwitch": true,
    // Prevents missing 'break' in switch statements
    // Catches: switch fallthrough bugs
  }
}
```

**Why strict mode is mandatory**:
- Catches null/undefined errors at compile time
- Forces explicit type annotations
- Prevents common runtime errors
- Industry standard for production TypeScript

---

## 📁 REQUIRED: Next.js Include Paths

### What Must Be Included

```json
{
  "include": [
    "next-env.d.ts",        // Next.js type definitions
    ".next/types/**/*.ts",  // Generated route types
    "**/*.ts",              // All TypeScript files
    "**/*.tsx"              // All React TypeScript files
  ],
  "exclude": [
    "node_modules"          // Never check dependencies
  ]
}
```

**Why `.next/types/**/*.ts` is critical**:
- Contains generated route types for typed routes
- Enables `typedRoutes: true` in next.config.ts
- Auto-generated on `next dev`, `next build`, `next typegen`

---

## 🎯 REQUIRED: Path Aliases

### Standard Next.js Aliases

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"],                    // Everything from root
      "@/components/*": ["components/*"], // Optional: specific paths
      "@/lib/*": ["lib/*"],
      "@/features/*": ["features/*"]
    }
  }
}
```

**Benefits**:
- Absolute imports instead of `../../..`
- Easy refactoring
- Consistent import paths

**Usage**:
```tsx
// ✅ GOOD: Absolute import
import { Button } from "@/components/ui/button"

// ❌ BAD: Relative import
import { Button } from "../../components/ui/button"
```

---

## 🚀 RECOMMENDED: IDE Integration

### VS Code TypeScript Plugin

Enable the Next.js TypeScript plugin:

1. `Ctrl/⌘ + Shift + P`
2. "TypeScript: Select TypeScript Version"
3. "Use Workspace Version"

**What it provides**:
- Warning for invalid segment config options
- IntelliSense for available options
- Ensures `'use client'` is used correctly
- Validates client hooks only in Client Components

### TypeScript Version

Use TypeScript 5.1.3+ for async Server Components:

```json filename="package.json"
{
  "devDependencies": {
    "typescript": "^5.7.3"  // Latest version
  }
}
```

---

## ✅ RECOMMENDED: Typed Routes

### Enable Statically Typed Links

```ts filename="next.config.ts"
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typedRoutes: true,  // Enable typed routing
}

export default nextConfig
```

```json filename="tsconfig.json"
{
  "include": [
    "next-env.d.ts",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mark0025/npm-auth-gateway](https://github.com/Mark0025/npm-auth-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
