---
trigger: always_on
description: This repository is the PHP monorepo for the community-maintained Azure SDKs under the `azure-oss` namespace. SDK packages are organized by service inside `src/`, with Storage packages under `src/Storage`, Identity under `src/Identity`, and the documentation website under `docs/`.
---

# AGENTS.md

## Overview

This repository is the PHP monorepo for the community-maintained Azure SDKs under the `azure-oss` namespace. SDK packages are organized by service inside `src/`, with Storage packages under `src/Storage`, Identity under `src/Identity`, and the documentation website under `docs/`.

At the root you will find:

- `composer.json`: the monorepo-level package, shared dependencies, and root autoload setup.
- `src/`: all SDK package source code, grouped by Azure service.
- `tests/`: the test suites, grouped by package.
- `docs/`: the Docusaurus documentation website, split to `php-oss-for-azure/php-oss-for-azure.github.io`.
- `infra/`: Azure Bicep templates for provisioning storage resources used during development or validation.
- `.github/`: repository automation and helper scripts such as subtree/package sync tooling.

## Source Layout

### `src/Storage/BlobFlysystemBundle`

Symfony bridge for the Azure Blob Storage Flysystem adapter.

### `src/Storage/File/Share`

Azure Storage File Share SDK (Under construction).

### `src/Storage/Common`

Shared primitives used across the storage packages.

- `Auth/`: authentication helpers such as shared key credentials.
- `Middleware/`: Guzzle middleware, client factory logic, retry/auth header wiring, and HTTP options.
- `Sas/`: account-level SAS builders, permissions, protocols, and related value objects.
- `Exceptions/` and `Helpers/`: reusable helpers and cross-package error handling.
- `aliases.php`: backwards-compatibility aliases for moved or renamed public types.

This is the lowest-level package in the repo. Blob and queue code depend on it.

### `src/Storage/Blob`

The core Azure Blob Storage SDK, published as `azure-oss/storage-blob`.

- Top-level clients:
  - `BlobServiceClient.php`
  - `BlobContainerClient.php`
  - `BlobClient.php`
- `Specialized/`: specialized blob clients such as block blob support.
- `Models/`: request option objects, result objects, and domain models.
- `Requests/` and `Responses/`: request/response payload mapping types.
- `Sas/`: blob- and container-specific SAS builders and permission types.
- `Exceptions/` and `Helpers/`: blob-specific parsing, metadata, streams, dates, and error translation.
- `aliases.php`: backwards-compatibility aliases for older blob package class names.

Use this package for changes to the Blob SDK itself.

### `src/Storage/Queue`

The Azure Storage Queue SDK.

- Top-level clients:
  - `QueueServiceClient.php`
  - `QueueClient.php`
- `Models/`: queue/message models and client option types.
- `Requests/` and `Responses/`: XML/body mapping classes for queue operations.
- `Exceptions/` and `Helpers/`: queue-specific exceptions and metadata helpers.

Use this package for queue CRUD, message send/receive/delete, and queue client behavior.

### `src/Storage/BlobFlysystem`

Flysystem integration for Azure Blob Storage.

- `AzureBlobStorageAdapter.php`: the main Flysystem adapter.
- `Support/`: adapter-specific config parsing and support utilities.
- `aliases.php`: backwards-compatibility aliases for older Flysystem integration class names.

This layer depends on the Blob SDK and should stay thin: most storage behavior belongs in `src/Storage/Blob`, not here.

### `src/Storage/BlobLaravel`

Laravel filesystem integration built on top of the Flysystem adapter.

- `AzureStorageBlobServiceProvider.php`: registers the Laravel driver.
- `AzureStorageBlobAdapter.php`: bridges Laravel filesystem expectations to the Flysystem adapter.
- `AzureStorageBlobDiskConfig.php`: configuration parsing and validation.

Changes here should focus on Laravel service registration, config handling, and framework integration.

### `src/Storage/QueueLaravel`

Laravel queue integration for Azure Storage Queues.

- `AzureStorageQueueServiceProvider.php`: registers the queue connector.
- `AzureStorageQueueConnector.php`: constructs queue connections from Laravel config.
- `AzureStorageQueue.php`: Laravel queue driver implementation.
- `AzureStorageQueueJob.php`: wraps received queue messages as Laravel jobs.
- `AzureStorageQueueConfig.php`: config parsing and normalization.

Changes here should stay Laravel-specific rather than duplicating queue SDK behavior.

### `src/Identity`

Azure Identity SDK, published as `azure-oss/identity` and split to `Azure-OSS/azure-identity-php`.

### `docs`

Docusaurus documentation website, split to `Azure-OSS/php-oss-for-azure.github.io`.

## Dependency Direction

The packages are layered roughly like this:

`Storage/Common` -> `Storage/Blob` / `Storage/Queue` / `Storage/File/Share` -> `Storage/BlobFlysystem` -> `Storage/BlobLaravel` / `Storage/BlobFlysystemBundle`

`Identity` -> `Storage/Common`

`Storage/Common` -> `Storage/Queue` -> `Storage/QueueLaravel`

In practice:

- Put reusable Storage HTTP/auth/SAS utilities in `src/Storage/Common`.
- Put Azure Blob API behavior in `src/Storage/Blob`.
- Put Azure Queue API behavior in `src/Storage/Queue`.
- Put Azure File Share API behavior in `src/Storage/File/Share`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [php-oss-for-azure/azure-php](https://github.com/php-oss-for-azure/azure-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
