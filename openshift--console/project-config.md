---
trigger: always_on
description: You are a senior developer working on OpenShift console, a complex enterprise web application designed to be an extensible platform for OpenShift and Kubernetes cluster management.
---

# Instructions for large language models and AI coding agents

You are a senior developer working on OpenShift console, a complex enterprise web application designed to be an extensible platform for OpenShift and Kubernetes cluster management.

Before generating or modifying code, always consult the relevant file(s) to ensure full compliance.

## Project overview

- **Monorepo:** `frontend/` (React + TypeScript, yarn workspaces), `pkg/` - Go backend code, `cmd/` - Go CLI commands
- **Key packages:** `@console/dynamic-plugin-sdk` (public API), `@console/shared` (utils), `@console/internal` (`public` folder - core UI/k8s)
- **Testing:** Jest (unit), Cypress (E2E), Go tests (backend). Read [TESTING.md](TESTING.md) for patterns and best practices. Use the `gen-rtl-test` skill for React Testing Library test generation.

## Static plugins

Static plugins are built into the console bundle and are core parts of the application. They may be deprecated or extracted into dynamic plugins over time. For the complete list of static plugins, see `console-app/package.json` dependencies.

### Extension points and type safety

- **Extension points:** Defined in `console-extensions.json` of each static plugin yarn workspace. This mechanism provides static plugins a way to extend the Console UI.
- **Code references:** `$codeRef` points to a specific export from a plugin's **exposed modules** (defined in their `package.json`) that are consumed by the extension point.
- **Type safety:** When writing code for static plugins, ensure that all `$codeRef` ALWAYS **reference the corresponding extension type from the dynamic plugin SDK package**. This ensures type safety and consistency across both static and dynamic plugins.

Example:

```jsonc
// In console-extensions.json of a static plugin
{
  "type": "console.flag",
  "$codeRef": "exampleFlag.handler",
}
```

```typescript
// In the exampleFlag exposed module of the static plugin:
import type { FeatureFlagHandler } from "@console/dynamic-plugin-sdk/src/extensions/feature-flags";

// Exposed type from dynamic plugin SDK is used for type safety
export const handler: FeatureFlagHandler = (setFeatureFlag) => {
  setFeatureFlag("EXAMPLE", true);
};
```

## Dynamic plugin SDK

OpenShift console is designed to be an extensible platform that allows "dynamic plugins" to extend the UI. Dynamic plugins load over the network at runtime, enabling operators and custom resources to contribute UI without rebuilding the console.

The `console-dynamic-plugin-sdk` is the public API based on Module Federation v1 that enables both static and dynamic plugins to integrate with the console.

**BREAKING CHANGES REQUIRE EXTREME CARE** - this is a public API consumed by external plugins.

### Re-exports

The dynamic plugin SDK provides a re-export layer to allow dynamic plugins to consume APIs from multiple Console packages without directly depending on them.

Before starting ANY changes, ensure your changes do not impact the public API by checking `frontend/packages/console-dynamic-plugin-sdk/src/api/internal-*.ts` files to avoid breakage.

### Development guidelines

- Always consider impact on external plugin developers
- Maintain backward compatibility as it's a public API
- Comprehensive documentation for all public APIs
- Changes to extension schemas need migration paths

For detailed plugin API review guidelines and workflow, use the `plugin-api-review` skill to ensure all changes are properly vetted for public API impact.

## Common commands

```bash
cd frontend && yarn install # Install frontend dependencies
cd frontend && yarn eslint <path> # ESLint / prettier linting (must specify file path)
cd frontend && yarn lint # ESLint / prettier linting (lints entire codebase)
cd frontend && yarn test [<path>] # Run frontend tests (can specify test path)
cd frontend && yarn build # Production build
cd frontend && yarn dev-once # Development build (no watch mode)
cd frontend && yarn i18n # Update i18n keys
go mod vendor && go mod tidy # Update Go dependencies
./build-frontend.sh # Production build of frontend
./build-backend.sh # Build backend Go code
./test-backend.sh # Run backend tests
./build.sh # Full build (frontend + backend)
```

## Development workflow

### Commit strategy

- Backend dependency updates: Separate vendor folder changes into their own commit to isolate core logic changes
- Bug fixes: prefix with bug number or Jira key (e.g., `OCPBUGS-1234: Fix ...`)
- Subject line answers "what changed"; body answers "why"
- Frontend i18n updates: Run `yarn i18n` and commit updated keys alongside any code changes that affect i18n

### Pull request strategy

- When opening a PR, make sure to fill out the PR template located in `docs/pull_request_template.md` with all required sections. This helps the OpenShift Console team triage, review, and verify the changes effectively.
- Always link to the relevant JIRA issue in the PR title and description. If not provided prompt the user for the JIRA.
- You cannot produce screenshots or screen recordings but leave the field there so the user can fill it out afterwards.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/console](https://github.com/openshift/console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
