---
trigger: always_on
description: Conformance test suite for i3X 1.0 server implementations, built for the CESMII i3X working group. Community implementers use it via a hosted web page, a self-hosted web UI, or a CLI — all from this one package.
---

# i3X 1.0 Conformance Test Suite — project context

Conformance test suite for i3X 1.0 server implementations, built for the CESMII i3X working group. Community implementers use it via a hosted web page, a self-hosted web UI, or a CLI — all from this one package.

## Commands

```bash
npm start                                  # web UI on :8330 (same page as the hosted version)
npm run mock                               # compliant reference server on :8331 (/v1 prefix)
npm test                                   # self-test: suite vs mock in 7 configs, asserts each verdict
node bin/i3x-test.js run <endpoint> [...]  # CLI runner (see --help)
```

There is no build step, linter config, or test framework — plain Node, run directly.

## Hard invariants (do not change casually)

- **Zero runtime dependencies, Node ≥ 18.17.** This is deliberate so community members can run it with nothing but Node. Don't add packages; built-in `fetch`, `node:http`, `node:zlib` cover everything.
- **Verdict strings are working-group-approved wording** and are string-matched in `bin/i3x-test.js`, `server/public/index.html`, and `test/selftest.js`:
  - `Full 1.0 Compliance` (green)
  - `1.0 Compliance, Immature Type System` (green) — all passed but every ObjectType schema is a primitive scalar
  - `1.0 Compatible` (teal/cyan — deliberately not orange; orange is reserved for warnings)
  - `Not Compliant` (red)
- **MAY features omitted ⇒ never a failure** (verdict caps at "1.0 Compatible"). A *declared* capability that misbehaves ⇒ failure.
- **SHOULD-level findings are warnings only** and never affect the verdict (e.g. plain HTTP, accepting unauthenticated requests, 501-honesty). The working group explicitly does not want to block people who disable auth for testing.
- **Write tests are non-destructive**: they write each object's current value back to itself. UPD-02 read-after-write does NOT require equality — live data sources legitimately move between write and read; a mismatch is a pass with a "Note:" message.
- **CLI exit code**: 0 for any compliance/compatible verdict, 1 only for Not Compliant/Error (CI-friendly).

## Architecture

```
lib/engine.js        orchestrator: resolves base URL (auto-probes <url>/v1 as fallback),
                     runs phases sequentially, emits events, feature-gates MAY tests
lib/report.js        verdict computation + isRichTypeSystem()
lib/validators.js    mini JSON Schema validator + SHAPES registry mirroring the OpenAPI
                     components + envelope/VQT semantic checks
lib/client.js        fetch wrapper (auth, timeout); rawGzipProbe uses node:http because
                     fetch hides Content-Encoding
lib/specrefs.js      REFS map: every test's `ref` key → Implementation Guide anchor URL
lib/version.js       single source of truth for SPEC_VERSION (tracks the spec) and
                     BUILD (bump when the tests themselves change). CLI prints both in
                     the banner/USAGE and stamps them into the JSON report; the static
                     UI reads them from GET /api/version and shows a header badge +
                     stamps them into the downloaded report.
lib/tests/*.js       test definitions per phase (transport, exploratory, query, update, subscribe)
bin/i3x-test.js      CLI (run / serve / mock)
server/serve.js      web server: static UI + POST /api/run → SSE event stream
server/public/       single-file UI (inline CSS/JS, no framework) + logo/favicon assets
mock/server.js       compliant reference server (demo pump-station model)
test/selftest.js     asserts the verdict for the mock configurations
```

### Test definition shape

```js
{ id: 'QRY-01', name: '…', level: 'MUST'|'MAY'|'SHOULD',
  feature: 'update.current',   // MAY only: dotted path into /info capabilities; engine
                               // auto-skips (reason 'omitted') when declared false
  ref: 'queryMethods',         // key into lib/specrefs.js REFS
  async run(ctx) { return pass/fail/warn/skip(message) } }   // helpers in lib/results.js
```

Tests run sequentially and share `ctx` (client, config, namespaces, objectTypes/typesById, relationshipTypes/relById, objects/objectsById, currentValues, subscriptionId/clientId, …). Missing prerequisites ⇒ `skip(…, 'blocked')`. Test IDs are referenced in the README table — keep numbering stable, append new ones.

### Display conventions

- `pass()` messages are normally hidden; a pass message starting with **`Note:`** is rendered (muted gray) in both CLI and web UI — used when a pass needs an honest caveat.
- Warnings are itemized in `summary.warnings` and repeated in the final CLI summary and web verdict banner.
- Web UI consumes `POST /api/run` as SSE (`fetch` + reader, since EventSource can't POST). Event types: `start`, `note`, `phase`, `test:start`, `test:end`, `done`.

## Normative sources & spec drift


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cesmii/i3X](https://github.com/cesmii/i3X) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
