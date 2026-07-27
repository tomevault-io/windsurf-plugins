---
trigger: always_on
description: This repository contains the VS Code extension for the "Power Query Connector SDK". This document provides context to help GitHub Copilot assist with development tasks, particularly converting test frameworks and creating regression tests.
---

# Copilot Instructions for Power Query Connector SDK Extension

This repository contains the VS Code extension for the "Power Query Connector SDK". This document provides context to help GitHub Copilot assist with development tasks, particularly converting test frameworks and creating regression tests.

![!IMPORTANT]
You are allowed to ask questions.
If you're unsure about any aspect or if the task lacks necessary information, say "I don’t have enough information to confidently assess this."

**Required Process**:

- **Plan First**:
  When given a complex task, before changing any code, analyze the task and then create a `*.task.md`
  to describe step by step how you will implement the task. Ask the user to review the plan.
  Revise the plan based on feedback.
- **Wait for Approval**: Do not start coding until the user explicitly says "Execute the task"
- **Checkpoint Journal**
  When executing the tasks, use `copilot-journal.md` to track progress and decisions.
- **Test After Each Change**: Run tests to verify functionality
- **Stop when Stuck**: If you hit a roadblock, such as syntax error, test failures, update the journal, stop and ask for help
- **Update Journal**: Mark checkpoints as complete
- **Simple Completion**: When done, just say "I'm done" (no summary needed)
- **Change log**:
  When you finish a task related to an end user feature, update the `CHANGELOG.md` file with a summary of changes.
  Increase the patch version number by running `npm version patch` and commit the change.
  Include these steps when generating the task plan.

## Project Overview

**Extension Name**: Power Query Connector SDK for Visual Studio Code  
**Publisher**: PowerQuery  
**Extension ID**: PowerQuery.vscode-powerquery-sdk

### Purpose

This extension is used to create custom connectors for Power Query in Power BI. Power Query connectors are written in the M language. The extension provides:

- Project creation and management
- Building connector files (.mez)
- Credential management
- Query testing and evaluation

### Dependencies

- **Required Extension**: `PowerQuery.vscode-powerquery` - provides M language service
- **Core Tool**: Microsoft.PowerQuery.SdkTools NuGet package containing:
    - `pqtest.exe` - M evaluation and testing
    - `pqservicehost.exe` - service-based M evaluation (experimental)
    - `makepqx.exe` - connector building utility

## Architecture Overview

### Key Components

1. **Extension Core** (`src/extension.ts`)
    - Main activation point
    - Dependency injection setup
    - Service initialization

2. **Testing Services**
    - `PqTestExecutableTaskQueue` - Queue-based execution of pqtest.exe
    - `PqServiceHostClient` - RPC client for pqservicehost.exe (experimental)
    - Both implement `IPQTestService` interface

3. **Test Output Processing**
    - pqtest.exe outputs JSON to stdout
    - Extension parses JSON and displays in webview
    - Results shown in `PQTest result` tab

4. **Configuration**
    - Extension settings in `package.json` under `contributes.configuration`
    - User settings validation via `schemas/UserSettings.schema.json`

## Current Testing Structure

### Test Directories

1. **`unit-tests/`** - Standard unit tests using Mocha
    - Tests utility classes and business logic
    - Run with: `npm run test:unit-test`

2. **`src/test/suite/`** - UI tests using `@vscode/test-electron` framework
    - Files: `extension.test.ts`, `project.test.ts`, `schema.test.ts`
    - Uses `.vscode-test.mjs` configuration
    - Clean test environment with dependency extensions

3. **`src/test/commonSuite/`** - UI tests using `vscode-extension-tester` framework
    - Files: `NewProject.spec.ts`, `PqSdkToolAcquisition.spec.ts`
    - Legacy framework that needs migration
    - Uses `scripts/test-e2e.ts` for execution

### Test Utilities (`src/test/utils/`)

- `editorUtils.ts` - VS Code editor interactions
- `settingUtils.ts` - Extension settings management
- `sideBarUtils.ts` - Sidebar/tree view interactions
- `connectorProjects.ts` - Project creation helpers
- `pqSdkNugetPackageUtils.ts` - NuGet package verification

## pqtest.exe Integration

### JSON Output Format

pqtest.exe operations return JSON to stdout with structures like:

```typescript
interface GenericResult {
  // Basic operation results
}

interface ExtensionInfo {
  // Connector metadata from "info" operation
  Members: Array<{
    Name: string;
    FunctionParameters?: Array<...>;
    // ... other metadata
  }>;
}

interface Credential {
  // Authentication information from "list-credential"
}
```

### Key Operations

- `info` - Get connector metadata
- `run-test` - Execute M query, returns evaluation result
- `test-connection` - Test connector connectivity
- `list-credential` - List stored credentials
- `set-credential` - Store authentication data
- `delete-credential` - Remove credentials

### Command Line Usage

```bash
PQTest.exe <operation> --extension <connector.mez> --queryFile <query.pq> [options]
```

### Test Settings Schema

- Files ending in `.testsettings.json` validated against `schemas/UserSettings.schema.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/vscode-powerquery-sdk](https://github.com/microsoft/vscode-powerquery-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
