---
trigger: always_on
description: This is a TypeScript-based GitHub Action that sends Events and Metrics to a
---

# Dynatrace GitHub Action — Coding Guidelines

## Project Overview

This is a TypeScript-based GitHub Action that sends Events and Metrics to a
Dynatrace monitoring environment via the Dynatrace REST API. Entry point:
`src/index.ts` → `src/main.ts`. Core logic lives in `src/dynatrace.ts`.

## Build & Test

```bash
npm install          # install dependencies
npm test             # run tests (Jest + ts-jest)
npm run all          # format + lint + test + coverage + bundle
npm run package      # bundle dist/index.js via ncc
```

The bundled `dist/index.js` is what GitHub Actions executes (see `action.yml`).

## GitHub Action Implementation Guidelines

### action.yml

- Keep inputs minimal and well-described; mark required inputs with
  `required: true`.
- Always specify the `runs.using` runtime explicitly (currently `node22`).
- The `main` field must point to the compiled entry point (`dist/index.js`).
- Update `inputs` in `action.yml` whenever new action inputs are added.

### TypeScript Source (`src/`)

- Use strict TypeScript (`tsconfig.json`); do not use `any` unless unavoidable.
- Validate all action inputs at the boundary (`src/main.ts`) before passing them
  deeper.
- Use `@actions/core` for reading inputs (`getInput`), logging (`info`,
  `warning`, `error`), and failing the action (`setFailed`).
- Prefer `@actions/http-client` for HTTP calls to Dynatrace APIs.
- Never log sensitive values (tokens, secrets) — mask them with `core.setSecret`
  if they appear in outputs.
- Keep API interaction logic in `src/dynatrace.ts`, orchestration in
  `src/main.ts`, and the Action entry point thin in `src/index.ts`.

### Security

- Do not accept unvalidated external input in API calls — sanitise all metric
  keys, metric values, and event properties.
- Store secrets only in GitHub Secrets, never in plain workflow YAML.
- API tokens must only be passed via the `token` input, which references
  `${{ secrets.DT_TOKEN }}`.

### Error Handling

- Use `core.setFailed(message)` for unrecoverable errors so the workflow step is
  marked as failed.
- Use `core.warning(message)` for non-fatal issues the user should know about.
- Always provide actionable error messages referencing the relevant input field.

## Test Coverage

- **Every new or changed function in `src/dynatrace.ts` must have corresponding
  unit tests in `__tests__/dynatrace.test.ts`.**
- **Every new input or code path in `src/main.ts` must be tested in
  `__tests__/index.test.ts` or a dedicated test file.**
- Tests use Jest with `ts-jest`; do not use `any` casts in tests.
- Mock `@actions/core` and HTTP calls — tests must not make real network
  requests.
- Aim for branch coverage on all validation and error-handling paths.
- Run `npm test` before committing; all tests must pass.

## Documentation (README.md)

- **All user-visible changes must be reflected in `README.md` before merging.**
- New inputs → add a row to the Inputs table.
- New metric formats or event types → add to the respective lists.
- New features or behaviour changes → add or update the relevant section and, if
  appropriate, add a usage example.
- Keep examples up to date with the latest major version tag (e.g., `@v9`).

## Contributing & Pull Request Quality Gate

Before a pull request can be merged, all of the following must be satisfied:

### Automated checks (must be green)

| Check                   | Command / Workflow                         |
| ----------------------- | ------------------------------------------ |
| Formatting              | `npm run format:check` (Prettier)          |
| Linting                 | `npm run lint` (ESLint)                    |
| Tests pass              | `npm test`                                 |
| Coverage badge updated  | `npm run coverage`                         |
| Distribution up to date | `npm run package` — commit `dist/` changes |
| CI workflow             | `.github/workflows/ci.yaml`                |
| dist/ check             | `.github/workflows/check-dist.yaml`        |

### Manual review checklist

- [ ] New/changed functionality is described correctly and completely in
      `README.md`.
- [ ] All new action inputs are documented in the Inputs table in `README.md`.
- [ ] Any new metric formats or event types are listed in `README.md`.
- [ ] `action.yml` is updated if inputs changed.
- [ ] No secrets or tokens are hardcoded or logged.
- [ ] Tests cover the new code paths, including error cases.
- [ ] No breaking changes without a major version bump.

## Release Process

### Versioning

Follow [Semantic Versioning](https://semver.org/): `vMAJOR.MINOR.PATCH`.

- **PATCH** — bug fixes, no API changes.
- **MINOR** — new backwards-compatible inputs or features.
- **MAJOR** — breaking changes to inputs, outputs, or behaviour.

### Steps to Publish a New Release

1. **Ensure the main branch is clean and all CI checks pass.**

2. **Update version** in `package.json`:

   ```bash
   npm version patch   # or minor / major
   ```

3. **Rebuild the distribution bundle:**

   ```bash
   npm run all
   ```

   Commit the updated `dist/` directory and `package.json`/`package-lock.json`.

4. **Update `README.md`:**

   - Bump the version tag in all usage examples (e.g., `@v9` → `@v10`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dynatrace-oss/dynatrace-github-action](https://github.com/dynatrace-oss/dynatrace-github-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
