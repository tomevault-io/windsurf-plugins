---
trigger: always_on
description: A **source connector** in Airweave is a Python module that extracts data from an external service and transforms it into searchable entities. This guide covers everything you need to build a production-ready connector.
---

# Building a Source Connector in Airweave

## Overview

A **source connector** in Airweave is a Python module that extracts data from an external service and transforms it into searchable entities. This guide covers everything you need to build a production-ready connector.

There are two types of source connectors:

1. **Standard (Sync-Based)**: Extracts and syncs all data from the source to Airweave's vector database
2. **Federated Search**: Searches the source's API at query time without syncing data

## Core Components

Every source connector requires three main components:

1. **Source implementation** (`backend/airweave/platform/sources/{short_name}.py`)
2. **Entity schemas** (`backend/airweave/platform/entities/{short_name}.py`)
3. **OAuth configuration** (`backend/airweave/platform/auth/yaml/dev.integrations.yaml`)

---

## Part 1: Entity Schemas

Start with entities because they define your data model.

### File Location
```
backend/airweave/platform/entities/{short_name}.py
```

### Entity Types

There are two base entity types:

1. **ChunkEntity** - Text-based entities (tasks, messages, documents, etc.)
2. **FileEntity** - File attachments (PDFs, images, etc.)

### Basic Structure

```python
"""Entity schemas for {Connector Name}."""

from datetime import datetime
from typing import Any, Dict, List, Optional

from pydantic import Field

from airweave.platform.entities._airweave_field import AirweaveField
from airweave.platform.entities._base import ChunkEntity, FileEntity


class MyConnectorEntity(ChunkEntity):
    """Schema for primary entity type."""

    # Required fields
    name: str = AirweaveField(
        ...,
        description="Display name of the entity",
        embeddable=True  # This field will be embedded for search
    )

    # Timestamps (critical for incremental sync)
    created_at: Optional[datetime] = AirweaveField(
        None,
        description="When this entity was created",
        embeddable=True,
        is_created_at=True  # Marks this as the creation timestamp
    )

    modified_at: Optional[datetime] = AirweaveField(
        None,
        description="When this entity was last modified",
        embeddable=True,
        is_updated_at=True  # Marks this as the update timestamp
    )

    # Content fields
    content: Optional[str] = AirweaveField(
        None,
        description="The main text content",
        embeddable=True  # Make searchable
    )

    # Metadata fields (not embeddable)
    external_id: str = Field(
        ...,
        description="Unique ID from the external system"
    )

    permalink_url: Optional[str] = Field(
        None,
        description="Direct link to view in external system"
    )
```

### Key Principles

#### 1. Use AirweaveField for Searchable Content

**Important: The `embeddable=True` flag is what makes your entities semantically searchable.**

Without `embeddable=True`, fields are only keyword-searchable, not semantically searchable. This limits the user's ability to find relevant entities.

**Best Practice: Mark most user-visible, content-rich fields as `embeddable=True`**

This includes:
- **Text content**: descriptions, notes, comments, body text
- **Names and titles**: entity names, display names, titles
- **People**: assignees, authors, owners, members (as dicts with name/email)
- **Status and metadata**: status fields, tags, labels, priorities
- **Structured data**: any dict/list that contains searchable information
- **Timestamps**: created_at, modified_at, due_dates (helps with recency)
- **URLs**: permalink_url, web_links (helps users find original content)

**Only exclude from embeddable:**
- Internal IDs (entity_id, external_id, database IDs)
- Binary/technical metadata (sizes, checksums, mime_types)
- System-only fields not relevant to user searches

**Example - Information-Rich Entity:**

```python
class MyConnectorTaskEntity(ChunkEntity):
    """Task entity - NOTE: Most fields are embeddable for rich search."""

    # Core content - ALWAYS embeddable
    name: str = AirweaveField(..., description="Task name", embeddable=True)
    description: Optional[str] = AirweaveField(
        None,
        description="Task description",
        embeddable=True  # ✅ Critical for semantic search
    )
    notes: Optional[str] = AirweaveField(
        None,
        description="Additional notes",
        embeddable=True  # ✅ Searchable content
    )

    # People - embeddable for "find tasks assigned to John" queries
    assignee: Optional[Dict] = AirweaveField(
        None,
        description="User assigned to this task",
        embeddable=True  # ✅ Enables "who" searches
    )

    owner: Optional[Dict] = AirweaveField(
        None,
        description="Task owner",
        embeddable=True  # ✅ Enables owner searches
    )

    # Status and metadata - embeddable for filtering/search
    status: Optional[str] = AirweaveField(
        None,
        description="Task status (open, in_progress, done)",
        embeddable=True  # ✅ Enables status-based search
    )

    priority: Optional[str] = AirweaveField(
        None,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
