---
trigger: always_on
description: Jules Extension - VS Code extension for managing Google Jules sessions, GitHub PR workflows, activity views, and local repository operations.
---


# Jules Extension - Copilot Instructions

**Jules Extension** is a TypeScript-based VS Code extension for creating, monitoring, and reviewing Google Jules coding sessions directly inside VS Code. It also handles GitHub authentication, PR status checks, diff viewing, and branch operations. This repository is a single-package extension project. Source code lives in `src/`, production output lives in `dist/`, and compiled test output lives in `out/`.

In this repository, the job is not finished when a PR is opened. A task is only complete after review conversations have replies, conversations are resolved, and CI has been monitored through completion. The `gh` workflow and the repeated `sleep 300 && gh pr checks <PR#>` loop are mandatory operating procedures here.

## Quick Reference

| Item | Tech / Responsibility | Location |
| --- | --- | --- |
| Extension core | TypeScript, VS Code Extension API | `src/extension.ts` |
| Jules API integration | Google Jules API client | `src/julesApiClient.ts` |
| GitHub integration | OAuth, repo URL, PR helpers | `src/githubAuth.ts`, `src/githubUtils.ts` |
| UI | Tree View, Chat View, Document Provider | `src/chatView.ts`, `src/planDocumentProvider.ts` |
| Session diff support | Diff, changeset, artifacts | `src/sessionArtifacts.ts`, `src/sessionContextMenuArtifacts.ts` |
| Security helpers | Log sanitization, URL credential stripping | `src/securityUtils.ts` |
| Tests | Mocha + `vscode-test` | `src/test/` |
| Build | esbuild | `esbuild.js`, `dist/` |
| CI | GitHub Actions, pnpm, Node 20/22 | `.github/workflows/` |

---

## Project Overview

This project is not a monorepo. The repository contains a single VS Code extension package with its code and configuration at the root.

- The extension manages Jules sessions, activities, chat, plan approval, PR opening, and diff inspection inside VS Code.
- GitHub auth and PR visibility are first-class features. Remote branch state, PR URL resolution, and GitHub sign-in flows are easy to break and must be treated carefully.
- Security-sensitive logging behavior is explicit in this codebase. Sanitization and credential stripping are part of the supported behavior, not optional cleanup.
- Do not hand-edit generated outputs in `dist/` or `out/`. Edit `src/` and regenerate through the normal build and test flow.

## Key Files and Responsibilities

| Purpose | Path |
| --- | --- |
| Extension entry point and command registration | `src/extension.ts` |
| Jules API communication | `src/julesApiClient.ts` |
| GitHub auth flow | `src/githubAuth.ts` |
| GitHub URL and repo helpers | `src/githubUtils.ts` |
| Session context menu actions | `src/sessionContextMenu.ts` |
| Diff and changeset presentation | `src/sessionContextMenuArtifacts.ts` |
| Session artifact cache | `src/sessionArtifacts.ts` |
| Chat webview | `src/chatView.ts` |
| Plan review document provider | `src/planDocumentProvider.ts` |
| Security utilities | `src/securityUtils.ts` |
| Unit and integration tests | `src/test/*.ts` |

When changing implementation, always inspect the related test coverage in the same area.

---

## Setup and Core Commands

### Initial Setup

```bash
pnpm install --frozen-lockfile
```

### Development and Build

```bash
pnpm run check-types   # TypeScript type check
pnpm run lint          # ESLint on src
pnpm run compile       # type check + lint + esbuild
pnpm run package       # production packaging build
pnpm run watch         # esbuild and tsc watch
```

### Tests

```bash
pnpm run compile-tests # compile tests into out/
pnpm run test:unit     # fast unit test pass
pnpm test              # vscode-test based extension test run
```

`pnpm test` triggers `pretest`, which runs `compile-tests`, `compile`, and `lint` first. It is slower and should be treated as a final verification step after targeted validation with `pnpm run test:unit`.

---

## Test Code Expectations

Tests are important in this repository. Do not treat test updates as optional when changing behavior.

### Test Layout

- Unit or module-focused tests: `src/test/*.unit.test.ts`
- Broader behavior or integration-style tests: `src/test/*.test.ts`
- VS Code mocks: `src/test/vscodeMock.ts`
- Security testing reference: `TESTING_GUIDE.md`

### Representative Test Files

- `src/test/activityUtils.unit.test.ts`
- `src/test/githubUtils.unit.test.ts`
- `src/test/julesApiClient.unit.test.ts`
- `src/test/securityUtils.unit.test.ts`
- `src/test/githubIntegration.test.ts`
- `src/test/extension.test.ts`
- `src/test/planNotification.unit.test.ts`

### Test Policy

- Always add or update tests for the module you changed.
- For changes in central modules such as `src/extension.ts`, `src/githubAuth.ts`, `src/sessionArtifacts.ts`, or `src/securityUtils.ts`, do not stop at unit tests if broader extension behavior may be affected. Run `pnpm test` when appropriate.
- For bug fixes, prefer adding a reproducing test first and then applying the fix.
- For security-related changes, review `TESTING_GUIDE.md` and do not weaken existing expectations around sanitization, credential handling, or edge cases.

### Minimum Verification for Most Changes

```bash
pnpm run check-types
pnpm run lint
pnpm run test:unit
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hiroki-org/jules-extension](https://github.com/Hiroki-org/jules-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
