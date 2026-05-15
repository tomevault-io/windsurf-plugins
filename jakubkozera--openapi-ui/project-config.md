---
trigger: always_on
description: OpenAPI UI is a multi-platform OpenAPI/Swagger documentation and testing interface distributed as:
---

# OpenAPI UI - AI Agent Instructions

## Project Overview

OpenAPI UI is a multi-platform OpenAPI/Swagger documentation and testing interface distributed as:
- **Core** (`src/core/`) - Vanilla JS/CSS bundle (no frameworks)
- **C# NuGet Package** (`src/c-sharp/OpenApiUi/`) - ASP.NET Core middleware
- **VS Code Extension** (`src/vsc-extension/openapi-ui/`) - TypeScript extension
- **Sample API** (`src/c-sharp/OpenApiUi.Sample/`) - Demo ASP.NET Core application

## Architecture Fundamentals

### Build System & Distribution Flow

**Critical workflow**: Core JavaScript/CSS → Bundled assets → Embedded in C# library AND VS Code extension

1. **Core Build** (`src/core/`)
   - Run `node build.js` to bundle JS/CSS into `dist/` directory
   - Run `node demo-build.js` to create demo with sample specs in `demo-dist/`
   - Bundles concatenate files in **specific order** (defined in build scripts) - order matters!
   - Build output: `dist/bundle.js`, `dist/bundle.css`, `dist/index.html`
   - Demo mode adds `js/demoMode.js` and `css/demo-mode.css` to bundles

2. **C# Package Integration**
   - Copy bundled assets from `src/core/dist/` to `src/c-sharp/OpenApiUi/Content/`
   - Assets are embedded as resources in `.csproj` using `<EmbeddedResource>` pattern
   - Middleware serves from embedded resources via `EmbeddedFileProvider`
   - Placeholders in HTML: `#swagger_path#` → replaced at runtime, `#base_url#` → current host

3. **VS Code Extension Integration**
   - Copy bundled assets from `src/core/dist/` to `src/vsc-extension/openapi-ui/core-dist/`
   - Extension uses webpack to bundle TypeScript → `dist/extension.js`
   - Webview loads core assets using `webview.asWebviewUri()` for VS Code security

### Module System

**No modern bundlers** - all modules are vanilla JS concatenated in order:
- Files declare functions in global scope or attach to `window` object
- Module order in `build.js` matters: utilities first, then features, then app initialization
- Example pattern:
  ```javascript
  // In js/auth.js
  window.auth = {
    initAuth: function() { /* ... */ },
    getAccessToken: function() { /* ... */ }
  };
  
  // In other files
  if (window.auth) {
    window.auth.initAuth();
  }
  ```

## Development Workflows

### Building Changes

**Always build in this order when modifying core UI:**

```bash
# 1. Modify files in src/core/js/ or src/core/css/
# 2. Build core bundle
cd src/core
node build.js

# 3. Copy to C# project
# Manually copy dist/* to src/c-sharp/OpenApiUi/Content/

# 4. Copy to VS Code extension
# Manually copy dist/* to src/vsc-extension/openapi-ui/core-dist/

# 5. Build C# NuGet (if releasing)
cd src/c-sharp/OpenApiUi
dotnet pack -c Release

# 6. Build VS Code extension (if releasing)
cd src/vsc-extension/openapi-ui
npm run compile
# or for production: npm run package
```

### Testing Locally

**C# Package:**
```bash
cd src/c-sharp/OpenApiUi.Sample
dotnet run
# Access: http://localhost:5120/openapi-ui
```

**VS Code Extension:**
- Press `F5` in VS Code (opens Extension Development Host)
- Use Command Palette: "Open OpenAPI UI"
- Add sources via sidebar activity bar icon

**Demo Mode:**
```bash
cd src/core
node demo-build.js
# Serve demo-dist/ folder with any HTTP server
# Click "Demo Mode" button to load sample specs
```

## Critical Conventions

### File Naming & Module Loading Order

**Never reorder files** in `build.js` or `demo-build.js` without understanding dependencies:
- `config.js` must be first (defines globals)
- `utils.js` early (used by many modules)
- `monacoSetup.js` before editor-dependent modules
- `app.js` must be last (initialization)

### Adding New JavaScript Modules

1. Create file in `src/core/js/`
2. Export functions via `window.moduleName = { ... }` pattern
3. Add to `jsFiles` array in `build.js` at appropriate position
4. Also add to `demo-build.js` if demo-relevant
5. Rebuild bundles

### Monaco Editor Integration

- CDN loaded via `monacoSetup.js`
- Editors initialized asynchronously - always check `window.monaco` exists
- Three editor instances: request body, response body, code snippets
- Theme syncing: `monacoSetup.setupMonacoThemeListener()` bridges app theme to Monaco

### Authentication System

Multi-scheme OAuth2/OIDC/Bearer/Basic/ApiKey support in `js/auth.js`:
- Security config loaded from OpenAPI `components.securitySchemes`
- Per-operation security requirements in `operation.security` array
- Tokens stored in localStorage with API title/version namespacing
- Pattern: `{api_title}_{api_version}_access_token_{scheme_key}`

### Variables & Collection Runner

Variables system (`js/variables.js`) for Postman-like workflows:
- Variables: `{{variableName}}` replaced in requests
- Output variables: Extract from responses via JSONPath (example: `$.data.id`)
- Storage namespaced by API title/version
- Collection Runner executes requests sequentially with variable chaining

### Placeholder System

HTML template uses runtime replacements:
- `#swagger_path#` - OpenAPI spec URL (C# middleware, VS Code extension)
- `#base_url#` - Server base URL (C# middleware only)
- Bundle paths updated in middleware for configurable UI paths

## C# Package Specifics

### Middleware Architecture

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakubkozera/openapi-ui](https://github.com/jakubkozera/openapi-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
