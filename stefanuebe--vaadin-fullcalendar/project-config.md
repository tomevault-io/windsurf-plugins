---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FullCalendar for Flow is a Vaadin Flow integration of the FullCalendar JavaScript library (v6). It provides Java components that wrap FullCalendar for use in Vaadin applications.

**Version 7**: Vaadin 25 / Java 21 (current)
**Version 6**: Vaadin 14-24 / Java 11

## Build Commands

```bash
# Build all modules (install to local repo for cross-module deps)
mvn clean install

# Build for production (optimized frontend)
mvn clean install -Pproduction -DskipTests

# Run the demo application (requires production profile for full build)
cd demo && mvn spring-boot:run -Pproduction

# Run unit tests (all modules)
mvn test

# Run a single test class
mvn test -pl addon -Dtest=EntryTest

# Run a single test method
mvn test -pl addon -Dtest=EntryTest#testSomeMethod

# Run integration tests
mvn verify

# Run E2E tests (starts Vaadin app + Playwright)
cd e2e-test-app && mvn clean verify -Pit

# Run mutation testing (PIT — unit tests)
mvn test -pl addon -Ppit
# Report: addon/target/pit-reports/index.html

# Run manual mutation test scripts (see specs/verification.md §3)
bash mutation-test-b.sh        # Unit mutations (~75s, standalone)
bash mutation-test-a.sh        # E2E mutations (~5min, requires app on :8080)

# Alternative: Use Maven wrapper from demo/ if mvn not available
./demo/mvnw clean install
```

## Release Workflow

The `v-herd-demo` branch always reflects the **currently released** version — it is the source for the demo server and the MCP server deployment. Therefore, releases must happen in a strict order so the tag, the demo branch, and the next snapshot all line up.

For every release (patch, minor, or major):

1. **Strip `-SNAPSHOT`** from every `<version>` and `<fullcalendar.version>` across all POMs (root, `addon`, `addon-scheduler`, `demo`, `e2e-test-app`).
2. **Update `ADDON_VERSION`** in `demo/src/main/java/org/vaadin/stefan/ui/layouts/AbstractLayout.java` to the release version (without `-SNAPSHOT`). This string constant is rendered in the demo/MCP-server footer, so it must match the deployed artifact version. The same file also exists in `spike/` but is not on the release path — leave it alone.
3. **Commit** as `Release <version>` (matches the style of existing release commits, e.g. `5f861d00`). The commit must include both the POM and the `ADDON_VERSION` change so the tag and the `v-herd-demo` merge both carry a consistent tree.
4. **Tag** the commit as annotated tag `<version>` with message `Release <version>` (e.g. `git tag -a 7.2.1 -m "Release 7.2.1"`).
5. **Do not bump the snapshot yet.** The next snapshot bump goes *after* step 6.
6. **Sync `v-herd-demo` to the release state.** Check out `v-herd-demo` and merge master (`git merge master -X theirs` to take master's POM values over the historical `v-herd-version` commit). Resulting tree should show the released version across all POMs *and* in `ADDON_VERSION`. Push.
7. **Back on master: bump to the next snapshot** (usually `+1` on the patch, e.g. `7.2.1 → 7.2.2-SNAPSHOT`). Update both the POMs *and* `ADDON_VERSION` (e.g. `"7.2.2-SNAPSHOT"`). Commit as `Bump to <next>-SNAPSHOT`.
8. **Push master and the new tag.**

Why this order matters: if the snapshot is bumped before `v-herd-demo` is synced, merging master into `v-herd-demo` brings in the snapshot version — then the demo server redeploys against a `-SNAPSHOT` artifact that isn't in any public repo, and the deployment breaks.

Why `ADDON_VERSION` matters: the demo server and the MCP server both render this constant as "Version X" in their UI. If it drifts from the actual POM version, deployed demos display a misleading version number. Forgetting to update it at release time has historically required a follow-up cherry-pick into `v-herd-demo`, which is the kind of thing the workflow is supposed to prevent.

Never push the release tag before confirming with the user — tags are public the moment they hit the remote and can't be rewritten cleanly.

## Verification / Testing Rules

- **Bug report triage: always check for existing tests first.** When a bug is reported, immediately check whether an existing test (unit, integration, or E2E) covers the affected use case. If a test exists but didn't catch the bug, fix the test. If no test exists, create one.
- **Every bug fix must include a verification test.** No bug fix is complete without a test that reproduces the bug and verifies the fix.
- **Every new feature must include verification** as defined in the relevant spec (see `specs/` directory).
- **Never commit test code without running it first.** Tests must pass before they are considered done.
- **Do not start a dev server for the user to verify.** Claude runs inside the devcontainer and its servers are unreachable from the host browser. For UI verification either (a) run Playwright yourself inside the devcontainer and report findings, or (b) hand verification back to the user explicitly ("please run `mvn spring-boot:run` on your host and tell me what you see"). Never leave a background `spring-boot:run` expecting the user to click around.

## Module Structure

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanuebe/vaadin-fullcalendar](https://github.com/stefanuebe/vaadin-fullcalendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
