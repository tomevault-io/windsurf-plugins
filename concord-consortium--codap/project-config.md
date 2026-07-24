---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About CODAP

CODAP (Common Online Data Analysis Platform) is an educational technology tool designed to help teach and learn data analysis and graphing. It is primarily used in classroom environments with students ranging from middle school through college. UI/UX recommendations and security considerations should account for this educational context and age range.

## Public-Facing Messages

When performing an action that includes a message visible to others — git commit messages, PR descriptions, PR review comments, Jira comments, Slack messages, etc. — always show the proposed message text to the user and get explicit confirmation before submitting. Authorizing an action (e.g., "yes, commit" or "yes, post the review") is NOT the same as approving the message content. The user must always have the opportunity to read, edit, or rewrite the message before it goes out.

## Repository Structure

This repository contains two versions of CODAP (Common Online Data Analysis Platform):
- **v2 (legacy)**: Root directory, SproutCore-based, uses `master` branch
- **v3 (active development)**: Located in `/v3`, React/TypeScript/MobX-State-Tree, uses `main` branch

**Active development is in v3.** Most work should be done in the `/v3` directory.

**Important:** Git commands must be run from the repository root (`/v3`'s parent), not from inside `/v3`. Since npm/jest/lint commands run from `/v3`, the working directory often drifts there. Always use absolute paths or `cd` back to the repo root before running git commands to avoid `v3/v3/...` path errors.

## Development Workflow

### Branch Policy

**Never commit directly to `main` or push to `main`.** All work must be done on a feature/PR branch, reviewed via pull request, and merged through GitHub. This applies even for small fixes, review feedback, or single-commit changes. If you're about to commit and the current branch is `main`, stop and create a branch first.

### Starting Work on a Jira Story

When the user says "Let's start work on CODAP-XXX" (or similar):

1. **Ask about the base branch**: Usually `main`, but occasionally work builds on a previous PR branch. Confirm before creating the branch.

2. **Create a feature branch**:
   - Branch name format: `{JIRA-ID}-{short-description}`
   - Example: `CODAP-1027-inbounds-url-param`
   - Use lowercase kebab-case for the description
   - Keep it concise but descriptive
   - **Always propose the branch name and get user approval before creating it**

3. **Update Jira status**: Transition the story to "In Progress"

### Creating a Pull Request

When the work is ready for initial CI validation:

1. **Create a PR**:
   - Title format: `{JIRA-ID}: {description}` (e.g., `CODAP-1027: add inbounds URL parameter`)
   - Description should include:
     - Summary of the changes
     - Reference to the Jira story (e.g., `Fixes CODAP-1027` or link to the story) so Jira links it automatically
   - Apply the `v3` label
   - **For PRs with code changes**: Create as a draft PR. Mark as "Ready for Review" only after CI passes.
   - **For docs-only PRs** (no code changes): Create as a regular (non-draft) PR — no need for CI validation.

2. **Initial CI validation** (code PRs only):
   - CI runs automatically on PRs (lint, build, limited Cypress tests)
   - Review CI results and fix any issues

3. **Finalize the PR**:
   - Request Copilot review
   - When PR is nearly ready for human review, apply the `run regression` label
   - The `run regression` label triggers the full Cypress test suite (we don't run it automatically to conserve our Cypress quota)

4. **Ready for review**:
   - Once full Cypress tests pass on CI:
     - Mark PR as "Ready for Review" (no longer draft)
     - Assign to another developer for review
     - Transition the Jira story to "In Code Review"
   - Ensure Jira story has approvers specified:
     - **Developer Approver**: The code reviewer
     - **Project Team Approver**: Someone to verify the fix/feature
   - If approvers aren't specified, ask the user about them

5. **After code review approval**:
   - Mark PR as approved
   - Transition Jira story to "Ready for Merge"

6. **After merge**:
   - Once the PR is merged and builds successfully on `main`
   - Transition Jira story to "In Project Team Review"

7. **Project team review**:
   - Project Team Approver verifies the fix/feature
   - If approved: Transition Jira story to "Done"
   - If rejected: Either stay in current status (for discussion) or transition back to "In Progress" if more work is required

8. **Rejection at any review stage**:
   - If Developer Approver or Project Team Approver rejects:
     - Stay in current review status if there's ongoing discussion
     - Transition back to "In Progress" if more work is needed

### Jira Integration

- The project uses Jira for issue tracking at `concord-consortium.atlassian.net`
- Stories are typically in the CODAP project (e.g., CODAP-1027)
- Use the Atlassian MCP tools to read/update Jira issues
- Jira status transitions:
  - **To Do** → **In Progress**: When starting work on a story

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [concord-consortium/codap](https://github.com/concord-consortium/codap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
