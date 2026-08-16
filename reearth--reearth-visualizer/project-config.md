---
trigger: always_on
description: This file governs how AI (Claude) should read, write, and reason about the E2E test suite under `e2e/`. Read it in full before making any changes to tests, pages, or utilities.
---

# E2E Test Suite — AI Management Rules

This file governs how AI (Claude) should read, write, and reason about the E2E test suite under `e2e/`. Read it in full before making any changes to tests, pages, or utilities.

---

## 1. Project Overview

This is a **Playwright E2E test suite** for Re:Earth Visualizer. Tests run against a live staging environment with real auth and real GraphQL API calls.

**Key facts:**
- Runtime: **WebKit headless** (`Desktop Safari`) — the only browser in CI
- Execution: **serial mode**, 2 workers, retries: 2 (CI) / 1 (local)
- Timeouts: default 120s per test, action/navigation 35s, expect 35s
- Auth: Auth0 storage state pre-loaded via `global-setup.ts`
- Config: `playwright.config.ts` — **do not modify** (managed separately)
- `global-setup.ts` — **do not modify** (managed via separate PR)

---

## 2. Directory Structure

```
e2e/
├── tests/              # All spec files (*.spec.ts)
├── pages/              # Page Object Model classes
├── utils/              # Shared helpers (auth, cleanup, constants)
├── api/                # GraphQL API test suite (separate project)
├── global-setup.ts     # Auth setup — DO NOT EDIT
├── global-teardown.ts  # Post-run cleanup
└── playwright.config.ts
```

**Spec files:**

| File | Feature Area | Notes |
|------|-------------|-------|
| `dashboard.spec.ts` | Dashboard, Recycle Bin | Full lifecycle |
| `dashboardFeatures.spec.ts` | Project Features | Search, sort, view, rename, export |
| `projectSettings.spec.ts` | Project Settings | All tabs (General/README/License/Story/Public/Assets) |
| `projects.spec.ts` | Editor | Layer creation, map interaction |
| `accountWorkspaceSettings.spec.ts` | Account & Workspace | **Entire suite skipped** — EE Config |
| `members.spec.ts` | Members Management | Conditionally skipped — Feature Flag |
| `layerDeletionReorder.spec.ts` | Layer Management | **Entire suite skipped** — WebKit DnD |
| `multipleStyles.spec.ts` | Layer Styles | Fully automated |
| `externalLayers.spec.ts` | External Layers | Mixed — some CI-skipped |
| `photoOverlay.spec.ts` | Photo Overlay | Mixed — some CI-skipped |
| `page-refresh-on-mutation.spec.ts` | Editor Mutations | **Entire suite skipped** — Canvas API |

---

## 3. User Story Database

All stories are tracked in the internal project management tool (see team wiki for the link).

- **Story ID format:** `US-{AREA}-{seq}` — e.g. `US-DASH-001`, `US-EDIT-003`
- **Feature Areas:** DASH · RBIN · PFEAT · PSET · EDIT · LAYER · STYLE · EXTL · PHOTO · MEM · ACCT · AUTH · STORY · PUB · PLUG

### Key fields

| Field | Purpose |
|-------|---------|
| `Story ID` | Stable reference. Use this in AI prompts and Allure annotations. |
| `Automation Status` | `Automated` / `Partial` / `Manual Only` / `Skipped (CI)` / `TODO` |
| `Skip Reason` | Root cause when skipped: `WebKit DnD` / `EE Config` / `Feature Flag` / `Canvas API` / `Not Yet Implemented` |
| `Spec File` | e.g. `tests/dashboard.spec.ts` |
| `Test Names` | Exact `test("...")` strings, one per line — used to link CI failures back to stories |
| `Acceptance Criteria` | Given/When/Then bullets. Each bullet = one `expect()` assertion. |
| `Manual Test Protocol` | Step-by-step instructions for `Manual Only` or `Skipped (CI)` stories |

### Automation Status lifecycle

```
TODO  ──(tests written)──►  Automated
                             │
              (suite skipped)▼
                         Skipped (CI)
                             │
              (flag enabled) ▼
                          Automated
```

---

## 4. How to Use Stories Day-to-Day

### Connecting test code to stories

Every test should carry its Story ID as an Allure annotation so that CI failures link directly to the story:

```typescript
test("Verify dashboard is loaded", async ({ page }) => {
  test.info().annotations.push({ type: "story", description: "US-DASH-001" });
  // ...
});
```

When CI fails, the Allure report shows the Story ID. Go to Notion, search that ID → open the story → read Acceptance Criteria to understand exactly what broke.

**Example — CI failure to story in three steps:**

```
FAIL  tests/dashboard.spec.ts
  ✗ Verify dashboard is loaded          ← test name
    Expected sidebar-tab-projects-link visible
    Received: timeout 35000ms
```

1. Allure report shows annotation `US-DASH-001`
2. Open Notion → search `US-DASH-001`
3. Acceptance Criteria reads: *"Given logged in, When visiting Dashboard URL, Then Projects nav entry is visible"* — now you know the failure scope is auth/navigation, not maps or data

Or ask AI directly:
> "US-DASH-001 failed in CI with `sidebar-tab-projects-link timeout`. Help me debug."

AI will look up the story's Acceptance Criteria, Spec File, and recent context without needing further explanation.

### Writing a new test from a story

1. Find a story with `Automation Status = TODO`
2. Read its `Acceptance Criteria` — each bullet becomes one `expect()` call
3. Write the test; add the Story ID annotation
4. Update the story: set `Automation Status → Automated`, fill in `Spec File` and `Test Names`

### Adding a story for a new feature (manual trigger)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reearth/reearth-visualizer](https://github.com/reearth/reearth-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
