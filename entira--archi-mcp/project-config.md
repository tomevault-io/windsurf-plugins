---
trigger: always_on
description: > 📖 **For complete project overview, features, and user documentation, see [README.md](README.md)**
---

# 🏗️ ArchiMate MCP Server - Claude Code Assistant Configuration

## Project Overview
> 📖 **For complete project overview, features, and user documentation, see [README.md](README.md)**

This document contains development-specific instructions and technical details for working with the ArchiMate MCP Server codebase.

## Tech Stack
> 📖 **For complete tech stack and dependencies, see [README.md](README.md#development)**

**Key Development Tools:**
- **UV** for fast package management and virtual environments
- **pytest** for comprehensive testing framework
- **black** + **ruff** for code formatting and linting
- **mypy** for type checking

## Requirements

- Python 3.11+
- uv (recommended) or pip
- Git
- Java 8+ (for PlantUML)
- PlantUML JAR file (see installation command below)

## Development Commands

### Setup
```bash
# Clone the repository
git clone https://github.com/entira/archi-mcp.git
cd archi-mcp

# Install dependencies
uv sync

# Install development dependencies
uv sync --dev

# Download PlantUML JAR (required for diagram generation)
curl -L https://github.com/plantuml/plantuml/releases/latest/download/plantuml.jar -o plantuml.jar
```

### Testing
```bash
# Run all tests (194 passing tests, 66% coverage)
uv run pytest

# Run tests with verbose output
uv run pytest -v

# Run tests with coverage
uv run pytest --cov=archi_mcp --cov-report=html

# Quick test for development
uv run pytest -xvs
```

### Test Coverage Summary

The test suite includes 194 passing tests with 66% code coverage across 13 test files:

- **Core Functionality**: Server operations, MCP protocol integration
- **ArchiMate Elements**: All 55+ element types across 7 layers
- **Relationships**: All 12 relationship types with validation
- **PlantUML Generation**: Syntax generation and validation
- **XML Export**: ArchiMate Exchange Format export and validation
- **Multi-Language**: Slovak/English detection and translation
- **Error Handling**: Comprehensive error scenarios and recovery
- **Layout Options**: Direction, spacing, grouping configurations

### Server Operations
```bash
# Start MCP server for Claude Desktop
uv run python src/archi_mcp/server.py

# Test server initialization
uv run python -c "from archi_mcp.server import mcp; print('✅ Server ready')"

# Generate sample diagrams
uv run python examples/generate_sample_diagrams.py

# Start HTTP server for diagram viewing (if needed separately)
uv run python -m http.server 8080 --directory exports
```

### Debugging
```bash
# Check generated exports
ls -la exports/

# View latest PlantUML code
cat exports/*/diagram.puml | head -20

# Test PlantUML rendering manually
java -Djava.awt.headless=true -jar plantuml.jar -tpng exports/*/diagram.puml

# Check HTTP server logs
tail -f exports/http_server.log

# Validate XML export
xmllint --noout exports/*/archimate_model.archimate
```

### Code Quality
```bash
# Format code
uv run black src tests
uv run isort src tests

# Lint code
uv run ruff src tests

# Type checking
uv run mypy src
```

## Available MCP Tools
> 📖 **For complete MCP tools documentation and usage examples, see [README.md](README.md#mcp-tools)**

**Development Notes:**
- Both tools are defined in `server.py` with full type annotations
- Input validation uses Pydantic models for schema enforcement
- Error handling includes comprehensive logging for debugging
- Tools are automatically discovered by FastMCP protocol

**Output Files (saved to exports/YYYYMMDD_HHMMSS/):**
- `diagram.puml` - PlantUML source code
- `diagram.png` - Production-ready PNG image
- `diagram.svg` - Vector SVG format (if generated)
- `architecture.md` - Extended documentation with embedded images
- `generation.log` - Comprehensive debug information
- `metadata.json` - Diagram statistics and metadata
- `archimate_model.archimate` - Archi-compatible XML format

## Project Structure
> 📖 **For complete project structure overview, see [README.md](README.md#project-structure)**

**Development-Specific Directories:**
```
tests/                     # 194 passing tests, 66% coverage
├── test_server.py         # Core server functionality
├── test_elements.py       # Element definitions
├── test_relationships.py  # Relationship validation
├── test_generator.py      # PlantUML generation
├── test_xml_export.py     # XML export functionality
└── ...                    # Additional test modules

exports/                   # Generated output (gitignored)  
plantuml.jar               # PlantUML JAR file (download separately)
.github/                   # CI/CD workflows (if any)
```

## Claude Desktop Integration
> 📖 **For complete Claude Desktop configuration, see [README.md](README.md#claude-desktop-configuration)**

**Development Configuration:**
- Use absolute path to your local development directory
- Enable `DEBUG` log level for development: `"ARCHI_MCP_LOG_LEVEL": "DEBUG"`
- Consider enabling experimental features for testing: `"ARCHI_MCP_ENABLE_VALIDATION": "true"`

**Example Development Config:**
```json
{
  "mcpServers": {
    "archi-mcp": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/your/archi-mcp", "python", "-m", "archi_mcp.server"],
      "cwd": "/path/to/your/archi-mcp",
      "env": {
        "ARCHI_MCP_LOG_LEVEL": "DEBUG",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [entira/archi-mcp](https://github.com/entira/archi-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
