---
trigger: always_on
description: A Vite + React + TypeScript single-page application that generates one-click install badges for Model Context Protocol (MCP) servers. Users fill a form, and the app generates markdown badges with encoded JSON configurations for VS Code, VS Code Insiders, and Visual Studio.
---

# MCP Badge Creator - AI Coding Agent Instructions

## Project Overview
A Vite + React + TypeScript single-page application that generates one-click install badges for Model Context Protocol (MCP) servers. Users fill a form, and the app generates markdown badges with encoded JSON configurations for VS Code, VS Code Insiders, and Visual Studio.

## Architecture & Key Files

### Multi-Page React App (CSS Modules)
- **Routing**: `App.tsx` provides shell + navigation; pages live in `src/pages/` (`Home`, `MCP`, `Extensions`, `Packages`, `Settings`).
- **Styling**: Migrated from monolithic `App.css` to CSS Modules + design tokens.
- **Server Config Logic**: Still centralized inside `MCP.tsx` for badge + README generation.
- **Badge Utilities**: Extension + package badge helpers in `src/utils/`.

### Build & Deploy Pipeline
- **Vite bundler**: Fast dev server and optimized production builds
- **GitHub Actions**: `.github/workflows/deploy.yml` auto-deploys to GitHub Pages on push to main
- **Base path**: `vite.config.ts` sets `base: '/mcp-badge-creator/'` for GitHub Pages hosting
- **TypeScript**: Strict mode with separate configs (`tsconfig.app.json`, `tsconfig.node.json`)

## MCP Badge URL Patterns (Critical Domain Knowledge)

The core business logic encodes MCP server configurations into URL parameters:

### VS Code URLs
```typescript
// VS Code stable
https://vscode.dev/redirect/mcp/install?name={NAME}&config={ENCODED_JSON}

// VS Code Insiders
https://insiders.vscode.dev/redirect/mcp/install?name={NAME}&config={ENCODED_JSON}&quality=insiders
```

### Visual Studio URL
```typescript
https://vs-open.link/mcp-install?{ENCODED_JSON}
```

### Configuration JSON Structures
See `generateConfig()` in `App.tsx` for exact patterns:
- **HTTP**: `{ name, type: "http", url }`
- **NPX**: `{ name, command: "npx", args: ["-y", package], env: {} }`
- **UVX**: `{ name, command: "uvx", args: ["--from", from, package] || [package], env: {} }`
- **Docker**: `{ name, command: "docker", args: ["run", "-i", "--rm", image], env: {} }`
- **Local**: `{ name, command, args: [...], env: {} }`

## Development Workflow

### Local Development
```bash
npm install       # Install dependencies
npm run dev       # Start dev server at http://localhost:5173/mcp-badge-creator/
npm run build     # TypeScript compile + Vite build to ./dist
npm run preview   # Preview production build locally
npm run lint      # ESLint check (ALWAYS run after code changes)
npm run test:unit:run  # Run Vitest unit tests
```

### Code Quality Standards

**CRITICAL: Always run linter after making changes**

```bash
npm run lint      # Must pass with 0 errors before committing
```

#### TypeScript & ESLint Rules
- **Strict TypeScript**: No `any` types allowed (use proper interfaces/types)
- **No unused variables**: Remove or prefix with `_` if intentionally unused
- **Explicit types**: Prefer explicit type annotations for clarity
- **Type safety**: All function parameters and return types should be typed

#### Common Linting Errors & Fixes

**❌ Avoid:**
```typescript
const data: any = {...}  // No any types
const unused = 5;        // Unused variables
```

**✅ Do this:**
```typescript
const data: Record<string, string> = {...}
const _unused = 5;  // Or remove if truly unused

// Define proper types
interface EnvVar {
  key: string;
  value: string;
  password: boolean;
  inputName: string;
  inputDescription: string;
}
const envList: EnvVar[] = [];
```

#### Pre-Commit Checklist
1. ✅ Run `npm run lint` - must pass with 0 errors
2. ✅ Run `npm run test:unit:run` - all tests must pass
3. ✅ Run `npm run build` - verify TypeScript compilation succeeds
4. ✅ Test functionality in browser (`npm run dev`)
5. ✅ Check console for runtime errors

### Deployment Process
1. Push to `main` branch triggers GitHub Actions workflow
2. Workflow runs `npm ci` → `npm run build` → deploys `./dist` to GitHub Pages
3. Must configure repo Settings > Pages > Source: "GitHub Actions"
4. **Critical**: `vite.config.ts` base path must match repo name

## Project-Specific Conventions

### No Component Splitting
- Entire UI in single `App.tsx` component (intentional design choice)
- Inline styles via `className` targeting `App.css`
- Form sections and output sections in one file for simplicity

### Badge Color Scheme
From `App.tsx` and documented in memory:
- **VS Code**: `#0098FF` (blue) with `visualstudiocode` logo
- **VS Code Insiders**: `#24bfa5` (green) with `visualstudiocode` logo
- **Visual Studio**: `#C16FDE` (purple) with `visualstudio` logo
- UI theme matches VS Code Insiders green gradient

### State Management Pattern
```typescript
// Each config type has dedicated state variables
const [configType, setConfigType] = useState<ConfigType>('http')
const [serverUrl, setServerUrl] = useState('')      // for http
const [npxPackage, setNpxPackage] = useState('')    // for npx
const [uvxPackage, setUvxPackage] = useState('')    // for uvx
// ... and so on

// Conditional rendering based on configType
{configType === 'http' && <input ... />}
{configType === 'npx' && <input ... />}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesmontemagno/mcp-badge-creator](https://github.com/jamesmontemagno/mcp-badge-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
