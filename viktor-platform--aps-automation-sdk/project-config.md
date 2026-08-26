---
trigger: always_on
description: Use when:
---

# AGENTS.md

> Canonical instructions file for this repository.
> `CLAUDE.md` imports this file so Codex and Claude share one instruction source.

## Repository Assessment

### What this repo is
`aps-automation-sdk` is a Python 3.11+ SDK for Autodesk Platform Services (APS) focused on Design Automation (AutoCAD/Revit), object storage (OSS), ACC Data Management integration, and optional Model Derivative translation for viewing.

Repository clarification used in this guide: Design Automation base URL is treated as US-only (`https://developer.api.autodesk.com/da/us-east/v3`).

If you want to create or modify a method of this repository, always check the autodesk documentation. Refer to "Method-to-APS Documentation Matrix" section of this document.
---

## Repository Sections and Functionalities

## 1) Package Entry (`aps_automation_sdk/__init__.py`)
Purpose: Re-exports the public API for easy imports.

Relevant exports:
- Workflow classes: `Activity`, `AppBundle`, `WorkItem`, `WorkItemAcc`, parameter classes.
- Auth/admin helpers: `get_token`, `set_nickname`, `create_bucket`, `publish_appbundle_update`, etc.
- Model Derivative helpers: `translate_file_in_oss`, `get_translation_status`, etc.

---

## 2) HTTP Core for Design Automation + OSS (`aps_automation_sdk/core.py`)
Purpose: Low-level wrappers around APS REST endpoints.

Functional groups and key methods:
- Region/URL helpers:
  - `get_da_base_url(region="US")`
- OSS signed upload/download:
  - `get_signed_s3_upload(...)`
  - `put_to_signed_url(signed_url, file_path)`
  - `complete_signed_s3_upload(...)`
  - `get_signed_s3_download(...)`
  - `dowload_from_signed_url(...)`
  - `build_oss_urn(bucketKey, objectKey)`
- AppBundle/activity management:
  - `register_appbundle(...)`
  - `upload_appbundle(upload_parameters, zip_path)`
  - `create_appbundle_alias(...)`
  - `create_activity(payload=...)`
  - `create_activity_alias(...)`
- Work item lifecycle:
  - `run_work_item(...)`
  - `run_public_work_item(...)`
  - `get_workitem_status(...)`
  - `poll_workitem_status(...)`
  - `fetch_report_content(report_url)`

Use when:
- You want direct API control without higher-level classes.

---

## 3) Auth and Administrative Helpers (`aps_automation_sdk/utils.py`)
Purpose: OAuth token retrieval + common setup/maintenance operations.

Relevant methods:
- OAuth/token:
  - `get_token(client_id, client_secret)`
- Forge app identity:
  - `set_nickname(token, nickname)`
  - `get_nickname(token)`
- Bucket and cleanup:
  - `create_bucket(bucketKey, token, ...)`
  - `delete_appbundle(appbundleId, token)`
  - `delete_activity(activityId, token)`
- AppBundle updates without downtime:
  - `create_appbundle_version(...)`
  - `move_or_create_alias(...)`
  - `publish_appbundle_update(...)`

Use when:
- You need fast bootstrap/auth and lifecycle utilities.

---

## 4) Data Models (`aps_automation_sdk/dsl.py`)
Purpose: Pydantic models for response/request payload structure.

Relevant models:
- Upload form and appbundle responses:
  - `FormData`, `UploadParameters`, `RegisterBundleResponse`
- Signed URL payloads:
  - `GetSignedS3UrlsResponse`, `CompleteUploadRequest`, `GetDownloadS3Url`

Use when:
- You need typed validation for payloads returned by APS.

---

## 5) Object-Oriented Workflow Layer (`aps_automation_sdk/classes.py`)
Purpose: High-level abstractions for bundles, activities, parameters, and work items.

### Parameter classes
- `ActivityParameter`
  - Storage helpers: `oss_keys()`, `upload_file_to_oss(...)`, `download_to(...)`, `generate_oss_urn()`
- `ActivityInputParameter`
  - `work_item_arg(token)`
- `ActivityOutputParameter`
  - `work_item_arg(token)`
- `ActivityJsonParameter`
  - `set_content(data)`, `work_item_arg()`

### Deployment classes
- `AppBundle`
  - `register()`, `upload()`, `create_alias()`, `deploy()`
- `Activity`
  - `set_revit_command_line()`
  - `set_autocad_command_line()`
  - `to_api_dict()`
  - `deploy()`

### Execution classes
- `WorkItem`
  - `build_arguments()`, `run()`, `poll()`, `execute()`

Use when:
- You want a clean end-to-end flow with minimal direct REST payload handling.

---

## 6) ACC Data Management Integration (`aps_automation_sdk/acc.py` + ACC classes)
Purpose: Resolve ACC storage IDs, create storage, and create/update ACC versions/items.

Core ACC functions (`acc.py`):
- Resolve folder/item relationships:
  - `item_from_version(...)`, `parent_folder_from_item(...)`, `resolve_parent_folder(...)`
- Tip/storage retrieval:
  - `get_item_tip_version(...)`, `find_tip_storage_id(...)`
- Write pipeline:
  - `create_storage(...)`
  - `find_item_by_name(...)`
  - `create_version_for_item(...)`
  - `create_item_with_first_version(...)`

ACC-specific parameter/work item classes (`classes.py`):
- Inputs from ACC:
  - `ActivityInputParameterAcc` (`work_item_arg_3lo(...)`)
  - `UploadActivityInputParameter` (`upload_and_create(...)`, `work_item_arg_3lo(...)`)
- Outputs to ACC:
  - `ActivityOutputParameterAcc` (`work_item_arg_3lo(...)`, `create_acc_item(...)`, `get_lineage_urn()`)
- Orchestration:
  - `WorkItemAcc.build_arguments_3lo(...)`
  - `WorkItemAcc.execute_and_finalize(...)`

Use when:
- Your inputs/outputs are in Autodesk Construction Cloud and require 3LO token handling.

---

## 7) Model Derivative Helpers (`aps_automation_sdk/model_derivative.py`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [viktor-platform/aps-automation-sdk](https://github.com/viktor-platform/aps-automation-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
