---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ComfyUI-KikoTools is a planned modular collection of custom ComfyUI nodes that will provide essential tools missing from the standard ComfyUI release. All nodes will be grouped under "ComfyAssets" in the ComfyUI interface. The project is designed for extensibility, allowing new tools to be added easily while maintaining clean separation of concerns.

**Current Status**: Project is in initial planning phase. Only documentation and licensing files exist.

## Architecture

### Design Principles
- **Modular Design**: Each tool is a separate, self-contained module
- **ComfyAssets Grouping**: All nodes appear under the "ComfyAssets" category
- **Test-Driven Development**: Every tool includes comprehensive tests
- **Clean Interfaces**: Standardized input/output patterns across tools

### Core Components
- **Tool Registry**: Central registration system for all KikoTools nodes
- **Base Classes**: Shared functionality for consistent tool behavior
- **Individual Tools**: Self-contained modules for specific functionality

### Current Tools

#### 1. Resolution Calculator (First Tool)
- **Purpose**: Calculate upscale resolution from image or latent inputs
- **Inputs**: 
  - Image or Latent tensor
  - Scale factor (1, 2, 3, 1.2, 1.5, 2.0)
- **Outputs**: 
  - Width (INT)
  - Height (INT)
- **Target Models**: Flux and SDXL optimized
- **Use Case**: Connect calculated dimensions to upscaler nodes

## Technology Stack

- **Backend**: Python with ComfyUI node patterns
- **Node Framework**: ComfyUI INPUT_TYPES, RETURN_TYPES, execute() patterns
- **Testing**: pytest with ComfyUI test fixtures
- **Code Quality**: black, flake8, mypy
- **Integration**: ComfyUI execution queue and tensor systems

## Development Commands

**Note**: These commands are planned for when the project structure is implemented.

### Initial Setup
```bash
# Create basic project structure
mkdir -p kikotools/{base,tools} tests/{unit,integration,fixtures} scripts examples

# Create entry point files
touch __init__.py kikotools/__init__.py
```

### Code Quality (Future)
```bash
# Format Python code
black .

# Python linting
flake8 .

# Type checking
mypy .
```

### Testing (Future TDD Workflow)
```bash
# Run all tests
pytest tests/

# Run tests for specific tool
pytest tests/unit/tools/test_{tool_name}.py

# Test coverage
pytest --cov=kikotools tests/
```

## Project Structure (Planned)

**Current State**: Only `CLAUDE.md` and `LICENSE` files exist.

**Planned Structure**:
```
├── __init__.py                 # ComfyUI node registration entry point
├── kikotools/                  # Main package
│   ├── __init__.py            # Package initialization and tool registry
│   ├── base/                  # Base classes and shared utilities
│   │   ├── __init__.py
│   │   ├── base_node.py       # Base node class with ComfyAssets grouping
│   │   └── utils.py           # Shared utility functions
│   ├── tools/                 # Individual tool implementations
│   │   ├── __init__.py
│   │   ├── resolution_calculator/  # First planned tool
│   │   │   ├── __init__.py
│   │   │   ├── node.py        # ResolutionCalculatorNode implementation
│   │   │   └── logic.py       # Core calculation logic
│   │   └── template/          # Template for new tools
│   │       ├── __init__.py
│   │       ├── node.py
│   │       └── logic.py
├── tests/                     # Comprehensive test suite (TDD approach)
│   ├── __init__.py
│   ├── conftest.py           # pytest fixtures and ComfyUI test setup
│   ├── unit/                 # Unit tests for individual components
│   │   ├── test_base_node.py
│   │   └── tools/
│   │       └── test_resolution_calculator.py
│   ├── integration/          # ComfyUI integration tests
│   │   ├── test_node_registration.py
│   │   └── test_workflow_execution.py
│   └── fixtures/             # Test data and workflow files
│       ├── workflows/        # .json workflow files for testing
│       ├── images/          # Test images
│       └── latents/         # Test latent tensors
├── scripts/                  # Development automation
│   ├── create_tool.py       # Tool template generator
│   ├── register_tool.py     # Tool registration helper
│   └── validate_nodes.py    # Node validation script
├── examples/                 # Usage examples and demonstrations
│   ├── workflows/           # Example workflow .json files
│   └── documentation/       # Usage documentation per tool
└── requirements-dev.txt     # Development dependencies
```

## Key ComfyUI Integration Points

### Node Registration Pattern
```python
# Each tool follows this pattern in kikotools/tools/{tool_name}/node.py
class ResolutionCalculatorNode:
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "scale_factor": ("FLOAT", {"default": 2.0, "min": 1.0, "max": 8.0, "step": 0.1}),
            },
            "optional": {
                "image": ("IMAGE",),
                "latent": ("LATENT",),
            }
        }
    
    RETURN_TYPES = ("INT", "INT")
    RETURN_NAMES = ("width", "height")
    FUNCTION = "calculate_resolution"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ComfyAssets/ComfyUI-KikoTools](https://github.com/ComfyAssets/ComfyUI-KikoTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
