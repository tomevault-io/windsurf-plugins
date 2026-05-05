---
trigger: always_on
description: A VS Code extension that generates interactive Miro-style diagrams for Solidity smart contracts. When invoked on a function, it displays:
---

# Solidity Diagram Extension

## Project Overview
A VS Code extension that generates interactive Miro-style diagrams for Solidity smart contracts. When invoked on a function, it displays:
- The main function code
- Referenced data structures (structs, enums)
- State variable declarations
- Inner function calls with their implementations
- Visual arrows connecting references to definitions

## Architecture

```
src/
├── extension.ts              # VS Code extension entry point
├── parser/
│   ├── solidityParser.ts     # Solidity AST parsing using @solidity-parser/parser
│   └── astTraverser.ts       # AST walking utilities
├── analyzer/
│   ├── functionAnalyzer.ts   # Main analysis orchestrator
│   ├── typeResolver.ts       # Resolves struct/enum definitions
│   ├── callGraphBuilder.ts   # Builds function call graph
│   ├── stateVariableResolver.ts  # Resolves state variable declarations
│   ├── dataFlowAnalyzer.ts   # Data flow tracking for DeFi code review
│   └── inheritanceResolver.ts # Interface/inheritance resolution & library method tracking
├── renderer/
│   ├── webviewProvider.ts    # VS Code webview panel management
│   ├── diagramGenerator.ts   # Generates HTML diagram
│   ├── canvasController.ts   # Miro-style pan/zoom canvas
│   ├── draggableBlocks.ts    # Drag functionality for code blocks
│   ├── arrowManager.ts       # Dynamic arrow connections
│   ├── syntaxHighlight.ts    # Solidity syntax highlighting
│   ├── importManager.ts      # Dynamic Cmd+Click import functionality
│   ├── dataFlowVisualizer.ts # Data flow hover/click visualization
│   └── notesManager.ts       # Text annotations/notes on the canvas
├── types/index.ts            # TypeScript type definitions
└── utils/sourceMapper.ts     # Source code mapping utilities
```

## Key Technologies
- **@solidity-parser/parser**: Parses Solidity into AST
- **VS Code Webview API**: Renders interactive HTML diagrams
- **Vanilla JS Canvas**: Custom pan/zoom/drag implementation (no external libs)

## Commands
- `npm install` - Install dependencies
- `npm run compile` - Build TypeScript
- `npm run watch` - Watch mode for development
- Press **F5** in VS Code to debug the extension

## Usage
1. Open a `.sol` file
2. Right-click inside a function
3. Select "Generate Function Diagram"
4. Interact with the diagram:
   - **Pan**: Drag the dotted background
   - **Zoom**: Mouse wheel
   - **Move blocks**: Drag the header (title bar) of any block
   - **Scroll code**: Hover over code area and scroll
   - **Resize blocks**: Drag corner or edges to resize
   - **Navigate**: Click "Go to source" to jump to code
   - **Import**: Cmd+Click (Mac) or Ctrl+Click (Windows/Linux) on highlighted tokens to import definitions
   - **Remove**: Click the X button on non-main blocks to remove them

## What Gets Displayed
- **Main Function**: The selected function with full source code
- **Data Structures**: Structs and enums used anywhere in the function, resolved from ALL workspace files:
  - Variable declarations: `DepositPool memory pool_`
  - Struct instantiation: `DepositPool({...})`
  - Enum comparisons: `strategy_ == Strategy.NO_YIELD`
- **State Variables**: Contract state variables referenced in the function (importable via Cmd+Click)
- **Internal Calls**: Only functions with actual definitions in the workspace

## Interface & Library Resolution
The extension now resolves interface calls to actual implementations:

### Interface Calls
- **Pattern Detection**: Recognizes `IContractName(address).methodName()` patterns with nested parentheses
- **Multiple Implementations**: Shows picker dialog when multiple implementations exist
- **Workspace Search**: Searches all `.sol` files for contracts implementing the interface
- **Inheritance Tracking**: Follows `is InterfaceName` declarations to find implementations

### Library Extension Methods
- **Using Directives**: Parses `using LibraryName for Type` statements (global and contract-level)
- **Method Resolution**: Resolves calls like `token.safeApprove()` to `SafeERC20.safeApprove()`
- **Context-Aware**: Uses calling contract's scope to find correct library attachments
- **Workspace + Dependencies**: Searches both workspace and `node_modules`

### External Dependencies
The extension scans `node_modules` for common Solidity libraries:
- **OpenZeppelin Contracts**: `@openzeppelin/contracts/**/*.sol`
- **Solmate**: `solmate/src/**/*.sol`
- **Solady**: `solady/src/**/*.sol`
- **Forge-std**: `forge-std/src/**/*.sol`

### Implementation Picker
When multiple implementations exist for an interface call:
1. User Cmd+Clicks on the method name
2. Modal dialog shows all implementations with:
   - Contract name and file path
   - Contract kind badge (contract, abstract, library)
   - Location in workspace or dependencies
3. User selects desired implementation
4. Code block is imported and displayed

## What Gets Excluded (After Resolution Attempts)
- Interface calls without found implementations (shows signature only)
- Type casts: `address(0)`, `uint256(value)`
- Built-in calls: `require()`, `keccak256()`, `abi.encode()`
- External library static calls without definitions

## Dynamic Import Feature

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xrudra99/SolidityDiagram](https://github.com/0xrudra99/SolidityDiagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
