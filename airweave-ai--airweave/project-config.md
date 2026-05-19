---
trigger: always_on
description: **End-to-end testing framework for Airweave connectors using real external APIs**
---

# Monke - Airweave Connector Testing Framework

**End-to-end testing framework for Airweave connectors using real external APIs**

## Overview

Monke validates Airweave's data synchronization pipeline by creating real test data in external systems, triggering sync jobs, verifying data in the vector database, and testing update/deletion scenarios. The framework separates configuration from runtime state, uses pluggable authentication, and supports parallel execution.

---

## Core Architecture

**Test Runner** (`runner.py`) - Entry point that orchestrates parallel test execution across connectors

**Test Flow** (`core/flow.py`) - Executes complete lifecycle for a connector: setup → test steps → cleanup

**Test Steps** (`core/steps.py`) - Individual operations (create, sync, verify, update, delete) with retry logic

**Configuration** (`core/config.py`) - YAML-based config with classes: `TestConfig`, `ConnectorConfig`, `TestFlowConfig`, `DeletionConfig`

**Test Context** (`core/context.py`) - Runtime state separate from config: tracks entities, infrastructure IDs, metrics

**Infrastructure** (`core/infrastructure.py`) - Creates/tears down test collections and source connections

**Authentication** (`auth/`) - Pluggable auth brokers (`BaseAuthBroker`, `ComposioBroker`) for credential resolution

---

## Test Flow

Default test steps execute in sequence:
```
1. collection_cleanup       # Clean leftover test collections
2. cleanup                  # Clean entities in external system
3. create                   # Create test entities
4. sync                     # Trigger Airweave sync
5. verify                   # Verify entities in vector DB
6. update                   # Update entities
7. sync → verify            # Verify updates
8. partial_delete           # Delete subset of entities
9. sync → verify_partial_deletion
10. verify_remaining_entities
11. complete_delete         # Delete all entities
12. sync → verify_complete_deletion
13. cleanup                 # Final cleanup
14. collection_cleanup      # Delete test collection
```

Steps are customizable per connector via YAML config.

---

## Bongos

**Bongos** create, update, and delete real test data in external systems.

### Structure
Each bongo inherits from `BaseBongo` and implements:
- `create_entities()` - Create test data
- `update_entities()` - Modify test data
- `delete_entities()` - Remove all test data
- `delete_specific_entities(entities)` - Remove specific entities
- `cleanup()` - Force cleanup remaining artifacts

Bongos are auto-discovered by `BongoRegistry` via the `connector_type` class attribute.

### Example
```python
class GitHubBongo(BaseBongo):
    def __init__(self, credentials: Dict[str, Any], config: Dict[str, Any]):
        super().__init__(credentials)
        self.repo_name = config["repo_name"]  # Now from config_fields
        self.token = credentials["personal_access_token"]

    async def create_entities(self) -> List[Dict[str, Any]]:
        # Create files in GitHub, return metadata

    async def update_entities(self) -> List[Dict[str, Any]]:
        # Update files

    async def delete_entities(self) -> List[str]:
        # Delete all tracked entities
```

---

## Configuration

Configs live in `configs/{connector_type}.yaml`:

```yaml
name: "GitHub Connector Test"
description: "End-to-end test of GitHub connector"

connector:
  name: "github_test"
  type: "github"
  auth_mode: direct  # or "provider"
  auth_fields:
    personal_access_token: MONKE_GITHUB_PERSONAL_ACCESS_TOKEN
  config_fields:
    repo_name: MONKE_GITHUB_REPO_NAME  # Moved from auth_fields
    branch: "main"
    post_create_sleep_seconds: 15

test_flow:
  steps: [cleanup, create, sync, verify, update, sync, verify, complete_delete, sync, verify_complete_deletion, cleanup]

entity_count: 3

deletion:
  partial_delete_count: 1
  verify_partial_deletion: true

verification:
  score_threshold: 0.1
  max_retries: 3
  retry_delay_seconds: 5
```

Environment variables are substituted using `${VAR_NAME}` syntax.

---

## Authentication

### Direct Mode
Credentials specified via `auth_fields` with env var names:
```yaml
auth_mode: direct
auth_fields:
  personal_access_token: MONKE_GITHUB_PERSONAL_ACCESS_TOKEN
```

### Provider Mode
External auth providers (e.g., Composio) fetch credentials:
```bash
DM_AUTH_PROVIDER=composio
DM_AUTH_PROVIDER_ID=provider_id
DM_AUTH_PROVIDER_API_KEY=api_key
GITHUB_AUTH_PROVIDER_AUTH_CONFIG_ID=ac_xxx
GITHUB_AUTH_PROVIDER_ACCOUNT_ID=ca_xxx
```

Auth resolution:
1. `credentials_resolver` determines mode
2. Direct: resolves env vars from `auth_fields`
3. Provider: uses `ComposioBroker` to fetch credentials
4. Credentials passed to bongo and Airweave source connection

---

## Content Generation

Generators in `generation/{connector}.py` create realistic test data with embedded tracking tokens:

```python
from monke.client.llm import LLMClient

async def generate_github_content(token: str) -> str:
    llm = LLMClient()
    prompt = f"Generate Python code. Include '{token}' in a comment."
    return await llm.generate(prompt)
```

Pydantic schemas in `generation/schemas/{connector}.py` define entity structures.

---

## Verification

After each sync, verification steps:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
