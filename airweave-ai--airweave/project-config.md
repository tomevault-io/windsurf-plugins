---
trigger: always_on
description: Generates realistic test content using LLM.
---

# Building Monke Tests for Source Connectors

## Overview

**Monke** is Airweave's end-to-end testing framework for source connectors. It creates real test data in external systems, triggers syncs, and verifies data appears correctly in the search index.

This guide shows you how to build comprehensive tests that verify **every entity type** your connector supports.

---

## Why Test Every Entity Type?

**Important: Your Monke tests should create and verify all entity types defined in your source connector.**

Many connector tests only verify top-level entities (e.g., tasks) but ignore nested entities (e.g., comments, files).

**Impact:**
- Comments might not sync properly → Silent failures in production
- File attachments might not be indexed → Missing searchable content
- Entity relationships might be broken → Poor search results
- Users can't search the full breadth of data they expect

**Solution:** Create test entities for every entity type your connector syncs, and verify each one appears in Qdrant.

**Before Writing Monke Tests:**
1. Open your source file: `backend/airweave/platform/sources/{short_name}.py`
2. List all entity types yielded in `generate_entities()`:
   - Example: WorkspaceEntity, ProjectEntity, TaskEntity, CommentEntity, FileEntity
3. Your Monke tests should create at least one instance of each type
4. Verify each instance appears in Qdrant after sync

**Validation:**
- Count entity classes in `entities/{short_name}.py`
- Count entity types created in `bongos/{short_name}.py::create_entities()`
- These counts should match (excluding parent/workspace entities that don't get stored)

---

## Core Components

Every Monke test requires four components:

1. **Bongo implementation** (`monke/bongos/{short_name}.py`)
2. **Generation schemas** (`monke/generation/schemas/{short_name}.py`)
3. **Generation adapter** (`monke/generation/{short_name}.py`)
4. **Test configuration** (`monke/configs/{short_name}.yaml`)

---

## Part 1: Bongo Implementation

The **Bongo** is a class that creates, updates, and deletes test data via the external API.

### File Location
```
monke/bongos/{short_name}.py
```

### Basic Structure

```python
"""{Connector Name} bongo implementation.

Creates, updates, and deletes test entities via the real {Connector Name} API.
"""

import asyncio
import time
import uuid
from typing import Any, Dict, List, Optional

import httpx
from monke.bongos.base_bongo import BaseBongo
from monke.utils.logging import get_logger


class MyConnectorBongo(BaseBongo):
    """Bongo for {Connector Name} that creates test entities for E2E testing.

    Key responsibilities:
    - Create test entities (including nested types like comments/files)
    - Embed verification tokens in content
    - Update entities to test incremental sync
    - Delete entities to test deletion detection
    - Clean up all test data
    """

    connector_type = "{short_name}"  # Must match source short_name

    API_BASE = "https://api.example.com/v1"

    def __init__(self, credentials: Dict[str, Any], **kwargs):
        """Initialize the bongo.

        Args:
            credentials: Dict with "access_token" or other auth
            **kwargs: Configuration from test config file
        """
        super().__init__(credentials)
        self.access_token: str = credentials["access_token"]

        # Test configuration
        self.entity_count: int = int(kwargs.get("entity_count", 3))
        self.openai_model: str = kwargs.get("openai_model", "gpt-4.1-mini")
        self.max_concurrency: int = int(kwargs.get("max_concurrency", 3))

        # Simple rate limiting (optional, add if needed)
        self.last_request_time = 0.0
        self.min_delay = 0.2  # 200ms between requests

        # Runtime state - track ALL created entities
        self._workspace_id: Optional[str] = None
        self._project_id: Optional[str] = None
        self._tasks: List[Dict[str, Any]] = []
        self._comments: List[Dict[str, Any]] = []
        self._files: List[Dict[str, Any]] = []

        self.logger = get_logger(f"{self.connector_type}_bongo")

    async def create_entities(self) -> List[Dict[str, Any]]:
        """Create ALL types of test entities.

        This is critical: You must create instances of EVERY entity type
        that your source connector syncs.

        Returns:
            List of entity descriptors with verification tokens
        """
        raise NotImplementedError("Implement in subclass")

    async def update_entities(self) -> List[Dict[str, Any]]:
        """Update a subset of entities to test incremental sync.

        Returns:
            List of updated entity descriptors
        """
        raise NotImplementedError("Implement in subclass")

    async def delete_entities(self) -> List[str]:
        """Delete all created test entities.

        Returns:
            List of deleted entity IDs
        """
        raise NotImplementedError("Implement in subclass")

    async def delete_specific_entities(self, entities: List[Dict[str, Any]]) -> List[str]:
        """Delete specific entities by ID.

        Args:
            entities: List of entity descriptors to delete

        Returns:
            List of successfully deleted entity IDs
        """
        raise NotImplementedError("Implement in subclass")


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
