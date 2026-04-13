---
trigger: always_on
description: This file provides guidance to Junie when working with code in this repository.
---

# Mojentic-MCP

This file provides guidance to Junie when working with code in this repository.

## Project Overview

Mojentic MCP is a library providing MCP server and client infrastructure for tools and agentic chat creators.

Key features:
- HTTP transport to expose the MCP protocol over HTTP
- STDIO transport to expose the MCP protocol over STDIO
- JSON-RPC 2.0 handler to handle standard MCP requests and responses

## Code Organization

### Import Structure
1. Imports should be grouped in the following order, with one blank line between groups:
   - Standard library imports
   - Third-party library imports
   - Local application imports
2. Within each group, imports should be sorted alphabetically

### Naming Conventions
1. Use descriptive variable names that indicate the purpose or content
2. Prefix test mock objects with 'mock_' (e.g., mock_memory)
3. Prefix test data variables with 'test_' (e.g., test_query)
4. Use '_' for unused variables or return values

### Type Hints and Documentation
1. Use type hints for method parameters and class dependencies
2. Include return type hints when the return type isn't obvious
3. Use docstrings for methods that aren't self-explanatory
4. Class docstrings should describe the purpose and behavior of the component
5. Follow Google-style docstrings (consistent with existing code)

### Logging Conventions
1. Use structlog for all logging
2. Initialize logger at module level using `logger = structlog.get_logger()`
3. Include relevant context data in log messages
4. Use appropriate log levels:
   - INFO for normal operations
   - DEBUG for detailed information
   - WARNING for concerning but non-critical issues
   - ERROR for critical issues
5. Use print statements only for direct user feedback

### Code Conventions
1. Do not write comments that just restate what the code does
2. Use pydantic BaseModel classes, do not use @dataclass
3. This project should use a pyproject.toml file and not a requirements.txt file for project requirements

## Testing Guidelines

### General Rules
1. Use pytest for all testing
2. Test files:
   - Named with suffix `_spec` (e.g., file_finder_spec.py)
   - Located beside the implementation file
3. Code style:
   - Max line length: 100 (as set in pyproject.toml)
   - Max complexity: 10
4. Run tests with: `pytest`
5. Run linting with: `flake8 src`

### BDD-Style Tests
We follow a Behavior-Driven Development (BDD) style using the "Describe/should" pattern to make tests readable and focused on component behavior.

#### Test Structure
1. Tests are organized in classes that start with "Describe" followed by the component name
2. Test methods:
   - Start with "it_should_"
   - Describe the expected behavior in plain English
   - Follow the Arrange/Act/Assert pattern (separated by blank lines)
3. Do not use comments (eg Arrange, Act, Assert) to delineate test sections - just use a blank line
4. No conditional statements in tests - each test should fail for only one clear reason
5. Do not test private methods directly (those starting with '_') - test through the public API

#### Fixtures and Mocking
1. Use pytest @fixture for test prerequisites:
   - Break large fixtures into smaller, reusable ones
   - Place fixtures in module scope for sharing between classes
   - Place module-level fixtures at the top of the file
2. Mocking:
   - Use pytest's `mocker` for dependencies
   - Use Mock's spec parameter for type safety (e.g., `Mock(spec=SmartMemory)`)
   - Only mock our own gateway classes
   - Do not mock library internals or private functions
   - Do not use unittest or MagicMock directly

#### Best Practices
1. Test organization:
   - Place instantiation/initialization tests first
   - Group related scenarios together (success and failure cases)
   - Keep tests focused on single behaviors
2. Assertions:
   - One assertion per line for better error identification
   - Use 'in' operator for partial string matches
   - Use '==' for exact matches
3. Test data:
   - Use fixtures for reusable prerequisites
   - Define complex test data structures within test methods

### Example Test
```python
class DescribeSmartMemory:
    """
    Tests for the SmartMemory component which handles memory operations
    """
    def should_be_instantiated_with_chroma_gateway(self):
        mock_chroma_gateway = Mock(spec=ChromaGateway)

        memory = SmartMemory(mock_chroma_gateway)

        assert isinstance(memory, SmartMemory)
        assert memory.chroma == mock_chroma_gateway
```
## Documentation

- Built with MkDocs and Material theme
- API documentation uses mkdocstrings
- Supports mermaid.js diagrams in markdown files:
  ```mermaid
  graph LR
      A[Doc] --> B[Feature]
  ```
- Build docs locally: `mkdocs serve`
- Build for production: `mkdocs build`
- Markdown files
    - Use `#` for top-level headings
    - Put blank lines above and below bulleted lists, numbered lists, headings, quotations, and code blocks
- Always keep the navigation tree in `mkdocs.yml` up to date with changes to the available documents in the `docs` folder

### API Documentation

API documentation uses mkdocstrings, which inserts module, class, and method documentation using certain markers in the markdown documents.

eg.

```
::: mojentic.llm.MessageBuilder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/svetzal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
