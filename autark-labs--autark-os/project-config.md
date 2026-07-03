---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

Project OS should feel like a calm, powerful appliance: install an app, open it, access it safely, and recover it when something goes wrong. Code should protect that product promise. Favor readable implementation, clear user states, and small validated slices over broad rewrites.

---

## 1. Core Product Principles

### Treat Project OS as a guided runtime, not a generic dashboard

The app should help users answer:

- What is installed?
- What is ready to use?
- What needs attention?
- What should I do next?

Avoid surfacing raw infrastructure details unless the user is in an advanced or diagnostics flow.

### Do not blur ownership states

Project OS must distinguish between:

- **Managed apps**: owned by the current Project OS instance.
- **Found apps**: detected on the host but not owned by this instance.
- **Linked services**: user-added shortcuts or externally managed services.
- **Conflicts**: resources that block install or recovery.

Never show a foreign or legacy app as simply “Installed” for the current instance. Use language such as “Found on this server,” “Owned by another Project OS instance,” “Recoverable,” or “Linked service.”

### One clear next action

Each page should have one obvious primary action. Do not add multiple competing banners, alerts, or hero actions. When in doubt, use the canonical recommended-action/state model instead of inventing page-specific health logic.

---

## 2. Development Workflow

### Work in lean slices

Each change should be small enough to validate both behavior and look/feel before moving on.

A good slice includes:

1. Backend or state-model change, if needed.
2. Frontend rendering of that state.
3. Loading, empty, success, and error states.
4. A visible way to test the feature manually.
5. Tests or smoke coverage for the important path.

Avoid landing invisible infrastructure unless it directly unlocks the current slice.

### No partial state-model implementations

When a request is to consolidate, replace, or make a single source of truth for a product concept, the work is not done until every active backend and frontend consumer uses that source or the user has explicitly approved a temporary exception.

Required behavior:

- Identify every endpoint, service, page, component, and client that reads or derives the affected state before editing.
- Add regression coverage for the exact cross-page or cross-endpoint mismatch being fixed.
- Migrate all active consumers in the same corrective pass.
- Do not leave parallel interpretation paths that can produce different user-facing answers.
- Do not call an implementation complete because the primary page works while secondary pages still compose state independently.
- If full migration is too large or risky, stop and ask before landing a partial implementation.

For application state specifically, Home, My Apps, Discover, Access, Backups, Storage, Monitoring, Settings, Support, and onboarding/setup must not independently decide what is installed, pinned, found, recoverable, or managed. Those states must come from the canonical application-state model.

### No partial cross-surface product behavior

When implementing a user-facing behavior pattern, update every active surface that exposes that behavior in the same pass. Examples include disabled-action explanations, install job progress, ownership labels, action notifications, app status badges, and recovery/cleanup safety copy.

Required behavior:

- Identify the active pages, dialogs, drawers, popovers, cards, and shared components that expose the behavior before editing.
- Update every active surface or stop and ask for an explicit lean-slice exception before coding.
- A lean-slice exception must name the excluded surfaces, explain why they are excluded, and leave a concrete follow-up story or checklist.
- Do not report a behavior as implemented if only the primary page, happy path, or newest component reflects it.
- Prefer shared components and contract tests for cross-surface behaviors so future changes cannot quietly drift back into partial implementations.

### Prefer additive refactors over broad rewrites

Refactor behind stable APIs/components when possible. Do not churn unrelated files. Preserve working behavior while simplifying the implementation.

### No dead controls

Every visible button, link, card click, dropdown item, and menu action must do one of the following:

- Perform the action.
- Navigate to the right screen.
- Open a modal/drawer/popover.
- Be hidden until it is implemented.
- Be explicitly disabled with a reason.

Never leave placeholder actions in production UI.

---

## 3. Code Readability Standards

### Optimize for the next human reader

Prefer clear names over clever abstractions.

Good:

```ts
const isForeignProjectOsApp = app.ownershipState === "foreign_project_os";
const canRecoverApp = app.availableActions.includes("recover");
```

Avoid:

```ts
const flag = x.state !== "owned" && x.a?.includes("r");
```

### Keep functions small and purposeful

A function should generally do one job:

- classify a resource
- map backend state to UI copy
- submit an action
- render a visual section


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autark-labs/autark-os](https://github.com/autark-labs/autark-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
