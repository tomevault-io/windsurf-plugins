---
trigger: always_on
description: Butler is a Node.js application that adds "superpowers" to Qlik Sense Enterprise on Windows, including advanced reload failure alerts, task scheduling, key-value store, file system access, and REST API capabilities.
---

# Butler - Agent Guide

Butler is a Node.js application that adds "superpowers" to Qlik Sense Enterprise on Windows, including advanced reload failure alerts, task scheduling, key-value store, file system access, and REST API capabilities.

## Commands

- `npm ci --include=dev --include=prod` — install all deps (33s, NEVER CANCEL, timeout 90+s)
- `npm test` / `npm run test:unit` — run unit tests only (~50 tests, excludes API tests, 10s)
- `npm run test:integration` — run integration tests only (25 REST API tests with Fastify)
- `npm run test:full` — run all tests with coverage
- `npm run lint:fix` then `npm test` — required quality gates
- `npm run lint` — ESLint validation (2s, NEVER CANCEL, timeout 30+s)
- `npm run format` — Prettier (tabWidth: 4, singleQuote: true, printWidth: 140, 4s, NEVER CANCEL, timeout 30+s)
- **Test single file:** `node --experimental-vm-modules node_modules/jest/bin/jest.js src/path/to/file.test.js`
- `node src/butler.js` — start app (needs config file via `-c` flag)
- `node src/butler.js -c ./src/config/config-gen-api-docs.yaml --no-qs-connection` — start with test config

## Build Process

- **Fast esbuild compilation:** `./node_modules/.bin/esbuild src/butler.js --bundle --external:axios --external:xdg-open --external:enigma.js --outfile=./build/build.cjs --format=cjs --platform=node --target=node22 --minify --inject:./src/lib/import-meta-url.js --define:import.meta.url=import_meta_url` (0.4s, NEVER CANCEL, timeout 30+s)
- Create build directory first: `mkdir -p build`
- **Generate Node.js Single Executable blob:** `node --experimental-sea-config build-script/sea-config.json` (0.04s)
- Test built application: `node build/build.cjs --version`
- The build process is extremely fast (sub-second for main compilation)

### Docker Build (WARNING: SLOW)

- Build Docker image: `docker build -t butler-test .` — takes 10+ minutes due to npm install in container. NEVER CANCEL. Set timeout to 20+ minutes.
- Docker builds work but are significantly slower than local development
- For development, prefer local Node.js setup over Docker

## Architecture

- **Entry point:** `src/butler.js` → initializes globals → delegates to `src/app.js` (Fastify)
- **Globals singleton:** `src/globals.js` — config, logger, shared state (always use `globals.logger`, never `console.log`)
- **Config:** YAML via `config` package, template at `src/config/production_template.yaml`
- **ESM only** (`"type": "module"`) — use `import`/`export`, all test files must use `--experimental-vm-modules`
- **UDP server:** `src/udp/udp_handlers.js` — task events from Qlik Sense schedulers
- **REST API:** serves on port 8081 when using test config, API docs at http://localhost:8081/documentation
- **Test API endpoint:** `curl localhost:8081/v4/butlerping` returns `{"response":"Butler reporting for duty","butlerVersion":"13.1.2"}`

## Directory Structure

```
src/
├── butler.js          # Main application entry point
├── app.js            # Core application setup
├── config/           # Configuration templates and files
├── test/             # Jest test suite
├── lib/              # Utility libraries
├── api/              # API route handlers
├── routes/           # Express route definitions
└── globals.js        # Global configuration and state

build-script/         # Build automation scripts
docs/                # Documentation
static/              # Static web assets
```

## Conventions

- **JSDoc enforced** — ESLint with `eslint-plugin-jsdoc`
- **Logging:** always `globals.logger`, never `console.log`
- **Config-driven** — prefer YAML config over env vars
- **Dependencies:** `npm ci --include=prod` for Docker/SEA builds
- **Conventional Commits:** use format `type(scope)!: short, imperative summary`
  - Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `perf`, `test`, `build`, `ci`, `style`, `revert`
  - Scopes: `api`, `routes`, `config`, `lib`, `docs`, `build`, `deps`, `k8s`, `docker`, `tests`
  - Rules: present/imperative mood, lowercase summary, no trailing period, mark breaking changes with `!` and `BREAKING CHANGE:` footer
  - Examples: `feat(api): add /v4/systeminfo endpoint`, `fix(config): handle missing smtp.auth.user`

## Testing Quirks

- **Jest + ESM:** all test runs need `node --experimental-vm-modules --no-warnings node_modules/jest/bin/jest.js`
- **Test separation:** unit tests (`*.test.js`) vs integration tests (`*.api.test.js` in `routes/rest_server/__tests__/`)
- **Unit tests:** `npm run test:unit` — mocks all external dependencies, tests individual modules
- **Integration tests:** `npm run test:integration` — spins up Fastify server, tests REST API endpoints
- **Single test:** `node --experimental-vm-modules --no-warnings node_modules/jest/bin/jest.js src/udp/__tests__/udp_handlers.test.js --verbose`
- **Mock pattern:** use `jest.unstable_mockModule()` (ESM mocks), never `jest.mock()`
- **Standalone test:** `src/test/config/systeminfo.test.js` validates configuration schema

## Validation

### Manual Testing Scenarios

ALWAYS test API functionality after making changes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ptarmiganlabs/butler](https://github.com/ptarmiganlabs/butler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
