---
trigger: always_on
description: This is the **Robot Developer Extensions for ROS 2** - a Visual Studio Code extension that provides debugging and development support for Robot Operating System 2 (ROS 2). The extension works with Visual Studio Code and Cursor.
---

# Project Overview
This is the **Robot Developer Extensions for ROS 2** - a Visual Studio Code extension that provides debugging and development support for Robot Operating System 2 (ROS 2). The extension works with Visual Studio Code and Cursor.

## Technology Stack
* **Language**: TypeScript + Python
* **Framework**: VS Code Extension API
* **Build Tool**: Webpack + npm
* **Testing**: Mocha (via VS Code test runner)
* **Target ROS**: ROS 2 Humble or greater (no ROS 1 support)
* **Supported Languages**: rclpy (Python), rclcpp (C++), rclrust (Rust), rcldotnet (.NET)

# Contribution Guidelines

## Building and Testing
1. **Install dependencies**: `npm ci` (preferred over `npm install`)
2. **Build the project**: `npm run build` (runs `npm run package` which executes webpack in production mode)
3. **Lint code**: Focus on passing the build; `npm run lint` is configured but ESLint setup needs verification
4. **Run tests**: Open Debug viewlet (`Ctrl+Shift+D`), select `Tests`, then hit `F5`
5. **Debug extension**: Open Debug viewlet, select `Extension`, then hit `F5`

## Code Quality Requirements
* All new features and bug fixes **must** include test cases in the `test` directory
* Changes must pass linting and build successfully
* Require/import statements must always be at the top of a file, never in the middle
* Follow `.editorconfig` settings: 2 spaces, LF line endings, insert final newline
* Be concise and to the point in code and documentation

## Acceptance Criteria for Changes
* Code must build without errors: `npm run build`
* Existing tests must continue to pass
* New functionality must include tests
* No introduction of security vulnerabilities
* Changes must not break ROS 2 environment detection or debugging capabilities

# Directory Structure
```
├── src/              # TypeScript source code
│   ├── debugger/     # Debugging functionality (attach, launch, process picker)
│   ├── ros/          # ROS-specific implementations (ROS 2 commands, lifecycle nodes)
│   ├── build-tool/   # Build tool integration (colcon)
│   └── test-provider/ # Test discovery and execution
├── test/             # Test suites and fixtures
│   ├── suite/        # Test implementations
│   └── launch/       # ROS launch file test fixtures
├── assets/           # Scripts and resources bundled with extension
├── scripts/          # Build and maintenance scripts
├── samples/          # Sample ROS 2 workspaces for testing
├── dist/             # Compiled output (do not commit)
├── docs/             # User-facing documentation (builds public docs)
└── spec/             # Technical specifications and design proposals
```

# Documentation Guidelines

## User Documentation (`/docs`)
The `/docs` directory contains user-facing documentation that is built into public documentation:
* **User guides** - How to use features (tutorials, quick starts)
* **Configuration** - Setup and configuration instructions
* **Troubleshooting** - Common issues and solutions
* **Usage examples** - Code samples and walkthroughs

**Do not place** technical specifications or design proposals in `/docs`.

## Technical Specifications (`/spec`)
The `/spec` directory contains technical specifications and design documents:
* **Feature proposals** - Design documents for new features
* **Technical specifications** - Detailed technical designs and architecture
* **API specifications** - Interface and protocol definitions
* **Design mockups** - UI/UX designs and wireframes

Each feature should have its own subdirectory in `/spec` containing all related specification documents.

# Code Style and Conventions

## TypeScript Style
```typescript
// Good: Use async/await for asynchronous operations
async function fetchRosNodes(): Promise<string[]> {
    const nodes = await ros2.getNodeList();
    return nodes;
}

// Good: Proper error handling
try {
    await debugManager.launchNode(config);
} catch (error) {
    vscode.window.showErrorMessage(`Failed to launch: ${error}`);
    throw error;
}

// Good: Use typed interfaces
interface LaunchConfig {
    target: string;
    arguments: string[];
    env?: Record<string, string>;
}
```

## Python Style
* Python code is used for interfacing directly with ROS 2
* Python code runs in a managed virtual environment in the extension directory
* Follow Python best practices for ROS 2 integration

# Security and Safety Boundaries

## Forbidden Actions
* **Never** commit secrets, API keys, or credentials to the repository
* **Never** remove or disable security-related code without explicit justification
* **Do not** create summary documents or planning files (work in memory)
* **Do not** use vcpkg for dependency management (use Pixi instead)

## Protected Areas
* `.venv/` directories contain Python virtual environments (git-ignored)
* `node_modules/` contains npm dependencies (git-ignored)
* Build artifacts in `dist/` and `out/` directories (git-ignored)
* Do not commit `.vscode/settings.json` or `.vscode/c_cpp_properties.json`

# Special Considerations

## ROS 2 Environment Setup
* When executing ROS commands, source the ROS 2 setup script first:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ranch-Hand-Robotics/rde-ros-2](https://github.com/Ranch-Hand-Robotics/rde-ros-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
