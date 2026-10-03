---
trigger: always_on
description: Node service that keeps a Bambu Lab AMS in sync with a [Spoolman](https://github.com/Donkie/Spoolman)
---

# HaspelSync

Node service that keeps a Bambu Lab AMS in sync with a [Spoolman](https://github.com/Donkie/Spoolman)
instance. It listens to the printer's MQTT report topic, mirrors the loaded
spools into Spoolman (create, merge, weight update), tracks filament
consumption per print, and serves a small web UI for the parts that need a
human decision.

Runs as a single container. No database: all persistent state is Spoolman plus
six JSON files under `printers/`.

## Intent Layer

**Before modifying code in a subdirectory, read its AGENTS.md first** to
understand local patterns and invariants.

- **Backend modules**: [`src/AGENTS.md`](src/AGENTS.md), covering MQTT ingest,
  AMS matching logic, the Spoolman client, G-code consumption and HTTP/SSE routes

`public/` (frontend) and `test/` have no node of their own; the rules that
matter for them are below.

## Layout

| Path | Role |
|---|---|
| `entrypoint.js` | Container entrypoint and supervisor. Forks `starting.js`, starts it again on the restart exit code, passes every other code on, forwards SIGTERM/SIGINT and waits for the child. `SUPERVISOR=false` runs the service in this process instead. Never put application logic here. |
| `starting.js` | The service process. Global error handlers, signal handling, then dynamic-imports `backend.js`. Forked by `entrypoint.js`, so the handlers live where the application does. |
| `backend.js` | Express app, the request guard from `src/security.js` and the password gate from `src/auth.js` in front of everything, static hosting, startup sequence: Spoolman health, the `tag` extra-field bootstrap, printer log files, monitor loops. |
| `src/` | All backend logic. See its AGENTS.md. |
| `public/` | Vanilla JS/HTML/CSS frontend. No build step, no framework, no bundler; files are served as-is. `favicon.svg`, `favicon.ico` and `apple-touch-icon.png` are the tab and home screen icon every page links in its `<head>`; they are listed in `PUBLIC_PATHS` in `src/auth.js` because the login tab asks for them before anybody is logged in. `login.html` and `login.js` are the exception to everything below: they are served before anybody is logged in and therefore import nothing from the rest of the UI. `menu.js` renders the whole menu bar for every page, the dark mode button included, so each page includes it before its own script and provides an empty `#menu-root` in its `#menubar`. It also fills the printer picker that lives in the page's own headline, `#printer-name` on the dashboard and `#headline` on the log viewer: the bar carries navigation and the session, the page carries what it is showing. `shared.js` holds the pure decisions the frontend and the server both make, including the slot options, the active print states and the external slot label; it lives here because a browser has to be able to load it unbuilt, and `src/` imports it from here. `ui.js` holds what every page does with the API and with a string, `fetchJson()` and `escapeHtml()`, which is browser only and therefore not in `shared.js`. `frontend.js`, `settings.js` and `api.js` are modules and import both; `menu.js` and `export.js` are classic scripts read off the global scope. Every visible string goes through `t("key")` from `i18n.js`, a classic script every page loads first together with one table per language under `i18n/` (`en.js`, `de.js`): English is the default and the fallback for a missing key, the language field on the settings page picks another language per browser, and another language is one more table dropped into `i18n/`: every page loads all of them through `i18n/all.js`, which `src/languages.js` puts together from the folder on each request. Its name in that field comes from a `language.<code>` key in the other tables, else from the name it registers itself with. `test/i18n.test.js` holds every table to the English keys and placeholders. Log lines, the content of the API page and every value the API hands out stay English; a failed answer carries a `code` the UI translates as `error.<code>`. `api.html` and `api.js` are the API page, reached from the API keys on the settings page rather than from the bar: they render whatever `/api/openapi.json` describes and know no route by name, so a route is added to `src/openapi.js` and never to the page. |
| `test/` | `node:test` suites (`npm test`). Fixtures in `test/fixtures/` are real slicer output, not synthetic. `test/fixtures/reports/` holds real printer reports copied from ha-bambulab under its MIT notice, see the README there and `THIRD_PARTY_NOTICES.md`; `test/reports.test.js` runs every one of them through the ingest pipeline and lists what they found that is not handled yet as `todo`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rdiger-36/HaspelSync](https://github.com/Rdiger-36/HaspelSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
