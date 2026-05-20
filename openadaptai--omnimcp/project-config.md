---
trigger: always_on
description: This document describes how to implement OmniMCP, a system for UI automation through visual understanding and the Model Context Protocol (MCP).
---

# CLAUDE.md - OmniMCP Implementation Guide

## Overview
This document describes how to implement OmniMCP, a system for UI automation through visual understanding and the Model Context Protocol (MCP).

## Core Architecture

The system consists of these essential components:

1. VisualState - Current screen state
2. MCP Server - Protocol implementation
3. Input Control - UI actions
4. UI Parser Integration - Visual analysis

## Implementation Approach

### 1. Start with VisualState
```python
class VisualState:
    def __init__(self):
        self.elements = []
        self.timestamp = None
        self.screen_dimensions = None
        
    def update(self, screenshot):
        """Update visual state from screenshot.
        
        Critical function that maintains screen state.
        Must handle:
        - Screenshot capture
        - UI element parsing
        - State updates
        - Coordinate normalization
        """
```

### 2. Implement Core MCP Server
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("omnimcp")

@mcp.tool()
async def get_screen_state() -> ScreenState:
    """Get current state of visible UI elements"""
    
@mcp.tool()
async def click_element(description: str) -> ClickResult:
    """Click UI element matching description"""

@mcp.tool() 
async def type_text(text: str) -> TypeResult:
    """Type text"""
```

### 3. Build Element Targeting
```python
def find_element(description: str) -> Element:
    """Find UI element matching description.
    
    Critical for action reliability.
    Consider:
    - Text matching
    - Element type
    - Location/context
    - Confidence scores
    """
```

## Implementation Order

1. Visual State Management
   - Screenshot capture
   - UI parsing
   - State updates
   - Basic caching

2. MCP Protocol
   - Observe endpoint
   - Simple actions
   - Rich responses
   - Error handling

3. Action System
   - Element targeting
   - Input simulation
   - Action verification
   - Error recovery

## Key Considerations

### State Management
- Always update before actions
- Cache intelligently
- Track history when needed
- Clear invalidation

### Error Handling
- Rich error context
- Recovery strategies
- Debug information
- Verification

### Performance
- Minimize updates
- Smart caching
- Async where beneficial
- Efficient targeting

## MCP Protocol Details

### Observe
```python
@dataclass
class UIElement:
    content: str
    type: str
    bounds: Bounds
    confidence: float

@dataclass
class ScreenState:
    elements: List[UIElement]
    dimensions: tuple[int, int]
    timestamp: float

@dataclass
class ActionResult:
    success: bool
    element: Optional[UIElement]
    error: Optional[str] = None
```

## Code Structure

Current implementation:
```
./
├── omnimcp/             # Main package directory
│   ├── omnimcp.py       # Core implementation with OmniMCP class and VisualState
│   ├── input.py         # Input controller for UI interactions
│   ├── types.py         # Type definitions (Bounds, UIElement, etc.)
│   ├── utils.py         # Utilities for screenshots, coordinates, etc.
│   ├── config.py        # Centralized configuration
│   └── omniparser/      # UI parsing functionality
│       ├── client.py    # Parser client and provider
│       └── server.py    # Parser deployment and management
├── tests/               # Test directory
│   ├── test_synthetic_ui.py  # Synthetic UI generation for testing
│   └── test_omnimcp.py       # Core functionality tests
└── run_omnimcp.py       # Command-line entry point
```

Planned expansion:
```
./
├── utils.py              # Core utilities and input control
├── omniparser/          # UI parsing functionality
│   ├── client.py        # Parser client and provider
│   └── server.py        # Parser deployment and management
├── core/               # Future: Core state management
│   ├── visual_state.py
│   └── element.py
└── mcp/                # Future: MCP implementation
    └── server.py
```

## Package Management

OmniMCP uses `uv` for dependency management. When adding new dependencies, use:

```bash
uv add <package-name>       # Add a regular dependency
uv add --dev <package-name> # Add a development dependency
uv pip install -e .         # Install all dependencies
```

This ensures dependencies are properly recorded in pyproject.toml.

## Configuration System

OmniMCP now uses a centralized configuration system with:

- Settings loaded from environment variables and `.env` file
- Default values for all settings
- Support for various configuration types:
  - Claude API settings
  - OmniParser connection settings 
  - AWS deployment configuration
  - Debug and logging settings

To configure OmniMCP, create a `.env` file in the project root with your settings:

## Implementation Notes

### Core Principles
1. Visual state is always current
2. Every action verifies completion
3. Rich error context always available
4. Debug information accessible

### Critical Functions
1. VisualState.update()
2. MCPServer.observe()
3. find_element()
4. verify_action()

### Error Handling
```python
@dataclass
class ToolError:
    message: str
    visual_context: Optional[bytes]  # Screenshot
    attempted_action: str

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAdaptAI/OmniMCP](https://github.com/OpenAdaptAI/OmniMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
