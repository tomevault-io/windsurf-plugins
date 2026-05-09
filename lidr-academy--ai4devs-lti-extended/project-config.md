---
trigger: always_on
description: Enforce mandatory steps from openspec/config.yaml when creating tasks.md artifacts
---


# OpenSpec Tasks: Mandatory Steps Enforcement

When creating or updating `tasks.md` artifacts in OpenSpec changes, you MUST:

## 1. Read openspec/config.yaml First

**BEFORE** creating or updating any `tasks.md` file, you MUST read `openspec/config.yaml` to understand:
- Backend-specific mandatory steps
- Branch naming conventions
- Task structure requirements
- Testing and documentation requirements

## 2. Mandatory Steps for Backend Changes

All backend implementation tasks MUST include these steps in the correct order:

### Step 0: Create Feature Branch (MUST BE FIRST)
- **Location**: Must be the very first step (Step 0)
- **Branch naming**: `feature/[ticket-id]-backend` or `feature/[change-name]-backend`
- **Action**: Create and switch to feature branch before any code changes

### Mandatory Steps (Must Be Included):
- **Step N**: Review and Update Existing Unit Tests (MANDATORY)
- **Step N+1**: Run Unit Tests and Verify Database State (MANDATORY)
- **Step N+2**: Manual Endpoint Testing with curl (MANDATORY) - **AGENT MUST EXECUTE**
- **Step N+3**: E2E Testing with Playwright MCP (MANDATORY if applicable) - **AGENT MUST EXECUTE**
- **Step N+4**: Update Technical Documentation (MANDATORY)

## 3. Manual Testing Requirements - CRITICAL: Agent Must Execute

**IMPORTANT**: The coding agent (AI) MUST perform all manual testing steps itself. **NEVER delegate testing to the user**. These tests must be executed by the agent to mark tasks as completed in `tasks.md`.

### Step N+2: Manual Endpoint Testing with curl (MANDATORY)

**Agent Responsibility**: The coding agent MUST execute all curl commands and verify responses. This is NOT optional and cannot be delegated to the user.

**Implementation Steps** (Agent must perform):
1. **Prepare Test Environment**:
   - Ensure the backend server is running (start if needed)
   - Verify database connection is active
   - Note the current database state (if testing CREATE/UPDATE/DELETE endpoints)

2. **Test GET Endpoints** (if any):
   - Create curl command to test GET endpoint
   - Execute curl command: `curl -X GET [endpoint-url] [headers]`
   - Verify response status code (200, 404, etc.)
   - Verify response body structure and content
   - Document the curl command and response in the task completion

3. **Test POST Endpoints** (CREATE operations):
   - Create curl command with request body: `curl -X POST [endpoint-url] -H "Content-Type: application/json" -d '[json-body]'`
   - Execute curl command and capture response
   - Verify response status code (201, 400, 422, etc.)
   - Verify response body contains created resource
   - **Restore Database State**: After testing, delete the created record to restore database to original state
   - Document the curl command, response, and cleanup action

4. **Test PUT/PATCH Endpoints** (UPDATE operations):
   - Create curl command with updated data: `curl -X PUT [endpoint-url] -H "Content-Type: application/json" -d '[json-body]'`
   - Execute curl command and capture response
   - Verify response status code (200, 404, 400, etc.)
   - Verify response body contains updated resource
   - **Restore Database State**: After testing, revert the updated record to its original values to restore database state
   - Document the curl command, response, and cleanup action

5. **Test DELETE Endpoints**:
   - Create curl command: `curl -X DELETE [endpoint-url]`
   - Execute curl command and capture response
   - Verify response status code (200, 204, 404, etc.)
   - Verify deletion was successful
   - **Restore Database State**: After testing, recreate the deleted record with original values to restore database state
   - Document the curl command, response, and cleanup action

6. **Test Error Cases**:
   - Test with invalid data (validation errors)
   - Test with non-existent resources (404 errors)
   - Test with unauthorized access (if applicable)
   - Verify error response format matches API specification

7. **Mark Task as Completed**: Only after all curl tests pass and database state is restored, mark the task as completed in `tasks.md`

**Dependencies**:
- Backend server running (agent must start if needed)
- Database access for state restoration
- curl command-line tool

**Notes**:
- This step is MANDATORY for all new endpoints
- **The agent MUST execute all curl commands itself** - never ask the user to run tests
- All CREATE/UPDATE/DELETE operations must restore database to original state after testing
- Document all curl commands and responses for future reference
- Verify that database state matches pre-test state after cleanup
- Do not skip manual testing even if unit tests pass
- **Task completion in tasks.md can only be marked after successful execution of all curl tests**

### Step N+3: E2E Testing with Playwright MCP (MANDATORY if applicable)

**Agent Responsibility**: The coding agent MUST execute all E2E tests using Playwright MCP tools. This is NOT optional and cannot be delegated to the user.

**When This Applies**:
- Frontend changes that affect user workflows
- Integration between frontend and backend endpoints
- User-facing features that require browser interaction

**Implementation Steps** (Agent must perform):
1. **Prepare Test Environment**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LIDR-academy/AI4Devs-LTI-extended](https://github.com/LIDR-academy/AI4Devs-LTI-extended) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
