---
trigger: always_on
description: Homebridge is a lightweight Node.js server that emulates the iOS HomeKit API, allowing "smart home" devices to integrate with Apple's HomeKit ecosystem through community-contributed plugins.
---

# Homebridge

Homebridge is a lightweight Node.js server that emulates the iOS HomeKit API, allowing "smart home" devices to integrate with Apple's HomeKit ecosystem through community-contributed plugins.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Prerequisites for Assignment

**CRITICAL**: Before any issue can be assigned to Copilot, it must have one of these labels applied:

- **`patch`**: For bug fixes and small improvements (patch version bump: x.y.z → x.y.z+1)
- **`minor`**: For new features and enhancements (minor version bump: x.y.z → x.y+1.0)
- **`major`**: For breaking changes and major refactors (major version bump: x.y.z → x+1.0.0)

Without these labels, Copilot cannot determine the appropriate branch targeting strategy and version increment.

## Branch Targeting Strategy

When creating pull requests or working with branches, always follow this strategy based on the issue label:

### Label-Based Targeting:

- **`patch` label** (bug fixes): Target for patch version increment (e.g., 1.11.0 → 1.11.1)
- **`minor` label** (new features): Target for minor version increment (e.g., 1.11.0 → 1.12.0)
- **`major` label** (breaking changes): Target for major version increment (e.g., 1.11.0 → 2.0.0)

### Branch Targeting Priority:

1. **Target the lowest available beta branch** that matches or exceeds the required version bump (e.g., beta-2.0.0, beta-3.0.0)
2. **If no suitable beta branch exists**, create a new beta branch following this naming convention:
   - For patch: `beta-{current.major}.{current.minor}.{current.patch+1}` (e.g., beta-1.11.1)
   - For minor: `beta-{current.major}.{current.minor+1}.0` (e.g., beta-1.12.0)
   - For major: `beta-{current.major+1}.0.0` (e.g., beta-2.0.0)
3. **Only target the latest branch as a last resort** and only for critical hotfixes

This ensures changes are properly tested in beta releases with appropriate version bumps before being merged to the main release branch.

## Beta Branch Creation

When no suitable beta branch exists for the required version bump:

1. **Determine the target version** based on the issue label and current version (found in `package.json`)
2. **Create a new beta branch** using the GitHub API or request branch creation from maintainers
3. **Use the naming convention**: `beta-{major}.{minor}.{patch}` (e.g., beta-1.12.0, beta-2.0.0)
4. **Base the new branch** on the latest stable branch to ensure it includes all current changes
5. **Target your PR** to the newly created beta branch

Example branch creation scenarios (assuming current version 1.11.0):

- Patch fix → Create `beta-1.11.1` if it doesn't exist
- Minor feature → Create `beta-1.12.0` if it doesn't exist
- Major change → Create `beta-2.0.0` if it doesn't exist

## Working Effectively

- Bootstrap, build, and test the repository:
  - `npm install` -- takes ~20 seconds. Install dependencies first.
  - `npm run build` -- takes ~3 seconds. Compiles TypeScript to JavaScript in the dist/ directory.
  - `npm run test` -- takes ~7 seconds. Runs the Vitest test suite.
  - `npm run lint` -- takes ~2 seconds. ALWAYS run before committing or CI will fail.
- Run Homebridge:
  - ALWAYS run build steps first
  - `./bin/homebridge.js -D` -- starts in debug mode, requires config.json in ~/.homebridge/
  - `./bin/homebridge.js -D -U /path/to/config` -- uses custom config directory
  - `./bin/homebridge.js --help` -- shows all CLI options
- Development mode:
  - `npm run dev` -- runs with DEBUG=\* environment and example plugins
  - `npm run watch` -- uses nodemon to auto-rebuild and restart on changes

## Validation

- ALWAYS manually validate Homebridge startup after making changes by running `./bin/homebridge.js -D`
- ALWAYS run `npm run lint` before committing or the CI (.github/workflows/build.yml) will fail
- Run `npm run test-coverage` to generate coverage reports (takes ~10 seconds)
- Verify that Homebridge shows QR code and bridge information when starting successfully
- Test with minimal config to ensure core functionality works

## Plugin Development Workflow

The most common development scenario is creating or testing Homebridge plugins:

- Create plugin in separate directory
- In plugin directory: `npm link` (makes plugin globally available)
- Create test config in `~/.homebridge-dev/config.json` with your plugin configuration
- Start Homebridge: `homebridge -D -U ~/.homebridge-dev` (uses separate config from production)
- Undo plugin link: `npm unlink` when done

## Common Tasks

The following are outputs from frequently run commands. Reference them instead of viewing, searching, or running bash commands to save time.

### Repository structure

```
/home/runner/work/homebridge/homebridge/
├── README.md              # Main documentation
├── package.json           # npm configuration and scripts
├── tsconfig.json          # TypeScript compiler configuration
├── eslint.config.js       # ESLint configuration
├── vitest.config.js       # Vitest test configuration
├── bin/homebridge.js      # Main executable script

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homebridge/homebridge](https://github.com/homebridge/homebridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
