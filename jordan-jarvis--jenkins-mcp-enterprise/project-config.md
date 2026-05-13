---
trigger: always_on
description: This file provides guidance to cli agents when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to cli agents when working with code in this repository.

## Project Overview

This is a production-ready MCP (Model Context Protocol) server for Jenkins integration. The codebase uses modern Python patterns with proper dependency injection, error handling, and testing.

## Development Commands

### ⚠️ IMPORTANT: Development vs Production

**Development Mode:** Use `python3` commands for local development and testing
**Production Mode:** Use Docker deployment (recommended) for production environments

### Development Setup (Local Python)
```bash
# Install dependencies (use python3)
python3 -m pip install -e .

# Or install with user flag if permissions issues
python3 -m pip install --user -e .

# Optional: enable vector/semantic search (large ML deps; not installed by default)
python3 -m pip install -e ".[vector]"

# Optional: start local Qdrant for vector search
./scripts/start_dev_environment.sh

# Create configuration file from template
cp config/mcp-config.example.yml config/mcp-config.yml
# Edit config/mcp-config.yml with your Jenkins URLs and credentials

# Run MCP server locally with configuration
python3 -m jenkins_mcp_enterprise.server --config config/mcp-config.yml
```

### ⚙️ Configuration File Usage

The `--config` option is **required** for most operations to specify your Jenkins instances and settings:

```bash
# All development commands should use --config
npx @modelcontextprotocol/inspector --cli --method tools/list --server jenkins-mcp --config inspector-config.json 
```

**Without `--config`:** The server will have no Jenkins instances configured and most tools will fail.

### Production Setup (Docker - Recommended)
```bash
# 1. Create configuration file from template
cp config/mcp-config.example.yml config/mcp-config.yml
# Edit config/mcp-config.yml with your Jenkins URLs and credentials

# 2. Copy environment template and configure
cp .env.example .env
# Edit .env if needed (optional for file-based config)

# 3. Start production stack
docker compose up -d

# 4. Check deployment
docker compose ps
docker compose logs jenkins_mcp_enterprise-server

# 5. Stop stack
docker compose down
```

### Testing
```bash
# Run unit tests (legacy - needs refactoring)
python3 -m pytest tests/

# Run MCP integration tests (preferred)
python3 -m pytest tests/mcp_integration/ -v

# Run performance tests
python3 scripts/run_integration_tests.py --performance

# Run with coverage
python3 scripts/run_integration_tests.py --coverage

# Run specific massive scale tests
python3 tests/test_massive_scale_integration.py
```

### MCP Inspector Usage for Testing

#### Development Mode (Local Python)
```bash
# Install MCP inspector for manual testing
npm install -g @modelcontextprotocol/inspector


# Use --cli flag for command-line interface
npx @modelcontextprotocol/inspector --cli --method tools/list --server jenkins-mcp --config inspector-config.json 
npx @modelcontextprotocol/inspector --cli --method tools/call --tool-name diagnose_build_failure --tool-arg job_name=QA_JOBS/master build_number=1225 custom_error_patterns='''["error"]''' --server jenkins-mcp --config inspector-config.json 
For more usage and info refer to: https://modelcontextprotocol.io/llms-full.txt


#### Production Mode (Docker - Required for Production Testing)
```bash
# 1. Ensure Docker stack is running


#### Common Docker MCP Patterns
```bash
# Test sub-build discovery
docker run --rm --network jenkins_mcp_enterprise_mcp-net jenkins_mcp_enterprise-jenkins_mcp_enterprise-server:latest python3 -c "
from jenkins_mcp_enterprise.jenkins.connection_manager import JenkinsConnectionManager
from jenkins_mcp_enterprise.jenkins.subbuild_discoverer import SubBuildDiscoverer
from jenkins_mcp_enterprise.config import JenkinsConfig

config = JenkinsConfig(url='https://your-jenkins.com', username='user', token='token', timeout=30, verify_ssl=False)
connection = JenkinsConnectionManager(config)
discoverer = SubBuildDiscoverer(connection)
subbuilds = discoverer.discover_subbuilds('job/name', 123)
print(f'Found {len(subbuilds)} sub-builds')
"

# Test console log analysis
docker run --rm --network jenkins_mcp_enterprise_mcp-net jenkins_mcp_enterprise-jenkins_mcp_enterprise-server:latest python3 -c "
# Get console log for analysis
response = connection.session.get(f'{config.url}/job/QA_JOBS/job/develop/2089/consoleText')
lines = response.text.split('\n')
print(f'Console log: {len(lines)} lines')
"
```

### Development Environment
```bash
# Optional: start Qdrant vector database for semantic search
./scripts/start_dev_environment.sh

# Check Qdrant health
curl http://localhost:6333/health

# Access Qdrant dashboard
open http://localhost:6333/dashboard

# Stop environment
docker compose down
```

### Configuration Validation
```bash
# Validate configuration (use python3)
python3 -m jenkins_mcp_enterprise.cli validate --config config/mcp-config.yml

# Validate with custom config file
python3 -m jenkins_mcp_enterprise.cli validate --config config/my-config.yml
```

### Diagnostic Configuration

The `diagnose_build_failure` tool is fully configurable through YAML parameters with **advanced regex pattern support**. See comprehensive documentation:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jordan-Jarvis/jenkins-mcp-enterprise](https://github.com/Jordan-Jarvis/jenkins-mcp-enterprise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
