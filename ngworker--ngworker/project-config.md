---
trigger: always_on
description: **Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---


# @ngworker/spectacular Angular Integration Testing

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

This is an Nx workspace containing the Spectacular Angular integration testing library and its documentation. The main package `@ngworker/spectacular` is a public npm package providing test harnesses for Angular applications, features, and pipes.

## Working Effectively

### Bootstrap the Repository

Assume that your environment has a supported version of Node.js and npm installed and that `npm ci` has been run to install dependencies. Actions setup steps are available in `.github/workflows/copilot-setup-steps.yml` instead of manual Node.js/npm and dependency installation.

Should you need to re-bootstrap the repository, follow these steps:

1. **Prerequisites**:
   - Node.js version: `node --version` (requires v22.19.0)
   - npm version: `npm --version` (requires v10.9.3)

2. **Install Dependencies** (24 seconds):
   ```bash
   npm ci
   ```
   NEVER CANCEL: Wait for completion. Expect npm audit warnings - these are acceptable. Make sure to run this command after making changes to `package.json` or `package-lock.json`.

### Build Commands

- **Build all packages** (10 seconds): `npm run build`
- **Build specific project**: `npx nx build <project-name>`
- **Build docs** (55 seconds): `npx nx build spectacular-docs` NEVER CANCEL: Documentation build takes 55+ seconds. Set timeout to 90+ minutes.

### Test Commands

- **Run all tests** (14 seconds): `npm run test`
- **Run specific project tests**: `npx nx test <project-name>`
- **Run tests with configuration**: `npm run test -- --configuration=ci`

### Lint and Format Commands

- **Check formatting** (6 seconds): `npm run format:check`
- **Fix formatting**: `npm run format`
- **Run linting** (3 seconds with cache): `npm run lint`

### Affected Commands (Incremental)

Use these for faster CI-style workflows:

- **Build affected**: `npm run affected:build`
- **Test affected**: `npm run affected:test`
- **Lint affected**: `npm run affected:lint`
- **Format check affected**: `npm run format:check -- --base=origin/main`

### CI Pipeline Validation

Always run before committing changes:

```bash
npm run format:check && npm run lint && npm run test && npm run build
```

**Use the CI scripts for comprehensive validation**:

- `npm run ci` - Full CI validation including build (format:check && lint && test && build && e2e)
- `npm run affected:ci` - CI validation for affected projects only

NEVER CANCEL: Full CI validation takes 30-90 seconds. Use timeout of 120+ seconds.

## Validation Scenarios

### Manual Validation Requirements

After making changes, always validate by running through these scenarios:

1. **Library Functionality Test**:

   ```bash
   npx nx test spectacular
   ```

   Verify all 121 tests pass in ~14 seconds.

2. **Documentation Site Test**:

   ```bash
   npx nx build spectacular-docs
   cd dist/packages/spectacular-docs
   python3 -m http.server 8000
   ```

   Navigate to http://localhost:8000 and verify the docs site loads correctly.

3. **Package Build Validation**:
   ```bash
   npx nx build spectacular
   ls -la dist/packages/spectacular/
   ```
   Verify the package builds and contains expected files.

## Repository Structure

### Key Projects

- **`spectacular`**: Main Angular testing library (packages/spectacular/)
  - Public npm package: `@ngworker/spectacular`
  - Provides test harnesses for Angular apps, features, and pipes
  - **BREAKING CHANGE WARNING**: Do not introduce breaking changes unless explicitly instructed
- **`spectacular-docs`**: Documentation website (packages/spectacular-docs/)
  - Docusaurus-based documentation site
  - Builds to dist/packages/spectacular-docs/
  - Deployed to GitHub Pages at https://ngworker.github.io/ngworker/
- **`test-util`**: Internal testing utilities (packages/internal/test-util/)
- **Example packages**: Tour of Heroes examples for testing (packages/examples/)

### Important Files

```
/
├── .github/workflows/ci.yml        # Main CI pipeline
├── .node-version                   # Node.js version requirement (lts/jod)
├── nx.json                         # Nx workspace configuration
├── package.json                    # Root package with scripts
├── jest.config.ts                  # Jest test configuration
└── packages/
    ├── spectacular/                # Main library
    ├── spectacular-docs/           # Documentation site
    ├── internal/test-util/         # Internal utilities
    └── examples/                   # Example packages
```

## Build Times and Timeouts

**CRITICAL**: NEVER CANCEL builds or tests. Always set appropriate timeouts:

| Command                         | Expected Time | Timeout Setting |
| ------------------------------- | ------------- | --------------- |
| `npm ci`                        | 24 seconds    | 60 seconds      |
| `npm run build`                 | 10 seconds    | 60 seconds      |
| `npm run test`                  | 14 seconds    | 60 seconds      |
| `npx nx build spectacular-docs` | 55 seconds    | 120 seconds     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngworker/ngworker](https://github.com/ngworker/ngworker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
