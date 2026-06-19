---
trigger: always_on
description: Lastest is a visual regression testing platform. It records browser interactions, runs Playwright tests, diffs screenshots against baselines, and uses AI to classify changes.
---

# AGENTS.md — Lastest

## What is Lastest?

Lastest is a visual regression testing platform. It records browser interactions, runs Playwright tests, diffs screenshots against baselines, and uses AI to classify changes.

## MCP Server

Install the MCP server to let AI agents interact with Lastest:

```bash
npx @lastest/mcp-server --url http://localhost:3000 --api-key YOUR_API_KEY
```

### Available Tools (50 total, all prefixed `lastest_`)

| Category                         | Tools                                                                                                                              |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Health & jobs                    | `health_check`, `list_active_jobs`, `get_job_status`                                                                               |
| Repositories                     | `list_repos`, `get_repo`, `create_repo`, `update_repo`                                                                             |
| Playwright settings (repo-level) | `get_playwright_settings`, `update_playwright_settings`                                                                            |
| Functional areas                 | `list_areas`, `create_area`, `update_area`, `delete_area`, `list_tests_by_area`                                                    |
| Tests                            | `list_tests`, `list_failing_tests`, `get_test`, `create_test`, `update_test`, `delete_test`, `heal_test`                           |
| Setup scripts                    | `list_setup_scripts`, `get_setup_script`, `create_setup_script`, `update_setup_script`, `delete_setup_script`                      |
| Storage states                   | `list_storage_states`, `create_storage_state`, `delete_storage_state`                                                              |
| Runs & builds                    | `run_tests` (accepts `forceVideoRecording`, `functionalAreaId`), `get_test_run`, `list_builds`, `get_build_status`, `review_build` |
| Diffs & baselines                | `get_diff`, `get_visual_diff`, `approve_diff`, `reject_diff`, `approve_all_diffs`, `approve_baseline`, `reject_baseline`           |
| Verify phase                     | `get_change_map`, `verify_build`, `approve_layer`                                                                                  |
| Sharing                          | `publish_share`, `list_build_shares`, `list_test_shares`, `revoke_share`                                                           |
| Coverage & QA                    | `get_coverage`, `qa_summary`                                                                                                       |

`lastest_update_test` self-configures a test end-to-end: name/code/URL, functional area, lifecycle (`quarantined`, `executionMode`), setup wiring (`setupTestId` | `setupScriptId`, `setupOverrides`, `teardownOverrides`), and runtime overrides (`playwrightOverrides`, `diffOverrides`, `stabilizationOverrides`, `viewportOverride`). Pass `null` to any override block to clear it.

### Typical Workflow

1. `lastest_run_tests` — start a build (`forceVideoRecording: true` if you need video for a share)
2. `lastest_build` `action:"get"` — poll until complete (`action:"review"` for failures + action items)
3. If visual changes: `lastest_get_diffs` `scope:"build"` — inspect diffs
4. `lastest_decide_diff` `action:"approve"|"reject"` — act on diffs/baselines
5. If failures: `lastest_heal_test` — auto-fix the test, then re-run; or `lastest_suggest_app_fix` for an app-code fix suggestion
6. To share: `lastest_publish_share` → public `/r/<slug>` URL. Manage with `lastest_share` `action:"list"|"revoke"`.

For a fast inner-loop check after a code change, use `lastest_validate_diff` (maps a diff to the affected tests, runs only those, returns one verdict).

### Build Status Values

- `safe_to_merge` — all tests passed, no pending diffs
- `review_required` — visual changes detected, awaiting review
- `blocked` — tests failed or diffs rejected
- `has_todos` — diffs marked as todo for later review

### Response Format

Every tool returns:

```json
{
  "status": "machine_readable_status",
  "summary": "Human-readable 1-2 sentence summary",
  "actionRequired": ["Next steps for the agent"],
  "details": {}
}
```

## REST API

Base URL: `http://localhost:3000/api/v1/`
Auth: `Authorization: Bearer <api-key>` (or browser cookie session)

The MCP server is a thin wrapper around these endpoints — anything an agent can do, a script can do.

### Read

| Method + Path                        | Purpose                                                                                   |
| ------------------------------------ | ----------------------------------------------------------------------------------------- |
| `GET /health`                        | Health check                                                                              |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [las-team/lastest](https://github.com/las-team/lastest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
