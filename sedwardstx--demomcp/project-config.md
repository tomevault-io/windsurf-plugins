---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build and Install
```bash
# Install the package in development mode
pip install -e .

# Install with development dependencies
pip install -e ".[dev]"

# On Windows, ensure pywin32 is properly installed for Event Log access
# If you encounter issues, try:
pip install pywin32>=300
python -c "import win32api"  # Test Windows API access
```

### Code Quality
```bash
# Format code
black .
isort .

# Type checking
mypy src

# Linting
flake8

# Run all quality checks
black . && isort . && mypy src && flake8
```

### Testing
```bash
# Run all tests with proper PYTHONPATH
PYTHONPATH=src python3 -m pytest tests/ -v

# Run tests with coverage
PYTHONPATH=src python3 -m pytest --cov=mcp_log_analyzer tests/

# Run specific test file
PYTHONPATH=src python3 -m pytest tests/test_base_parser.py -v

# Test server import
PYTHONPATH=src python3 -c "from mcp_log_analyzer.mcp_server.server import mcp; print('Server import successful')"
```

### Running the MCP Server

**Important**: MCP servers don't show output when started - they communicate via stdin/stdout with MCP clients.

```bash
# Start the MCP server (runs silently)
python main.py

# Test the server is working
python check_server.py

# Add to Claude Code
claude mcp add mcp-log-analyzer python main.py

# List MCP servers
claude mcp list

# Remove MCP server
claude mcp remove mcp-log-analyzer
```

**No Output is Normal**: When you run `python main.py`, you won't see any console output. The server is running and waiting for MCP protocol messages from clients like Claude Code.

## Architecture Overview

### MCP Server Structure
This project follows the FastMCP framework pattern, refactored from the quick-data-mcp architecture:

- **Entry Point** (`main.py`): Simple script that imports and runs the MCP server
- **MCP Server** (`src/mcp_log_analyzer/mcp_server/server.py`): FastMCP server coordinator
- **Tools** (`src/mcp_log_analyzer/mcp_server/tools/`): Organized MCP tools by category
- **Resources** (`src/mcp_log_analyzer/mcp_server/resources/`): System monitoring resources
- **Prompts** (`src/mcp_log_analyzer/mcp_server/prompts/`): Comprehensive user guides
- **Core Logic** (`src/mcp_log_analyzer/core/`): Models and configuration
- **Parsers** (`src/mcp_log_analyzer/parsers/`): Log format-specific parsers

### Organized Tool Structure

**1. Core Log Management Tools** (`tools/log_management_tools.py`):
- `register_log_source`: Register new log sources (Windows Event Logs, JSON, XML, CSV, text)
- `list_log_sources`: List all registered log sources
- `get_log_source`: Get details about specific log source
- `delete_log_source`: Remove log source
- `query_logs`: Query logs with filters and time ranges
- `analyze_logs`: Perform pattern detection and anomaly analysis

**2. Windows System Tools** (`tools/windows_test_tools.py`):
- `test_windows_event_log_access`: Test Windows Event Log access and permissions
- `get_windows_event_log_info`: Get detailed Windows Event Log information
- `query_windows_events_by_criteria`: Query Windows Events with specific filters
- `get_windows_system_health`: Windows system health overview from Event Logs

**3. Linux System Tools** (`tools/linux_test_tools.py`):
- `test_linux_log_access`: Test Linux log file and systemd journal access
- `query_systemd_journal`: Query systemd journal with specific criteria
- `analyze_linux_services`: Analyze Linux services status and recent activity
- `get_linux_system_overview`: Comprehensive Linux system overview

**4. Process Monitoring Tools** (`tools/process_test_tools.py`):
- `test_system_resources_access`: Test system resource monitoring capabilities
- `analyze_system_performance`: Analyze current system performance and resource usage
- `find_resource_intensive_processes`: Find processes consuming significant resources
- `monitor_process_health`: Monitor health and status of specific processes
- `get_system_health_summary`: Overall system health summary

**5. Network Diagnostic Tools** (`tools/network_test_tools.py`):
- `test_network_tools_availability`: Test availability of network diagnostic tools
- `test_port_connectivity`: Test connectivity to specific ports
- `test_network_connectivity`: Test network connectivity to various hosts
- `analyze_network_connections`: Analyze current network connections and listening ports
- `diagnose_network_issues`: Comprehensive network diagnostics

### Organized Resource Structure

**1. Log Management Resources** (`resources/logs_resources.py`):
- `logs/sources`: List of registered log sources
- `logs/source/{name}`: Details about specific log source

**2. Windows Resources** (`resources/windows_resources.py`):
- `windows/system-events/{param}`: Windows System Event logs with configurable parameters
- `windows/application-events/{param}`: Windows Application Event logs with configurable parameters

**3. Linux Resources** (`resources/linux_resources.py`):
- `linux/systemd-logs/{param}`: Linux systemd journal logs with configurable parameters
- `linux/system-logs/{param}`: Linux system logs with configurable parameters

**4. Process Resources** (`resources/process_resources.py`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sedwardstx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
