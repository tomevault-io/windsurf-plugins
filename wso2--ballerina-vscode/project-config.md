---
trigger: always_on
description: The `@wso2/bi-diagram` package is a React-based flow diagram renderer specifically designed for visualizing Ballerina code blocks (functions, resource functions, and other code constructs). It leverages the `@projectstorm/react-diagrams` library to create interactive, visual flow diagrams that represent the execution flow of Ballerina code.
---

# BI-Diagram Package Context

## Overview

The `@wso2/bi-diagram` package is a React-based flow diagram renderer specifically designed for visualizing Ballerina code blocks (functions, resource functions, and other code constructs). It leverages the `@projectstorm/react-diagrams` library to create interactive, visual flow diagrams that represent the execution flow of Ballerina code.

## Architecture

### Core Pattern: Visitor Pattern
The package uses the **Visitor Pattern** extensively to process flow data and transform it into renderable diagram models. The main workflow involves:

1. **Data Processing**: Multiple visitors traverse the flow data to calculate positions, sizes, and relationships
2. **Model Creation**: Visitors create node models that conform to react-diagrams API
3. **Rendering**: React components render the models using react-diagrams engine

### Key Dependencies
- `@projectstorm/react-diagrams` (v7.x) - Core diagramming library
- `@wso2/ballerina-core` - Provides core types and utilities for Ballerina AST
- `@wso2/ui-toolkit` - Shared UI components
- `lodash` - Utility functions
- `React 18.2.0`

## Main Components

### 1. Diagram Component (`src/components/Diagram.tsx`)
**Purpose**: Main orchestrating component that initializes the diagram engine and manages the overall diagram state.

**Key Features**:
- Manages diagram engine lifecycle
- Handles breakpoint integration
- Supports error flow visualization
- Provides callback mechanisms for node interactions
- Manages diagram zoom and positioning

**Key Methods**:
- `getDiagramData()` - Orchestrates visitor execution to process flow data
- `drawDiagram()` - Creates and renders the diagram model
- `getErrorHandlerIdForActiveBreakpoint()` - Handles breakpoint-based error handler expansion

### 2. Visitors (`src/visitors/`)
**Purpose**: Implement the visitor pattern to process flow data and prepare it for rendering.

#### Core Visitors:
- **`InitVisitor`**: Initializes node view states and handles error handler expansion
- **`SizingVisitor`**: Calculates dimensions for all nodes based on content and type
- **`PositionVisitor`**: Determines x,y coordinates for optimal diagram layout
- **`NodeFactoryVisitor`**: Creates react-diagram node models from flow data
- **`LinkTargetVisitor`**: Establishes connections between nodes
- **`BreakpointVisitor`**: Applies breakpoint information to nodes

#### Utility Visitors:
- **`AddNodeVisitor`**: Handles adding new nodes to the flow
- **`RemoveNodeVisitor`**: Manages node removal
- **`RemoveEmptyNodesVisitor`**: Cleans up empty nodes from the flow

### 3. Node Types (`src/components/nodes/`)
Each node type consists of three files following react-diagrams pattern:
- `*NodeModel.ts` - Data model extending react-diagrams NodeModel
- `*NodeWidget.tsx` - React component for rendering
- `*NodeFactory.tsx` - Factory for creating instances

#### Available Node Types:
- **`BaseNode`**: Standard statement/action nodes
- **`IfNode`**: Conditional branching with multiple branches
- **`WhileNode`**: Loop constructs
- **`ApiCallNode`**: HTTP/API call operations
- **`AgentCallNode`**: AI agent invocations with tool management
- **`PromptNode`**: AI prompt nodes for model interactions
- **`StartNode`/`EndNode`**: Flow start and end markers
- **`EmptyNode`**: Placeholder for empty flow sections
- **`DraftNode`**: Nodes in draft/editing state
- **`CommentNode`**: Documentation annotations
- **`ButtonNode`**: Interactive action buttons
- **`ErrorNode`**: Error handling display
- **`WaitEventNode`**: Workflow wait/pause node (circle + external dashed arrow + details)

### 4. Links and Ports (`src/components/NodePort/`, `src/components/NodeLink/`)
- **NodePort**: Connection points on nodes (input/output)
- **NodeLink**: Visual connections between nodes with routing capabilities

## Type System

### Core Types (from `@wso2/ballerina-core`)
```typescript
- Flow: Root container for the entire flow
- FlowNode: Individual node in the flow with branches and properties
- Branch: Container for child nodes (e.g., if-then, if-else, while-body)
- LineRange: Source code position information
- NodePosition: Diagram positioning data
- ToolData: AI tool configuration data
- AgentData: AI agent configuration
```

### Diagram-Specific Types
```typescript
- NodeModel: Union type of all possible node models
- LinkableNodeModel: Nodes that can have links (excludes ButtonNode)
- FlowNodeStyle: Visual styling options ("default" | "ballerina-statements")
```

## Key Features

### 1. Breakpoint Integration
- Supports active breakpoint highlighting
- Automatic error handler expansion when breakpoint is in onFailure branches
- Visual indicators for breakpoint states

### 2. AI Integration
- Agent nodes with tool management
- Prompt nodes for model interactions
- Support for multiple AI model providers (OpenAI, Anthropic, Azure, Ollama, etc.)
- Tool selection and configuration UI

### 3. Interactive Editing
- Add/remove nodes through UI interactions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wso2/ballerina-vscode](https://github.com/wso2/ballerina-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
