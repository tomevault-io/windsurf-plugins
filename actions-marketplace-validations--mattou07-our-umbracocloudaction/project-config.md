---
trigger: always_on
description: handles a specific deployment operation: `start-deployment`, `check-status`,
---

# Copilot Instructions for Umbraco Cloud Deployment Action

## Project Overview

This is a GitHub Action (Node.js/TypeScript) for orchestrating deployments to
Umbraco Cloud. It provides a unified CI/CD entry point for deploying code and
content changes across Umbraco Cloud environments (Development → Staging →
Production).

**What it does**: Automates the Umbraco Cloud deployment workflow by handling
artifact management, deployment orchestration, status monitoring, and content
synchronization through the Umbraco Cloud REST API.

**Key Pattern**: Command-dispatcher architecture - `main.ts` routes to action
handlers in `src/actions/` based on the `action` input parameter. Each action
handles a specific deployment operation: `start-deployment`, `check-status`,
`add-artifact`, `get-changes`, `apply-patch`.

**Umbraco Context**: Umbraco Cloud is a managed hosting platform where
deployments flow from Development → Staging → Production environments. Code
changes (`.cloudsource` artifacts) are packaged, uploaded, and deployed via the
API. The action automates CI/CD by eliminating manual Cloud Portal steps.

## Architecture

### Core Components

1. **API Layer** (`src/api/umbraco-cloud-api.ts`)

   - Single class `UmbracoCloudAPI` wrapping Umbraco Cloud REST endpoints (v2
     API)
   - Handles authentication via `Umbraco-Cloud-Api-Key` header
   - **Resilience features**:
     - Rate limiting: Exponential backoff with max 3 retries for 429 responses
       (extracts retry-after from headers)
     - Environment alias case-sensitivity: Automatic fallback to lowercase if
       API rejects uppercase
     - Network errors: Wrapped with operation context for debugging
   - **Core methods**:
     - `startDeployment(request: DeploymentRequest): Promise<string>` -
       Initiates deployment, returns deploymentId
     - `checkDeploymentStatus(deploymentId: string): Promise<DeploymentResponse>` -
       Fetches deployment state
     - `uploadArtifact(filePath: string, ...): Promise<ArtifactResponse>` -
       Uploads `.cloudsource` ZIP files
     - `getChangesById(deploymentId: string, targetAlias: string): Promise<ChangesResponse>` -
       Retrieves diff/patch
     - `applyPatch(changeId: string, targetAlias: string): Promise<void>` -
       Applies content changes

2. **Action Handlers** (`src/actions/`)

   - **`start-deployment.ts`**:
     - Validates `artifactId` and `targetEnvironmentAlias` inputs
     - Calls `api.startDeployment()` with deployment config (build/restore
       flags, skip version check)
     - Invokes `pollDeploymentStatus()` to await completion (blocking operation)
     - Sets GitHub output: `deploymentId`
   - **`check-status.ts`**:
     - Monitors existing deployment via `deploymentId`
     - Polls until `Completed` or `Failed` state
     - On success: Retrieves changes and optionally creates PR with diff
     - Handles "updating marker" errors (site blocked during Umbraco version
       upgrades)
     - Outputs: `deploymentState`, `deploymentStatus`, optional `pullRequestUrl`
   - **`add-artifact.ts`**:
     - Packages `.cloudsource` ZIP from `filePath` input
     - Removes excluded paths (`.git/`, `.github/` by default) to reduce upload
       size
     - Optionally modifies NuGet.config if custom package sources provided
     - Uploads with retry logic (configurable retries, timeouts)
     - Outputs: `artifactId`, `fileName`, `blobUrl`
   - **`get-changes.ts`**:
     - Retrieves diff between deployment and target environment
     - Returns patch/changeset as JSON output
   - **`apply-patch.ts`**:
     - Applies content changes identified by `changeId`
     - Used for non-code content syncs
   - Each handler validates inputs via `validateRequiredInputs()` and returns
     `ActionOutputs`

3. **Input/Output Management** (`src/main.ts`)

   - `getActionInputs()`: Parses GitHub Action inputs (via `@actions/core`) into
     `ActionInputs` type
   - Type-safe conversion:
     - String inputs: `core.getInput('name')`
     - Boolean inputs: `core.getBooleanInput('name')` (parses "true"/"false")
     - Integer inputs: `parseInt(core.getInput('name'), 10)` for timeouts,
       retries
   - Default fallbacks: Base URL, timeouts, excluded paths defined here
   - Input relay flow: `action.yml` defines available inputs →
     `getActionInputs()` parses → `main.ts` routes to handlers

4. **Polling & Status Management** (`src/utils/deployment-polling.ts`)

   - `pollDeploymentStatus()` continuous loop with configurable interval
     (default 25s) and max duration (default 15min)
   - Polls `/v2/projects/{projectId}/deployments/{deploymentId}` endpoint
   - Handles transient errors: 401 (auth), 404 (not found), network timeouts
     with retries
   - Emits real-time status messages from Umbraco: deployment state, phase
     names, progress
   - Detects blocking errors: "updating marker" (site locked during version
     upgrades)
   - Supports conditional requests via `lastModifiedUtc` query param to optimize
     API calls

5. **Type System** (`src/types/index.ts`)
   - `ActionInputs`: All possible action parameters (32+ fields)
     - Deployment: `artifactId`, `targetEnvironmentAlias`, `commitMessage`,
       `skipVersionCheck`
     - Polling: `deploymentId`, `timeoutSeconds`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [actions-marketplace-validations/mattou07_Our.UmbracoCloudAction](https://github.com/actions-marketplace-validations/mattou07_Our.UmbracoCloudAction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
