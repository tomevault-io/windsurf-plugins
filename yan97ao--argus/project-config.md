---
trigger: always_on
description: Argus is a Python-based GitHub repository monitoring tool that tracks daily commits across specified repositories and generates automated reports using GitHub Issues. It includes optional AI analysis via DeepSeek API.
---

# Argus - GitHub Repository Monitoring Tool

Argus is a Python-based GitHub repository monitoring tool that tracks daily commits across specified repositories and generates automated reports using GitHub Issues. It includes optional AI analysis via DeepSeek API.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Setup
- Set up Python environment and install dependencies:
  - `python -m venv .venv` -- create virtual environment (recommended)
  - `source .venv/bin/activate` -- Linux/Mac activation
  - `.venv\Scripts\activate` -- Windows activation  
  - `pip install -r requirements.txt` -- takes 30-60 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
  
### Build and Test
- Validate the installation:
  - `PYTHONPATH=src python -c "from github_utils import *; print('GitHub utils loaded')"` -- test module imports
  - `PYTHONPATH=src python -c "from llm import *; print('LLM module loaded')"` -- test LLM module  
  - `python -m py_compile src/monitor.py src/github_utils.py src/llm.py` -- validate syntax
  - `python src/monitor.py --help` -- verify CLI functionality

### Run the Application
**IMPORTANT**: All configuration must be provided via environment variables. The application will fail to start if any required environment variable is missing.

- **Set up environment variables**:
  ```bash
  # Copy and configure the environment file
  cp .env.example .env
  # Edit .env with your actual values
  # Then load it
  source .env
  ```

- **Dry-run mode** (outputs report to console, does NOT create GitHub issues):
  ```bash
  source .env  # Load environment variables first
  python src/monitor.py --dry-run --enable-analysis
  ```

- **Dry-run with verbose logging** (outputs report and verbose logs, does NOT create issues):
  ```bash
  source .env  # Load environment variables first
  python src/monitor.py --dry-run --debug --enable-analysis
  ```

- **Production mode** (creates GitHub issues):
  ```bash
  source .env  # Load environment variables first
  python src/monitor.py --enable-analysis
  ```

### GitHub Actions Deployment
- Configure repository secrets in `Settings` → `Secrets and variables` → `Actions`:
  - `TOKEN`: GitHub personal access token (requires repo and issues permissions) - **REQUIRED**
  - `REPOSITORY`: Target repository for creating issues (format: owner/repo) - **REQUIRED**
  - `LLM_API_KEY`: DeepSeek API key for AI analysis - **REQUIRED**
  - `LLM_MODEL`: LLM model name (e.g., `deepseek-chat`) - **REQUIRED**
  - `LLM_BASE_URL`: LLM API endpoint (e.g., `https://api.deepseek.com/chat/completions`) - **REQUIRED**
- Enable GitHub Actions in `Settings` → `Actions` → `General`
- Workflow runs daily at 2 AM CST (18:00 UTC) or can be triggered manually

## Validation

### Manual Testing Scenarios
- **ALWAYS validate module imports** before making changes to ensure no syntax errors:
  ```bash
  PYTHONPATH=src python -c "from github_utils import *; print('GitHub utils loaded')"
  PYTHONPATH=src python -c "from llm import *; print('LLM module loaded')"
  python -m py_compile src/monitor.py src/github_utils.py src/llm.py
  ```
- **Test CLI functionality** with `--help` to verify argument parsing:
  ```bash
  python src/monitor.py --help
  ```
- **Validate GitHub Actions workflow syntax**:
  ```bash
  python3 -c "
  import yaml
  with open('.github/workflows/daily-update.yml', 'r') as f:
      workflow = yaml.safe_load(f.read())
      print('✓ GitHub Actions workflow syntax is valid')
      print(f'✓ Workflow name: {workflow.get(\"name\")}')
      triggers = workflow.get(True, {})  # 'on' becomes True in YAML parsing
      if isinstance(triggers, dict):
          print(f'✓ Has schedule: {\"schedule\" in triggers}')
          print(f'✓ Has manual trigger: {\"workflow_dispatch\" in triggers}')
      print(f'✓ Jobs: {list(workflow.get(\"jobs\", {}).keys())}')
  "
  ```
- **Run dry-run mode** to test the workflow without creating issues:
  - Monitor logs for successful GitHub API connections
  - Verify commit data retrieval from monitored repositories  
  - Check LLM analysis integration (when enabled)
  - Confirm issue content generation and formatting
- **Test production mode** only with test repositories to avoid spam in real projects

### Required Testing Steps
- **Import validation**: Test all Python modules to check for syntax errors
- **CLI validation**: Run the application with `--help` to verify argument parsing
- **GitHub Actions validation**: Check workflow file syntax with YAML parser
- **Dry-run testing**: Run the application in dry-run mode with valid credentials
- **API connectivity**: Verify the application can connect to GitHub API and retrieve repository data
- **Report generation**: Test the complete report generation workflow
- **Output validation**: Check that generated reports contain expected sections and formatting

## Common Tasks

### Repository Structure
```
argus/
├── src/
│   ├── monitor.py          # Main entry point and workflow orchestration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yan97ao/argus](https://github.com/yan97ao/argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
