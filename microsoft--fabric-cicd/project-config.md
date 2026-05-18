---
trigger: always_on
description: fabric-cicd is a Python library for Microsoft Fabric CI/CD automation. It supports code-first Continuous Integration/Continuous Deployment automations to integrate Source Controlled workspaces into a deployment framework.
---

# Fabric CICD

fabric-cicd is a Python library for Microsoft Fabric CI/CD automation. It supports code-first Continuous Integration/Continuous Deployment automations to integrate Source Controlled workspaces into a deployment framework.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Quick Command Reference

**Prerequisites**: Requires Python 3.9+

| Task         | Command                                                                                  | Timeout |
| ------------ | ---------------------------------------------------------------------------------------- | ------- |
| Setup        | `pip install uv && uv sync --dev` (NEVER CANCEL)                                         | 120+s   |
| Test         | `uv run pytest -v` (NEVER CANCEL)                                                        | 120+s   |
| Import check | `uv run python -c "from fabric_cicd import FabricWorkspace; print('Import successful')"` | 30s     |
| Format       | `uv run ruff format` (Fix formatting issues)                                             | 60s     |
| Lint check   | `uv run ruff check` (Check for linting issues)                                           | 60s     |
| Format check | `uv run ruff format --check` (Verify formatting is correct)                              | 60s     |
| Docs build   | `uv run mkdocs build --clean` (Build documentation)                                      | 60s     |
| Docs serve   | `uv run mkdocs serve` (Start local documentation server)                                 | 60s     |

**Mandatory Validation (ALWAYS):**

1. Import check → 2. Run tests → 3. Format code → 4. Check linting → 5. Commit

**Critical**: NEVER cancel build/test commands. CI (`.github/workflows/validate.yml`) will fail if validation workflow incomplete.

## Authentication

Must provide explicit `token_credential` parameter to `FabricWorkspace`.

**Methods:**

- **Local development**: `AzureCliCredential()` or `AzurePowerShellCredential()`
- **CI/CD pipelines**: `ClientSecretCredential()` with service principal
- **Testing/imports**: No authentication needed

**Example:**

```python
from azure.identity import AzureCliCredential
from fabric_cicd import FabricWorkspace

token_credential = AzureCliCredential()
workspace = FabricWorkspace(
    workspace_id="your-id",
    repository_directory="/path/to/workspace/items",
    token_credential=token_credential
)
```

## Basic Usage

### Programmatic API

```python
from azure.identity import AzureCliCredential
from fabric_cicd import FabricWorkspace, publish_all_items, unpublish_all_orphan_items

token_credential = AzureCliCredential()
# Initialize workspace (supports either workspace_id OR workspace_name)
workspace = FabricWorkspace(
    workspace_id="your-workspace-id",  # Alternative: workspace_name="your-workspace-name"
    environment="DEV",
    repository_directory="/path/to/workspace/items",
    item_type_in_scope=["Notebook", "DataPipeline", "Environment"],
    token_credential=token_credential
)

# Deploy items
publish_all_items(workspace)

# Clean up orphaned items
unpublish_all_orphan_items(workspace)
```

### Config-Based Deployment

Alternative: `deploy_with_config()` centralizes deployment settings in YAML.

```python
from azure.identity import AzureCliCredential
from fabric_cicd import deploy_with_config
token_credential = AzureCliCredential()
result = deploy_with_config(
    config_file_path="config.yml",
    environment="dev",
    token_credential=token_credential
)
```

**Implementation files:**

- Entry points: `deploy_with_config()`, `publish_all_items()`, `unpublish_all_orphan_items()` in `src/fabric_cicd/publish.py`
- Config utilities: `src/fabric_cicd/_common/_config_utils.py` (loading, extraction)
- Config validation: `src/fabric_cicd/_common/_config_validator.py`
- Documentation: `docs/how_to/config_deployment.md`
- Tests: `tests/test_deploy_with_config.py`, `tests/test_config_validator.py`

### Public API Exports

Only import from the top-level package (`src/fabric_cicd/__init__.py`). Do not import internal modules directly.

**Exported symbols:**

- `FabricWorkspace` - Main workspace management class
- `publish_all_items` - Deploy all items in scope
- `unpublish_all_orphan_items` - Remove orphaned items
- `deploy_with_config` - Config-based deployment
- `DeploymentResult`, `DeploymentStatus` - Deployment result types
- `ItemType` - Enum of supported Fabric item types
- `FeatureFlag` - Enum of feature flags
- `append_feature_flag` - Add feature flags programmatically
- `change_log_level`, `configure_external_file_logging`, `disable_file_logging` - Logging utilities

## Project Structure

```
/
├── .github/workflows/    # CI/CD pipelines (test.yml, validate.yml, bump.yml)
├── docs/                # Documentation source files
├── docs/example/        # CI/CD scenario patterns (Azure DevOps, GitHub Actions, local development)
├── sample/              # Example workspace structure and items
├── src/fabric_cicd/     # Main library source code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/fabric-cicd](https://github.com/microsoft/fabric-cicd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
