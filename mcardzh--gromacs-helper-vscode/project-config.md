---
trigger: always_on
description: This is a VS Code extension providing comprehensive language support for GROMACS molecular dynamics simulation files (`.mdp`, `.top`, `.gro`, `.pdb`, `.ndx`, `.xvg`, `.pka`, `.packmol`). The extension combines TextMate grammar-based syntax highlighting with programmatic semantic token providers, offering intelligent completion, validation, and visualization features.
---

# GROMACS Helper VS Code Extension - AI Coding Agent Instructions

## Project Overview

This is a VS Code extension providing comprehensive language support for GROMACS molecular dynamics simulation files (`.mdp`, `.top`, `.gro`, `.pdb`, `.ndx`, `.xvg`, `.pka`, `.packmol`). The extension combines TextMate grammar-based syntax highlighting with programmatic semantic token providers, offering intelligent completion, validation, and visualization features.

## Architecture Patterns

### Dual-Layer Syntax Highlighting
The project uses a **two-tier highlighting approach** that differs from typical VS Code extensions:

1. **TextMate Grammars** (`syntaxes/*/`) - Base syntax coloring via scope-based patterns
2. **Semantic Tokens Providers** (`src/providers/*SemanticTokensProvider.ts`) - Context-aware runtime highlighting

Semantic tokens override TextMate scopes and provide dynamic coloring based on residue types, parameter categories, etc. Defined in [baseSemanticTokensProvider.ts](src/providers/baseSemanticTokensProvider.ts) with custom token types registered in [package.json](package.json) `semanticTokenTypes`.

### Language Support Module Pattern
Each file format follows a consistent activation pattern in [src/languages/](src/languages/):

```typescript
export class XxxLanguageSupport {
  public activate(context: vscode.ExtensionContext): void {
    // Register providers: completion, hover, diagnostics, semantic tokens, etc.
    context.subscriptions.push(/* disposables */);
  }
}
```

Main entry point [src/extension.ts](src/extension.ts) instantiates and activates all language modules. **Critical**: Use `context.subscriptions.push()` for all disposables to prevent memory leaks.

### Domain-Specific Constant Definitions
- **Residue Types**: [src/constants/residueTypes.ts](src/constants/residueTypes.ts) - Amino acid classifications (acidic, basic, polar, nonpolar, aromatic, special, nucleotide, ion, water)
- **MDP Parameters**: [src/constants/mdpParameters.ts](src/constants/mdpParameters.ts) - All GROMACS 2025.2 parameters with categories, types, units, defaults, and validation rules

These drive both semantic highlighting and intelligent features like diagnostics/completion.

### Custom Snippet Management
Unlike standard snippet files, this project uses [src/snippetManager.ts](src/snippetManager.ts) to store user-editable snippets in global storage (`context.globalStorageUri`), not workspace files. Snippets support VSCode placeholder syntax (`${1:default}`, `${1|option1,option2|}`).

## Development Workflows

### Build & Package
```bash
npm run compile        # Development build via webpack
npm run watch          # Auto-rebuild on changes (use for development)
npm run package        # Production build (minified, hidden source maps)
./build.sh            # Interactive version bump + package VSIX
```

**Webpack Configuration**: [webpack.config.js](webpack.config.js) bundles TypeScript to `dist/extension.js`. The `media/` folder is copied to `dist/media/` for WebView resources (Packmol preview HTML).

### Testing
```bash
npm run pretest        # Compile tests + lint
npm test              # Run extension tests
npm run test:ci       # CI-specific test config
```

Tests use `@vscode/test-electron`. Test files in [src/test/](src/test/). Run via `npm: watch-tests` task for watch mode.

### Debugging
Use the "Run Extension" launch configuration (`.vscode/launch.json` if present). Webpack watch mode (`npm run watch`) enables live reload without repackaging.

## Key Technical Patterns

### Parameter Validation (MDP Files)
[src/providers/mdpDiagnosticProvider.ts](src/providers/mdpDiagnosticProvider.ts) performs two-pass validation:
1. Parse all parameter lines, check format (`parameter = value`)
2. Validate values against type/range rules from `MDP_PARAMETERS` constant

Example: Temperature must be positive number, `integrator` must be one of predefined values. **Important**: Diagnostics update on document change via `vscode.workspace.onDidChangeTextDocument`.

### Semantic Token Color Management
[src/providers/colorManager.ts](src/providers/colorManager.ts) singleton manages runtime color updates via `vscode.window.createTextEditorDecorationType()`. Colors read from workspace config `gromacsHelper.colors.*` and apply via decoration ranges, **not** via theme contribution (allows user customization without theme editing).

### WebView Panels (3D Visualization)
Packmol preview uses both:
- **Panel**: [PackmolPreviewPanel.ts](src/providers/packmolPreviewPanel.ts) - Standalone preview window
- **Sidebar View**: [PackmolPreviewProvider.ts](src/providers/packmolPreviewProvider.ts) - Activity bar webview

Both render [media/packmol_preview.html](media/packmol_preview.html) with Three.js for 3D geometry visualization. **Critical**: Use `asWebviewUri()` for all resource references in webviews.

### Residue-Aware Highlighting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcardZH/gromacs-helper-vscode](https://github.com/mcardZH/gromacs-helper-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
