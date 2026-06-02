---
trigger: always_on
description: This document is intended for AI coding agents (e.g., OpenAI Codex) working in this repository.
---

# AGENTS.md — Strømligning Integration

This document is intended for AI coding agents (e.g., OpenAI Codex) working in this repository.
It defines setup, constraints, workflow, safety rules, and quality expectations for the Webast Connect integration.

Agents must follow this document strictly.

---

## Agent Objectives

1. Implement and maintain the Strømligning integration.
2. Keep changes minimal, isolated, and testable.
3. Prefer deterministic, explicit implementations over implicit or heuristic behavior.
4. Never fabricate missing technical details.

---

## No-Assumption Rule (Facts Only)

If required technical details are missing, the agent MUST:

- Locate the information inside this repository or from the `pystromligning` repository, or
- Explicitly request clarification before implementing a dependent solution.

The agent must NOT:

- Invent API endpoints
- Invent protocol structures
- Guess authentication flows
- Introduce undocumented environment variables

If there is uncertainty, stop and request clarification.

---

## Repository Structure

(Adjust once the structure is finalized.)

- `custom_components/stromligning` — Integration logic
- `tests/` — Unit and integration tests
- `docs/` — Protocol documentation, architecture notes
- `scripts/` — Development utilities, mostly for running Home Assistant tasks

If the repository becomes a monorepo, additional `AGENTS.md` files may exist in subdirectories with scoped instructions.

## External Module Repository

The Python communication module source code is maintained in:

- `/workspaces/pystromligning`

From this workspace, the agent has access to work in that repository as well, including creating branches, making edits, committing, and pushing.

All module changes MUST be made in dedicated branches intended for merge (no direct commits to main/master), so full change tracking is preserved.

When testing integration changes in this repository that depend on in-progress `pystromligning` module changes, the agent MUST update `custom_components/stromligning/manifest.json` to point to the specific `pystromligning` branch being used for that test cycle.

When asked to merge, revert the `custom_components/stromligning/manifest.json` requirement changes and check if there is a newer version of the `pystromligning` module, available from PyPI.

## Environment & Setup

The agent must use the exact versions defined in the project configuration files.

### Requirements

- Runtime: Python 3.14

## Strømligning Integration Rules

Devices must be treated as external hardware systems.

The agent must:

- Avoid changes that require physical hardware validation unless:
  - Proper mocks are provided, or
  - A clear manual test plan is included.
- Ensure network calls include timeouts.
- Avoid infinite retry loops.
- Handle disconnections gracefully.

If certificates or tokens are required:

- Never commit them.
- Never log them in plaintext.
- Always load them from environment variables or a local secure store.
- Document required environment variables clearly.

---

## Logging & Error Handling

- Never log credentials, tokens, certificates, or sensitive identifiers.
- Prefer structured errors where applicable.
- Fail explicitly rather than silently ignoring errors.
- Surface actionable error messages.
- ALWAYS test for race conditions in relevant async/concurrent flows before considering a change complete.

---

## Code Standards

- Follow existing project formatting and naming conventions.
- Do not introduce large refactors in the same change as functional modifications unless explicitly requested.
- Keep commits small and focused.
- Avoid introducing new dependencies unless justified.
- Use `ruff` as formatter/linter in this repository, if not found local, install it.
- Before committing, the agent MUST run the agreed `ruff` commands for this repository, including `ruff format` and `ruff check`, and fix any reported issues.

## Home Assistant Compliance

The integration MUST comply with Home Assistant integration standards and developer guidelines.

The agent must:

- Follow Home Assistant architecture patterns for config entries, setup/unload flows, and platform forwarding.
- Implement entities according to Home Assistant entity model conventions (state, availability, device info, unique IDs, and naming).
- Use `DataUpdateCoordinator` where periodic or shared polling is required.
- Provide and maintain `config_flow`, diagnostics/repair handling (when relevant), and translations.
- Missing translations may be added directly in this repository, including with AI assistance, as long as the result is consistent with the English source strings and current integration behavior.
- Keep `manifest.json`, services, and supported features aligned with Home Assistant requirements.
- Ensure changes target and maintain at least Home Assistant Silver quality expectations, and move toward Gold where feasible.
- Add or update tests for behavior changes, especially setup, coordinator behavior, and entity state handling.

---

## Git Workflow

Branch naming convention:

```text
feature/<name> - for introducing new features
fix/<name> - fixing bugs
enhancement/<name> - for code or stability enhancements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MTrab/stromligning](https://github.com/MTrab/stromligning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
