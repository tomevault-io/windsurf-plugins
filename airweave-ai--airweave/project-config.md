---
trigger: always_on
description: This is the **master guide** for building a complete, production-ready source connector for Airweave. It combines source implementation with comprehensive E2E testing using the Monke framework.
---

# Building and Testing a Source Connector: End-to-End Guide

## Overview

This is the **master guide** for building a complete, production-ready source connector for Airweave. It combines source implementation with comprehensive E2E testing using the Monke framework.

**Use this guide with your AI coding assistant** to build connectors systematically.

---

## Prerequisites

**Note:** The human has already completed these setup steps:
- ✅ OAuth credentials configured in `backend/airweave/platform/auth/yaml/dev.integrations.yaml`
- ✅ Monke authentication configured in `monke/configs/{short_name}.yaml` (Composio or direct)
- ✅ API documentation loaded into context

Your task is to write the code. The human will handle testing and running commands.

---

## Important Guidelines

These are the most common mistakes when building connectors:

### 1. Make Entities Information-Rich (Embeddable Fields)

**Rule:** Mark ~70% of entity fields as `embeddable=True`

**Why:** Without `embeddable=True`, fields are only keyword-searchable, not semantically searchable. Users won't be able to find relevant data.

**What to mark embeddable:**
- ✅ All text content (descriptions, notes, comments, body)
- ✅ All names and titles
- ✅ All people (assignees, authors, owners, members)
- ✅ All status/metadata (status, priority, tags, labels)
- ✅ All timestamps (created_at, modified_at, due_dates)

**What NOT to mark embeddable:**
- ❌ Internal IDs (entity_id, external_id, database IDs)
- ❌ Binary metadata (sizes, checksums, mime_types)

**Bad Example:**
```python
# Avoid: Sparse entity - users can't search by anything except name
class TaskEntity(ChunkEntity):
    name: str = AirweaveField(..., embeddable=True)
    description: str = Field(...)  # Should be embeddable
    assignee: Dict = Field(...)     # Should be embeddable
```

**Good Example:**
```python
# Better: Information-rich - users can search everything
class TaskEntity(ChunkEntity):
    name: str = AirweaveField(..., embeddable=True)
    description: str = AirweaveField(..., embeddable=True)
    assignee: Dict = AirweaveField(..., embeddable=True)
    status: str = AirweaveField(..., embeddable=True)
    external_id: str = Field(...)  # ID correctly not embeddable
```

### 2. Test Entity Types Your Source Actually Implements

**Rule:** Your Monke tests should create and verify the entity types that your source actually yields

**Why:** Untested entity types may break in production without detection.

**Important:** Only test entities that your source implementation yields. You don't need to test every theoretically possible entity type from the API—just the ones your connector actually implements.

**How to verify:**
1. Open your source: `backend/airweave/platform/sources/{short_name}.py`
2. Find all `yield` statements in `generate_entities()`
3. List the entity types your source ACTUALLY yields (e.g., Task, Comment, File)
4. Your `bongos/{short_name}.py::create_entities()` should create at least one of each yielded type
5. Your `create_entities()` should return descriptors for all yielded types

**Example:** If your SharePoint source only yields `ListItem`, `Page`, and `DriveItem` entities (not `User`, `Group`, `Site`), then your Monke bongo only needs to create those three types—not the entire SharePoint API surface.

**Bad Example:**
```python
# Avoid: Only creates tasks, ignores comments and files
async def create_entities(self):
    for i in range(self.entity_count):
        task = await self._create_task(...)
        all_entities.append(task)
    # Source yields comments and files, but we don't test them
    return all_entities
```

**Good Example:**
```python
# Better: Creates all entity types from source
async def create_entities(self):
    for i in range(self.entity_count):
        # Create parent
        task = await self._create_task(...)
        all_entities.append(task)

        # Create comments (source yields them)
        for j in range(2):
            comment = await self._create_comment(task["id"], ...)
            all_entities.append(comment)

        # Create file (source yields them)
        file = await self._upload_file(task["id"], ...)
        all_entities.append(file)

    return all_entities  # Returns tasks, comments, AND files
```

---

---

## Phase 1: Research & Planning

### Step 1: Understand the API

**Questions to answer:**

1. What is the **entity hierarchy**?
   - Example: Asana has `Workspace → Project → Section → Task → Comment/File`
   - Example: GitHub has `Repository → Issue → Comment`  and `Repository → Folder1 → Folder2 → codefile.go`

2. What **entities should be searchable**?
   - Primary entities (tasks, documents, tickets)
   - Secondary entities (comments, messages, threads)
   - Attachments (files, images, PDFs)

3. What **authentication** does it use?
   - OAuth2 with refresh tokens?
   - OAuth2 without refresh tokens?
   - API key / Personal Access Token?

4. Does it support **incremental sync**?
   - Can you filter by `modified_since` or `updated_at`?
   - Does each entity have `created_at` and `modified_at` timestamps?

5. Does it support **deletion detection**?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
