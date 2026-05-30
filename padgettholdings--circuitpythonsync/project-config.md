---
trigger: always_on
description: CircuitPythonSync is a TypeScript-based VS Code extension that provides development tools for Adafruit's CircuitPython microcontroller framework. The extension enables developers to sync files and libraries from PC to CircuitPython boards, manage CircuitPython libraries, explore board contents, and scaffold new projects.
---

# CircuitPythonSync VS Code Extension

CircuitPythonSync is a TypeScript-based VS Code extension that provides development tools for Adafruit's CircuitPython microcontroller framework. The extension enables developers to sync files and libraries from PC to CircuitPython boards, manage CircuitPython libraries, explore board contents, and scaffold new projects.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

- Bootstrap and build the repository:
  - `npm install` -- takes 2 minutes. NEVER CANCEL. Set timeout to 5+ minutes.
  - `npx eslint src/` -- lint source code only, takes ~5 seconds (avoids bundled dependency warnings)
  - `npm run webpack` -- development build, takes ~6 seconds  
  - `npm run vscode:prepublish` -- production build, takes ~15 seconds. NEVER CANCEL. Set timeout to 30+ minutes.
  - `npx vsce package` -- creates VSIX extension package, takes ~20 seconds. NEVER CANCEL. Set timeout to 60+ minutes.

- Test the extension:
  - `npm run test-compile` -- compiles tests but has known minizlib dependency issue (non-blocking)
  - Full VS Code extension testing requires VS Code installation with Extension Test Runner
  - Test files are located in `src/test/` directory
  - Manual testing requires loading extension in VS Code development window (F5 in VS Code)

## Validation

- Always run `npm run lint` before committing changes or the CI (.github/workflows/release.yml) will fail
- Always build with `npm run webpack` after making changes to verify compilation
- Manual validation requires VS Code with a CircuitPython project containing `code.py` or `main.py` files
- ALWAYS test extension functionality by installing the VSIX file in VS Code and exercising core features:
  - Board connection and drive mapping 
  - File sync to CircuitPython device
  - Library management and installation
  - Project template creation
- The extension activates on workspaces containing `code.py`, `main.py`, or `lib`/`Lib` folders

## Build System Details

- **Package Manager**: npm (version 10.8.2+)
- **Node.js**: Version 20.19.4+ required  
- **TypeScript**: Version 5.7.2+ for compilation
- **Bundler**: webpack 5.99.3+ for extension packaging
- **VS Code**: Extension targets VS Code 1.91.0+

## Key Dependencies and Issues

- **KNOWN WARNING**: `osx-temperature-sensor` dependency missing warning during webpack build - this is expected and non-critical
- **KNOWN ISSUE**: `npm run test-compile` fails due to minizlib TypeScript definitions conflict - use webpack builds instead
- **Extension Dependencies**: Requires `ms-python.python` extension
- **External Dependencies**: systeminformation, axios, tar, zip-lib for core functionality

## Common Tasks

### Repository Structure
```
.
├── .devcontainer/          # VS Code development container config
├── .github/workflows/      # CI/CD pipeline (release.yml)
├── .vscode/               # VS Code project settings
├── dist/                  # Webpack build output
├── out/                   # TypeScript compilation output  
├── resources/             # Extension assets (images, help files)
├── src/                   # TypeScript source code
│   ├── extension.ts       # Main extension entry point
│   ├── boardFileExplorer.ts # Board file system browser
│   ├── libraryMgmt.ts     # CircuitPython library management
│   ├── projectBundle.ts   # Project template system
│   ├── strings.ts         # String constants
│   └── test/              # Extension tests
├── package.json           # Extension manifest and dependencies
├── tsconfig.json          # TypeScript configuration
├── webpack.config.js      # Webpack bundling configuration
└── eslint.config.mjs      # ESLint linting rules
```

### Development Workflow
1. `npm install` -- install dependencies (first time setup)
2. `npm run webpack-dev` -- start development build with watch mode
3. Press F5 in VS Code to launch Extension Development Host
4. Test extension functionality in the development window
5. `npx eslint src/` -- validate source code style before committing (avoid `npm run lint` which includes bundled files)
6. `npx vsce package` -- create VSIX for distribution testing

### CI/CD Pipeline
- Release workflow triggers on version tags (v*)
- Builds on Ubuntu with Node.js 20.x
- Creates GitHub release with generated VSIX artifact
- See `.github/workflows/release.yml` for complete pipeline

### Extension Commands
The extension provides these VS Code commands (see package.json):
- `circuitpythonsync.helloWorld` -- Welcome and Help
- `circuitpythonsync.button1` -- CP Copy Files to Board  
- `circuitpythonsync.button2` -- CP Copy Libs to Board
- `circuitpythonsync.opendir` -- CP Set Drive
- `circuitpythonsync.mngcplibs` -- CP Manage Libs Copy
- `circuitpythonsync.libupdate` -- CP Install or Update Libraries and Stubs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [padgettholdings/circuitpythonsync](https://github.com/padgettholdings/circuitpythonsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
