---
trigger: always_on
description: **Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# Jupyter MCP Server

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

Jupyter MCP Server is a Python-based Model Context Protocol (MCP) server implementation that enables real-time interaction with Jupyter Notebooks. The project uses a modern Python build system with hatch, and includes comprehensive testing, linting, and documentation.

## Working Effectively

### Environment Setup
- **Python Requirements**: Python 3.10 or higher (tested with 3.9-3.13)
- **Network Considerations**: PyPI installs may fail due to SSL certificate issues or timeout limitations. This is a known environment constraint.

### Build and Install (CRITICAL: Network Limitations)
```bash
# Standard installation (may fail with network issues)
pip install ".[test,lint,typing]"

# Alternative if pip install fails:
# 1. Install dependencies individually with longer timeouts
pip install --timeout=300 pytest
pip install --timeout=300 ruff  
pip install --timeout=300 mypy

# 2. Or use Docker approach (preferred for consistency)
docker build -t jupyter-mcp-server .
```

**NETWORK TIMEOUT WARNING**: pip install commands may fail with SSL certificate errors or read timeouts when connecting to PyPI. If installs fail:
- Try increasing timeout: `pip install --timeout=300`
- Use Docker build which handles dependencies internally
- Document the network limitation in any testing notes

### Core Development Commands
```bash
# Development installation (when network allows)
make dev
# Equivalent to: pip install ".[test,lint,typing]"

# Basic installation  
make install
# Equivalent to: pip install .

# Build the package
make build
# Equivalent to: pip install build && python -m build .
```

### Testing (CRITICAL: Use Long Timeouts)
```bash
# Run tests using hatch (when available)
make test
# Equivalent to: hatch test

# Run tests directly with pytest (when network allows install)
pytest .

# NEVER CANCEL: Test suite timing expectations
# - Full test suite: Allow 15-20 minutes minimum
# - Network-dependent tests may take longer
# - Set timeout to 30+ minutes for safety
```

**VALIDATION REQUIREMENT**: When testing is not possible due to network issues, verify at minimum:
```bash
# Syntax validation (always works)
python -m py_compile jupyter_mcp_server/server.py
find . -name "*.py" -exec python -m py_compile {} \;

# Import validation
PYTHONPATH=. python -c "import jupyter_mcp_server; print('Import successful')"
```

### Linting and Code Quality (CRITICAL: Use Long Timeouts)
```bash
# Full linting pipeline (when network allows)
bash ./.github/workflows/lint.sh

# Individual linting commands:
pip install -e ".[lint,typing]"
mypy --install-types --non-interactive .  # May take 10+ minutes, NEVER CANCEL
ruff check .                              # Quick, usually <1 minute  
mdformat --check *.md                     # Quick, usually <1 minute
pipx run 'validate-pyproject[all]' pyproject.toml  # 2-3 minutes

# TIMING WARNING: mypy type checking can take 10+ minutes on first run
# Set timeout to 20+ minutes for mypy operations
```

### Running the Application

#### Local Development Mode
```bash
# Start with streamable HTTP transport
make start
# Equivalent to:
jupyter-mcp-server start \
  --transport streamable-http \
  --document-url http://localhost:8888 \
  --document-id notebook.ipynb \
  --document-token MY_TOKEN \
  --runtime-url http://localhost:8888 \
  --start-new-runtime true \
  --runtime-token MY_TOKEN \
  --port 4040
```

#### JupyterLab Setup (Required for Testing)
```bash
# Start JupyterLab server for MCP integration
make jupyterlab
# Equivalent to:
pip uninstall -y pycrdt datalayer_pycrdt
pip install datalayer_pycrdt
jupyter lab \
  --port 8888 \
  --ip 0.0.0.0 \
  --ServerApp.root_dir ./dev/content \
  --IdentityProvider.token MY_TOKEN
```

#### Docker Deployment
```bash
# Build Docker image (NEVER CANCEL: Build takes 10-15 minutes)
make build-docker  # Takes 10-15 minutes, set timeout to 20+ minutes

# Run with Docker  
make start-docker
# Or manually:
docker run -i --rm \
  -e DOCUMENT_URL=http://localhost:8888 \
  -e DOCUMENT_ID=notebook.ipynb \
  -e DOCUMENT_TOKEN=MY_TOKEN \
  -e RUNTIME_URL=http://localhost:8888 \
  -e START_NEW_RUNTIME=true \
  -e RUNTIME_TOKEN=MY_TOKEN \
  --network=host \
  datalayer/jupyter-mcp-server:latest
```

### Manual Validation Scenarios

**When full testing is not possible due to network constraints, always verify:**

1. **Syntax and Import Validation**:
   ```bash
   # Validate all Python files compile
   find . -name "*.py" -exec python -m py_compile {} \;
   
   # Test local imports work
   PYTHONPATH=. python -c "import jupyter_mcp_server; print('SUCCESS')"
   ```

2. **Configuration Validation**:
   ```bash
   # Verify pyproject.toml is valid
   python -c "import tomllib; tomllib.load(open('pyproject.toml', 'rb'))"
   
   # Test module structure
   python -c "import jupyter_mcp_server.server, jupyter_mcp_server.models"
   ```

3. **Documentation Build** (when Node.js available):
   ```bash
   cd docs/
   npm install  # May have network issues

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datalayer/jupyter-mcp-server](https://github.com/datalayer/jupyter-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
