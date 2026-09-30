---
trigger: always_on
description: Sampo-UI is a framework for building faceted-search portals over SPARQL endpoints. It consists of a React client, an Express server, and a JSON/JS config layer that defines the entire portal without touching the framework code.
---

# AGENTS.md — sampo-ui

Sampo-UI is a framework for building faceted-search portals over SPARQL endpoints. It consists of a React client, an Express server, and a JSON/JS config layer that defines the entire portal without touching the framework code.

---

## Architecture Overview

```
Browser
  └─ React client  (dev: port 8080 | prod: port 80)
        │  fetches config on startup
        │  sends facet/result queries
        ▼
Express server  (port 3001)
        │  loads config at startup
        │  builds & executes SPARQL queries
        ▼
SPARQL endpoint (e.g. Ontop, Fuseki)
        ▼
RDF / Virtual Knowledge Graph
```

The server is stateful: it loads all configs at startup and holds the resolved `backendSearchConfig` object for the lifetime of the process. The client is stateless: it re-fetches config files and results via HTTP.

In production the client and server are typically bundled into a single container (see [Combo Build](#combo-build) below).

---

## Client

### Entry point & startup

`client/src/index.js` bootstraps the app:
1. Calls `useConfigsStore.getState().initConfigs()` to fetch `portalConfig.json` from the server and then each perspective's JSON config.
2. Initializes the Redux store via `configureStore()` with redux-observable epic middleware.
3. Renders the app once configs are loaded.

Configs are stored in a **Zustand store** (`client/src/stores/configsStore.js`). The store exposes `getStaticFileUrl(path)` and `getConfigJsonFile(file)` helpers used by components and custom components alike.

### Routing & perspectives

`SemanticPortal.js` reads `perspectiveConfigs` from the Zustand store and dynamically creates React Router routes:

| URL pattern | Renders |
|---|---|
| `/:locale/:perspectiveID/faceted-search` | Faceted search page |
| `/:locale/:perspectiveID/page/:id` | Instance (detail) page |
| `/:locale/full-text-search` | Full-text search |

### Redux store

One set of reducers is created **per perspective** at startup:

- `state[perspectiveID]` — result data, pagination, sort
- `state[${perspectiveID}Facets]` — facet values, active filters, fetch state

Key action types (defined in `client/src/actions/index.js`):
- `FETCH_RESULTS`, `FETCH_PAGINATED_RESULTS`, `UPDATE_RESULTS`
- `FETCH_FACET`, `UPDATE_FACET_VALUES`, `UPDATE_FACET_OPTION`, `CLEAR_FACET`
- `FETCH_RESULT_COUNT`

### Facet interaction flow

When a user selects a facet value:

1. **UI component** (e.g. `HierarchicalFacet`) calls `dispatch(updateFacetOption({facetClass, facetID, option, value}))`.
2. **Facets reducer** updates `state[perspectiveIDFacets].facets[facetID][filterType]` and increments `facetUpdateID`.
3. **Epics** (`client/src/epics/index.js`, RxJS + redux-observable) watch for facet/result actions and fire API requests:
   - `POST /api/v1/faceted-search/{facetClass}/facet/{facetID}` — refreshes facet value counts
   - `POST /api/v1/faceted-search/{resultClass}/all` — fetches updated results
4. **Constraint serialization** (`client/src/helpers/helpers.js` — `stateToUrl()`) reads the current facet Redux state and builds a `constraints` array included in the request body.

### Result rendering

`ResultClassRoute.js` renders the correct component based on `resultClassConfig.component`:

- Built-ins: `ResultTable`, `LeafletMap`, `Deck`, `ApexCharts`, `Network`, `InstancePageTable`, `Export`, …
- `CustomComponent` — dynamically loaded from `/custom-components/{componentName}.js`

`FacetBar.js` does the same for facet components based on `facet.filterType`:

- Built-ins: `HierarchicalFacet`, `TextFacet`, `SliderFacet`, `RangeFacet`, `DatasetSelector`, …
- `CustomComponent` — same dynamic loading mechanism

### Custom components

Custom components are JavaScript bundles built with webpack and served at `/custom-components/{ComponentName}.js`. They must register themselves on `window.__customComponents[ComponentName]`.

**Loading**: `client/src/helpers/loadCustomComponent.js` injects a `<script>` tag and polls `window.__customComponents[name]` until the module is available.

**Shared libraries**: The main app exposes core dependencies on `window.__sharedLibraries` so custom components don't bundle their own copies:

```js
const { MUI, MuiIcons, intl, reactRedux, configsStore, helpers } = window.__sharedLibraries
```

Available libraries include: Material-UI, MUI Icons, react-intl-universal, react-redux, react-router-dom, lodash, PropTypes, query-string, and the Zustand configsStore.

**Props passed to a custom result component**: `results`, `fetching`, `resultClass`, `facetClass`, `facetState`, `facetUpdateID`, `resultClassConfig`, `perspectiveConfig`, `portalConfig`, `updateFacetOption`, `fetchResults`, `fetchPaginatedResults`, `showError`, and more.

**Props passed to a custom facet component**: `facetID`, `facetClass`, `facet`, `facetState`, `facetUpdateID`, `updatedFacet`, `updatedFilter`, `updateFacetOption`, `clearFacet`, `fetchFacet`, `perspectiveConfig`, `portalConfig`, and more.

---

## Server

### API routes

All routes are prefixed `/api/v1` (`server/src/index.js`):

| Endpoint | Method | Purpose |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SemanticComputing/sampo-ui](https://github.com/SemanticComputing/sampo-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
