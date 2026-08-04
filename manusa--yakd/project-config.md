---
trigger: always_on
description: YAKD (Yet Another Kubernetes Dashboard) is a Quarkus (Java 25) + React web UI for Kubernetes and OpenShift clusters. Frontend lives under `src/main/frontend/`, gets built into the backend JAR under `frontend/` (see `pom.xml:120-123`). This file documents what an old hand would tell a new contributor on day one — read this first; fall back to grep only when something here doesn't match what you see.
---

# YAKD — AI Agents Instructions

YAKD (Yet Another Kubernetes Dashboard) is a Quarkus (Java 25) + React web UI for Kubernetes and OpenShift clusters. Frontend lives under `src/main/frontend/`, gets built into the backend JAR under `frontend/` (see `pom.xml:120-123`). This file documents what an old hand would tell a new contributor on day one — read this first; fall back to grep only when something here doesn't match what you see.

## Canonical commands

Use the `Makefile`. CI runs these exact targets (`.github/workflows/build-and-test.yaml`), so what works for you matches what gates a PR.

| Intent                                | Command                  |
|---------------------------------------|--------------------------|
| Fast "still wired together" check     | `make quick-build`       |
| Full build (frontend + backend)       | `make build`             |
| Test the Quarkus app — all of it (local)    | `make test-app`    |
| Java unit tests only (CI's `unit-tests` job) | `make test-unit`  |
| Selenium UI tests only (CI's `it-tests` job) | `make test-it`    |
| Full local CI mirror — am I ready to push? | `make check`         |
| Backend dev loop (hot reload, :8080)  | `make dev-backend`       |
| Frontend dev loop (Vite, :3000)       | `make dev-frontend`      |
| Deploy to local Minikube              | `make deploy-minikube`   |

`make help` lists every target. Run `dev-backend` and `dev-frontend` side-by-side — the Vite proxy assumes the backend is up.

## Architecture

Feature-per-folder, not layer-per-folder. Each Kubernetes resource type owns:
- a backend package: `src/main/java/com/marcnuri/yakd/<resource>/` (`<Resource>Resource.java` for JAX-RS, `<Resource>Service.java` for Fabric8 calls)
- a frontend directory: `src/main/frontend/src/<resource>/` (List, Detail, Edit components + `api.js` + `selectors.js`)

`ApiResource.java` is the JAX-RS root; it delegates each `/api/v1/<resource>` to the relevant `*Resource.java`. WebSocket entry points: `PodExecEndpoint.java` (terminal exec) and `WatchResource.java` (resource watches).

Redux state lives in `src/main/frontend/src/redux/` with per-resource slices. **This project uses plain Redux** (manual `reducer.js`/`actions.js`, action types like `CRUD_ADD_OR_REPLACE`), **not Redux Toolkit** — don't reach for `createSlice`/`createAsyncThunk`.

**To add a new resource type**, touch all of: the backend package (`<Resource>Resource.java` + `<Resource>Service.java`); the `@Path` delegation in `ApiResource.java`; a frontend feature directory with `index.jsx` + `api.js` + `selectors.js` + List/Detail/Edit components; a slice in `src/main/frontend/src/redux/`; and the route in `src/main/frontend/src/router.jsx`.

### Barrel exports (enforced)

Every frontend feature directory has an `index.js`/`index.jsx` that re-exports its public surface. **Production code** MUST import through the barrel; tests (`__test__/*.test.{js,jsx}`) import sibling files directly.

```javascript
// Correct (production)
import {PodsList, api, selectors} from '../pods';
import {Card, Icon} from '../components';

// Wrong (production) — bypasses the barrel
import {PodsList} from '../pods/List';
import {Card} from '../components/Card';
```

Barrel conventions:
- `export * as api from './api'` (namespaced)
- `export * as selectors from './selectors'` (namespaced)
- `export {PodsDetailPage} from './PodsDetailPage'` (named)
- Rename generic component names semantically: `export {List as PodsList} from './List'`

## Code style

**Java.** Java 25. Apache 2.0 header required on every source file (the license-check script gates this; see Gotchas).

**JavaScript/React.**
- Prettier config (in `package.json:70-77`): `singleQuote`, `jsxSingleQuote`, `arrowParens: avoid`, `bracketSpacing: false`, `trailingComma: none`. `make fmt` runs it (mutating — see Gotchas).
- ESLint with `eslint-plugin-react-hooks`; `make lint` runs ESLint + the license-header check.
- Tests live in `__test__/` directories named `*.test.{js,jsx}`.
- **Type checking via JSDoc + `// @ts-check`** — see Gotchas for how this actually works (`checkJs: false` in jsconfig.json means the directive is the opt-in, not a no-op).

## Testing

**MOCKS ARE STRICTLY FORBIDDEN.** Black-box testing only — verifies behavior, survives refactors. If you think you need a mock, ask first. Background: <https://blog.marcnuri.com/blackbox-whitebox-testing-comparison>.

Test the public API. One assertion per test. Group related cases with nested `describe` / `beforeXxx`. Descriptive names ("renders a slashed wifi icon", not "test1").

### Preferred: Selenium IT against Fabric8 MockServer

Full-stack `*IT.java` tests in `src/test/java/` validate the rendered UI against a mocked Kubernetes API. No real cluster. Highest confidence per minute of test runtime — **this is the preferred test type for full features**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manusa/yakd](https://github.com/manusa/yakd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
