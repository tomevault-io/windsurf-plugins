---
trigger: always_on
description: How to add a new storage or data connector for Label Studio
---

# Implementing New Storage Providers in Label Studio

## Overview
This document describes the process and best practices for adding a new storage provider to Label Studio using the declarative provider schema system.

Label Studio supports 2 types of cloud storages:
1. **Import Storages** (Source Cloud Storages) - for importing tasks/data
2. **Export Storages** (Target Cloud Storages) - for exporting annotations

See comprehensive overview about storages @io_storages/README.md. 


## Implementation Checklist

Follow all steps below to implement a new storage. More details follow after the checklist; review them all. Do it on your best, until all items are done and tests are passing. 

### 1. Exploration and preparation
1. [ ] Carefully read @io_storages/README.md
2. [ ] Search official documentation for the new storage you want to add
  - [ ] Determine whether pre-signed URLs are supported, or whether only direct reads are possible. In case of direct reads, we should hide pre-signed URLs toggle and use Label Studio proxy. 
  - [ ] Determine whether writes are supported, and how annotations will be stored (objects/blobs, files, rows/strings in a table, etc.)
  - [ ] Understand the provider's Python API/SDK, especially how to read, write, and list objects. If SDK is available, use SDK
3. If the requester hasn't specified the target edition, recommend Open Source or Enterprise and confirm the choice
4. Check storage directory structure in `label_studio/io_storages` (or `label_studio_enterprise/lse_io_storages` for Enterprise) and the `s3` (or `s3s` for Enterprise) subfolder
5. [ ] Create the new provider directory structure based on the pattern you observed
6. [ ] Create a README.md file in the new provider folder
7. [ ] Add a brief Overview section about the new storage and your findings from step 2
8. [ ] Add a section on how to configure the storage from scratch for users unfamiliar with it. Provide clear, correct, up-to-date steps with links to official docs to reduce manual QA time

### 2. Backend Implementation
1. [ ] Implement storage mixin with common fields:
  - [ ] Basic fields: bucket, prefix, regex_filter, use_blob_urls (pre-signed URLs on/off), recursive_scan (if applicable)
  - [ ] URL resolution: presign, presign_ttl (if applicable to the storage)
  - [ ] Provider credentials: api_key, secret_key, endpoint_url
  - [ ] Common methods: get_client(), validate_connection()
2. [ ] Create import storage base class with required methods:
  - [ ] `iter_objects()` - iterate over storage objects
  - [ ] `get_data()` - load task data from objects
  - [ ] `generate_http_url()` - create HTTP URLs
  - [ ] `can_resolve_url()` - check URL resolution capability
  - [ ] `validate_connection()` - validate credentials and that the prefix contains files
3. [ ] Create export storage class with required methods:
  - [ ] `save_annotation()` - save single annotation to storage
  - [ ] `delete_annotation()` - delete annotation from storage (optional)
  - [ ] `validate_connection()` - validate credentials and bucket access (NO prefix check)
4. [ ] Create non-abstract provider-specific concrete classes for import and export
5. [ ] Implement storage link models:
  - [ ] ImportStorageLink for tracking task imports
  - [ ] ExportStorageLink for tracking annotation exports
6. [ ] **CRITICAL: Add `app_label = 'io_storages'` to Meta classes** - All concrete storage models (ImportStorage, ExportStorage, and StorageLink classes) must include `app_label = 'io_storages'` in their Meta class to avoid Django app registration errors. This is required because storage providers are in subdirectories of `io_storages` but need to be registered under the main `io_storages` app. **Note**: Enterprise providers do NOT need app_label - see enterprise guide.
7. [ ] Create serializers with validation logic
8. [ ] Implement API views following existing patterns
9. [ ] Register URLs in storage URL configuration
10. [ ] Add signal handlers for auto-export functionality:
  - [ ] post_save signal for automatic annotation export
  - [ ] pre_delete signal for automatic annotation deletion
  - [ ] Async export functions with error handling
11. [ ] If you use SDK: add provider SDK library to pyproject.toml
  - [ ] Make poetry lock: `poetry install && poetry lock`
12. [ ] Create database migrations using `poetry run python manage.py makemigrations` only!   
13. [ ] Ensure that you correctly handle token and security fields; they should not be displayed on the frontend or backend after they are initially entered and saved. Verify how this works with other storage codes.
14. [ ] **Error Handling Strategy:**  
   - **Import**: If `VolumesAPI.iter_files` or other listing helpers call `resp.raise_for_status()`, make sure callers wrap these in try/except (or downgrade to debug logging) so recursive listing failures can be surfaced without crashing the sync job.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HumanSignal/label-studio](https://github.com/HumanSignal/label-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
