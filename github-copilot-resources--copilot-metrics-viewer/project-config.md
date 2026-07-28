---
trigger: always_on
description: Nuxt 3 web application for GitHub Copilot usage metrics and analytics
---


# GitHub Copilot Metrics Viewer

GitHub Copilot Metrics Viewer is a Nuxt 3 web application that displays GitHub Copilot usage metrics and analytics for organizations and enterprises. The application visualizes data from the GitHub Copilot Metrics API using Vue.js, TypeScript, Vuetify, and Chart.js.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Security and Boundaries

### Critical Rules
- **NEVER commit secrets or credentials** to the repository
- **NEVER modify `.env` file** - environment variables should only be documented, not changed
- **DO NOT modify** the following without explicit approval:
  - Production configuration files (azure.yaml, Dockerfile)
  - GitHub workflows in `.github/workflows/`
  - Security policies (SECURITY.md, CODE_OF_CONDUCT.md)
  - License files (LICENSE.txt)
- **ALWAYS validate** that changes don't introduce security vulnerabilities
- **ALWAYS run security scanning** before finalizing changes

### Safe Modification Areas
- Application source code in `app/`, `server/`, `shared/`
- Tests in `tests/` and `e2e-tests/`
- Documentation files (README.md, CONTRIBUTING.md, DEPLOYMENT.md)
- Configuration files specific to your changes

## Working Effectively

### Initial Setup
- **Node.js requirement**: Uses Node.js 20+ (verified: v20.19.4 works)
- Install dependencies: `npm install` 
  - **NEVER CANCEL**: Takes 3 minutes to complete. Set timeout to 5+ minutes.
  - Includes postinstall script that runs `nuxt prepare`

### Build and Development
- **Development server**: `npm run dev`
  - Starts on http://localhost:3000/
  - **Font provider warnings are normal** - application works despite "Could not fetch fonts" errors
  - Supports hot reload and auto-refresh
- **Production build**: `npm run build`
  - **NEVER CANCEL**: Takes 30 seconds to complete. Set timeout to 2+ minutes.
  - Builds successfully despite font provider connection warnings
  - Outputs to `.output/` directory
- **Production preview**: Built server requires proper environment setup
  - After build: `NUXT_SESSION_PASSWORD=something_long_and_random_thats_at_least_32_characters node .output/server/index.mjs`
  - **NOTE**: Health endpoints may not work correctly in built mode in some environments
  - **Recommendation**: Use `npm run dev` for development and testing validation scenarios

### Testing
- **Unit tests**: `npm test` (using Vitest)
  - **NEVER CANCEL**: Takes 15 seconds to complete. Set timeout to 2+ minutes.
  - Runs 97 tests, all should pass
  - Uses mocked data environment
  - Test files are located in `tests/` directory
- **E2E tests**: `npm run test:e2e` (using Playwright)
  - **NOTE**: Playwright browser installation may fail in some environments due to download issues
  - Install browsers first: `npx playwright install` 
  - Uses mocked data for testing
- **Type checking**: `npm run typecheck`
  - **KNOWN ISSUE**: Currently fails with 18 TypeScript errors
  - Takes 10 seconds to complete
  - Errors are in existing codebase, not blocking for development

### Bug Fix Workflow (TDD)
When fixing any bug, **always follow this order**:
1. **Write a failing test first** that reproduces the bug — run `npm test` and confirm the new test fails
2. **Apply the fix** to the production code
3. **Run `npm test` again** and confirm the previously failing test now passes and no other tests regressed
4. Commit both the fix and the test together (or test first in a separate commit)

This ensures every bug has a regression guard before the fix lands.

### Code Quality
- **Linting**: `npm run lint`
  - **KNOWN ISSUE**: Currently fails with 43 ESLint errors (mostly @typescript-eslint/no-explicit-any)
  - Takes 3 seconds to complete
  - `npm run lint:fix` can fix some formatting issues but not the core errors
  - **Always run linting** but expect failures in current codebase

## Environment Configuration

### Required Environment Variables
- **NUXT_SESSION_PASSWORD**: Required, minimum 32 characters
  - Used for session encryption
  - Example: `NUXT_SESSION_PASSWORD=something_long_and_random_thats_at_least_32_characters`

### GitHub Integration
- **Mock mode (default)**: `NUXT_PUBLIC_IS_DATA_MOCKED=true`
  - Works without GitHub tokens
  - Uses sample data for development and testing
- **Real GitHub data**: Requires GitHub Personal Access Token
  - `NUXT_GITHUB_TOKEN=<your_token>`
  - Token needs permissions: Read access to members, organization copilot metrics, and organization copilot seat management

### Scope Configuration
- **NUXT_PUBLIC_SCOPE**: Sets default scope ('organization', 'enterprise', 'team-organization', 'team-enterprise')
- **NUXT_PUBLIC_GITHUB_ORG**: Target organization name
- **NUXT_PUBLIC_GITHUB_ENT**: Target enterprise name
- **NUXT_PUBLIC_GITHUB_TEAM**: Target team name (optional)

### OAuth Configuration (Optional)
- **NUXT_PUBLIC_USING_GITHUB_AUTH**: Enable GitHub OAuth (default: false)
- **NUXT_OAUTH_GITHUB_CLIENT_ID**: GitHub App client ID
- **NUXT_OAUTH_GITHUB_CLIENT_SECRET**: GitHub App client secret

## Validation

### Manual Testing Scenarios
Always test these scenarios after making changes (use development mode for reliable validation):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github-copilot-resources/copilot-metrics-viewer](https://github.com/github-copilot-resources/copilot-metrics-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
