---
trigger: always_on
description: This is `uapi-json`, an old-style Node.js CommonJS package that wraps the Travelport Universal API. It exposes JSON/Promise service factories over SOAP calls for Air, Hotels, Utils, and Terminal workflows.
---

# AGENTS.md

## Project Overview

This is `uapi-json`, an old-style Node.js CommonJS package that wraps the Travelport Universal API. It exposes JSON/Promise service factories over SOAP calls for Air, Hotels, Utils, and Terminal workflows.

The public package entry is `index.js`, which re-exports `src/index.js`. `src/index.js` exports:

- `createAirService(settings)`
- `createHotelService(settings)`
- `createUtilsService(settings)`
- `createTerminalService(settings)`
- grouped error classes under `errors`
- `errorCodes`

Keep the implementation style conservative. This project uses `require`, `module.exports`, plain JavaScript objects, and Promise/async functions without a build step or transpiler.

## Commands

- Install dependencies: `npm install`
- Run tests: `npm test`
- Run coverage: `npm run cover`
- Run lint: `npm run lint`
- Apply lint fixes: `npm run fix`

The package manager is npm. Keep `package-lock.json` in sync when dependencies change. There is no `.nvmrc`; `.travis.yml` still documents the historical Node.js 8 CI target, while the current lockfile uses modern npm lockfile format.

## Repository Layout

- `index.js` - root package entry, delegates to `src/`.
- `src/index.js` - public API aggregation for service factories and error exports.
- `src/config.js` - Travelport endpoint URL construction by region and production/pre-production mode.
- `src/Request/` - shared SOAP request stack:
  - `uapi-request.js` validates inputs, compiles Handlebars XML, sends `axios` POST requests, parses SOAP responses, maps SOAP faults, and calls service-specific parsers.
  - `prepare-request.js` injects auth-derived fields into template params (`TargetBranch`, `Username`, `provider`, `emulatePcc`).
  - `uapi-parser.js` wraps `xml2js` and contains the generic recursive XML-to-JSON collapse/keying logic.
  - `default-config.js` and `errors-config.js` tune parser behavior for normal responses and SOAP faults.
  - `RequestErrors.js` defines request-layer validation/runtime/SOAP errors.
- `src/Services/` - service-specific public wrappers, endpoint mappings, parsers, validators, errors, and SOAP templates.
- `src/Services/Air/` - the largest service. `Air.js` is the high-level public wrapper; `AirService.js` maps individual SOAP operations through `uApiRequest`; `AirValidator.js`, `AirParser.js`, `AirFormat.js`, `AirErrors.js`, `validators/`, `transformers/`, and `templates/` provide the request and response details.
- `src/Services/Hotels/` - hotel search/rate/book/cancel behavior, with service mapping, validators, parser, errors, amenities lookup, and templates.
- `src/Services/Terminal/` - terminal session/command/close behavior. `Terminal.js` manages session state, auto-close hooks, `MD` pagination, retries, emulation, and terminal-specific runtime errors.
- `src/Services/Utils/` - currency conversion and reference-data requests.
- `src/utils/` - shared helpers for validation/transform composition, price/name formatting, parser helpers, booking extraction, promise inflate/deflate helpers, and other small utilities.
- `src/error-types.js`, `src/error-codes.js`, `src/node-errors-helpers.js` - shared error hierarchy and helper wiring.
- `src/reference-data-types.js` - exported reference-data constants.
- `docs/` - user-facing Markdown API docs for Air, Hotels, Terminal, Utils, and reference data types.
- `examples/` - runnable usage examples grouped by service.
- `test/` - Mocha/Chai/Sinon/Proxyquire tests.
- `test/FakeResponses/` and `test/Terminal/*Responses/` - SOAP XML, JSON, and terminal-screen fixtures used by parser and workflow tests.

## Request Flow

Most service calls follow this shape:

1. A public factory in `src/index.js` creates a service from `settings`.
2. A high-level service wrapper such as `src/Services/Air/Air.js` may add orchestration, retries, fallbacks, deprecation wrappers, or terminal calls.
3. A low-level service mapping such as `AirService.js`, `HotelsService.js`, `UtilsService.js`, or `TerminalService.js` wires an operation to `uApiRequest(serviceUrl, auth, template, rootObject, validator, errorHandler, parser, debug, options)`.
4. The validator is usually a `compose(validate(...), transform(...))` pipeline.
5. The template is a JS module exporting a Handlebars XML string.
6. `src/Request/uapi-request.js` compiles the template, posts SOAP XML with `axios`, parses XML through `uapi-parser.js`, handles SOAP faults, and invokes the service parser with parser state as `this`.

When adding or changing a SOAP operation, update all relevant layers: public wrapper if needed, low-level service mapping, validator/transformer, template export, parser/error handling, tests, docs, and examples.

## Service Conventions

- Use CommonJS (`const x = require(...)`, `module.exports = ...`).
- Service settings are validated with `src/utils/validate-service-settings.js` where existing services do so. Required auth fields are `username`, `password`, and `targetBranch`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Travelport-Ukraine/uapi-json](https://github.com/Travelport-Ukraine/uapi-json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
