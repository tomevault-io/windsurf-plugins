---
trigger: always_on
description: Guidance for future Claude sessions on this repo. Read this first before making changes.
---

# kube-argus — Claude working notes

Guidance for future Claude sessions on this repo. Read this first before making changes.

## What this is

Real-time Kubernetes dashboard shipped as a single Go binary + Helm chart. In-cluster deployment, OIDC / Google SSO auth, JIT exec workflow, Slack + generic webhook notifications, cost/spot advisor, AI-diagnose via LLM gateway, audit trail persisted to ConfigMaps. Maintained by @manishchaudhary101. Public repo: `github.com/manishchaudhary101/kube-argus`. ArtifactHub: `artifacthub.io/packages/helm/kube-argus/kube-argus`. Landing page: `manishchaudhary101.github.io/kube-argus`.

## Architecture (post-v1.2.8 reorganisation)

```
cmd/server/main.go              — 238-line wire-up ONLY (env → clients → DI → mux → ListenAndServe)

internal/
├── httpx/                      — response helpers (JSON, Error, K8sError, JSONGz, WsUpgrader, WsCheckOrigin)
├── auth/                       — sessions, OIDC, Google SSO, RequireAdmin, IsAdmin, ClientIP, middleware
├── audit/                      — trail, dedup, retention, ConfigMap persistence, WebSocket presence broadcast
├── jit/                        — JIT store, approve/deny/revoke/expiry, ConfigMap persistence
├── notify/                     — Slack + generic webhook, HMAC signing, event filtering
└── api/                        — every HTTP handler + cache + informers + Prometheus + spot advisor
                                  + AI diagnose + Helm SDK + CRD browser (dynamic client)
                                  Init(clientset, metricsCl, restConfig, clusterName) wires globals.

web/src/
├── main.tsx                    — QueryClientProvider (TanStack Query) wraps App
├── App.tsx                     — router, sidebar, tab switching, WebSocket presence, JIT poll
├── hooks/
│   ├── useFetch.tsx            — original SWR-style fetch hook (still used by most views)
│   └── useApi.ts               — TanStack Query drop-in wrapper (POC; rename useFetch→useApi to migrate)
└── components/
    ├── views/                  — top-level tab views (Overview, Pods, Workloads, CRDs, etc.)
    └── modals/                 — YAML, HelmRelease, Info popover, Settings, Audit, JITRequests, etc.
```

Cross-package callbacks use package-level `var Foo = func(){}` hooks in `notify`, `auth`, `jit`; `main.go` wires them at startup. Never remove or add these without updating `main.go` in the same commit.

Globals in `internal/api/`: `clientset`, `metricsCl`, `restCfg`, `dynamicClient`, `apiextClient`, `cache`. Handlers can assume these are set — `main.go`'s init order guarantees it before `mux` is served.

## The rule: test after every backend change

**Every time you edit `.go` under `internal/` or `cmd/`, run before declaring the task done:**

```bash
cd /Users/manish/Desktop/kube-argus
go build ./... && go vet ./... && go test ./...
```

**Every time you edit `.tsx` or `.ts` under `web/src/`, run before declaring the task done:**

```bash
cd /Users/manish/Desktop/kube-argus/web && npx tsc -b
```

Non-negotiable. A `.claude/settings.local.json` `Stop` hook enforces this — the harness will refuse to end your turn until backend/frontend gates pass on modified files.

If tests fail, do NOT declare done or hand back to the user — fix the failure first.

## Common commands

```bash
# Backend
go build ./...
go vet ./...
go test ./...                              # 24 tests, ~3s
go run ./cmd/server                        # localhost:8080, uses ~/.kube/config

# Frontend
cd web && npm run dev                      # localhost:5173, proxies /api → :8080
cd web && npx tsc -b                       # typecheck
cd web && npm run build                    # production bundle

# Run everything locally (two terminals)
# Terminal 1: aws sso login && go run ./cmd/server
# Terminal 2: cd web && npm run dev
# Open http://localhost:5173/

# Stop servers
lsof -ti:8080 -ti:5173 | xargs kill 2>/dev/null
```

## Coding conventions

**Go:**
- Handlers: `func apiSomething(w http.ResponseWriter, r *http.Request)`. Use `httpx.JSON`, `httpx.Error`, `httpx.K8sError` for responses.
- Admin gate: `if !auth.RequireAdmin(w, r) { return }`. For role-branching without writing 403 use `auth.IsAdmin(r)`.
- Cross-package call: assign to a `var Foo = func(){}` hook in the callee package, wire in `main.go`.
- Audit any admin action or sensitive read: `audit.Record(email, role, action, resource, detail, ip)`. Grab email/role from `r.Context().Value(auth.UserCtxKey).(*auth.SessionData)`.
- New route: register in `internal/api/routes.go` under the right section.
- Multi-verb / sub-path routes: use a dispatcher function (see `helmDispatch`, `apiCronJobRouter`).

**Frontend:**
- Data fetching: prefer `useApi` from `hooks/useApi.ts` (TanStack Query). Legacy `useFetch` still works; migrate incrementally by renaming the import.
- URL state for anything a user should be able to bookmark or navigate back to. Wire popstate in `useEffect` for browser back.
- Tailwind + existing design tokens (`hull-*` for surfaces, `neon-*` for status accents). Don't introduce new palettes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manishchaudhary101/kube-argus](https://github.com/manishchaudhary101/kube-argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
