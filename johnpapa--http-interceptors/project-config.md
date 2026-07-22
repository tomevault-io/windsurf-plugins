---
trigger: always_on
description: A comparison monorepo demonstrating HTTP interceptors in **Angular** (HttpClient interceptors) and **Svelte** (Axios interceptors). Same "Tour of Heroes, Movie Edition" app implemented in both frameworks with a `json-server` mock backend.
---

# HTTP Interceptors — Agent Guide

A comparison monorepo demonstrating HTTP interceptors in **Angular** (HttpClient interceptors) and **Svelte** (Axios interceptors). Same "Tour of Heroes, Movie Edition" app implemented in both frameworks with a `json-server` mock backend.

## Repository Structure

```
http-interceptors/
├── angular-app/                    # Angular 14 app — HttpClient interceptors
│   ├── src/app/
│   │   ├── core/
│   │   │   ├── interceptors/       # Auth, busy, CSRF, SSL, logging, read-only, transform
│   │   │   ├── components/         # Nav, header, sign-in, auth-failed, not-found
│   │   │   ├── session.service.ts  # Session/token management
│   │   │   └── model/              # Hero, Villain, Movie interfaces
│   │   ├── heroes/                 # Hero CRUD feature module
│   │   ├── villains/               # Villain CRUD feature module
│   │   ├── store/                  # NgRx store (entity metadata, config)
│   │   └── shared/                 # Shared components
│   ├── proxy.conf.json             # Dev proxy → json-server backend
│   ├── db.json                     # Mock database (json-server)
│   └── package.json                # Angular CLI, NgRx, json-server-auth
│
├── angular-http-hard-way/          # Angular 14 app — manual HttpClient (no NgRx)
│   ├── src/app/
│   │   ├── core/interceptors/      # Same interceptor set, different wiring
│   │   ├── heroes/                 # Hero CRUD (manual HTTP, no NgRx)
│   │   └── shared/
│   ├── proxy.conf.json
│   ├── db.json
│   └── package.json
│
├── svelte-app/                     # Svelte + Vite app — Axios interceptors
│   ├── src/
│   │   ├── interceptors/           # Same interceptor concepts via Axios
│   │   ├── heroes/                 # Hero CRUD (Svelte components)
│   │   ├── villains/               # Villain CRUD
│   │   ├── store/                  # Svelte stores + data services
│   │   ├── models/                 # TypeScript interfaces
│   │   └── components/             # Shared UI components
│   ├── db.json
│   └── package.json                # Vite, Svelte, Axios, json-server-auth
│
├── .vscode/                        # VS Code workspace settings
├── CONTRIBUTING.md                 # Contribution guidelines
├── CODE_OF_CONDUCT.md              # Community standards
├── README.md                       # Project overview, setup instructions
└── LICENSE                         # MIT
```

## Tech Stack

| Area | Angular apps | Svelte app |
|------|-------------|------------|
| **Framework** | Angular 14 | Svelte + Vite |
| **HTTP client** | `@angular/common/http` (HttpClient) | Axios |
| **Interceptors** | `HttpInterceptor` interface (class-based) | Axios interceptors (function-based) |
| **State management** | NgRx (`angular-app`), manual services (`angular-http-hard-way`) | Svelte stores |
| **Styling** | Bulma + SCSS | Bulma + SCSS |
| **Mock API** | `json-server-auth` | `json-server-auth` |
| **Language** | TypeScript | TypeScript |

## Interceptor Catalog

Both Angular and Svelte apps implement the **same 8 interceptors** — this is the core comparison:

| Interceptor | Purpose | Angular | Svelte |
|-------------|---------|---------|--------|
| `auth` | Attach Bearer token, handle 401s | `AuthInterceptor` class | `authInterceptor()` function |
| `busy` | Track in-flight requests for loading UI | `BusyInterceptor` | `busyInterceptor()` |
| `csrf` | Add CSRF token header | `CsrfInterceptor` | `csrfInterceptor()` |
| `ensure-ssl` | Rewrite HTTP → HTTPS | `EnsureSSLInterceptor` | `ensureSSLInterceptor()` |
| `log-headers` | Log request/response headers | `LogHeadersInterceptor` | `logHeadersInterceptor()` |
| `log-http` | Log HTTP traffic | `LogHttpInterceptor` | `logHttpInterceptor()` |
| `read-only` | Block POST/PUT/DELETE for unauthorized users | `ReadOnlyInterceptor` | `readOnlyInterceptor()` |
| `transform` | Transform response shape | `TransformInterceptor` | `transformInterceptor()` |

**Key difference**: Angular interceptors are classes implementing `HttpInterceptor` and registered via DI providers. Svelte interceptors are plain functions that call `axios.interceptors.request.use()` / `axios.interceptors.response.use()`.

## Build & Run

Each app is independent — there is **no monorepo workspace config**. Each has its own `node_modules` and runs separately.

### Angular apps (`angular-app` or `angular-http-hard-way`)

```bash
cd angular-app       # or angular-http-hard-way
npm install
npm run full-stack   # starts json-server backend + Angular dev server
```

- `angular-app` runs on port **4202** (backend: 4302)
- `angular-http-hard-way` runs on port **4201** (backend: 4301)

### Svelte app

```bash
cd svelte-app
npm install
npm run full-stack   # starts json-server backend + Vite dev server
```

- Svelte app runs on port **5173** (backend: 5001)

### Mock API

All apps use `json-server-auth` with a `db.json` file. The mock API provides:
- CRUD endpoints for heroes, villains, and movies
- JWT authentication (register/login at `/register`, `/login`)
- Route-based access control via `routes.json`

## Testing

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpapa/http-interceptors](https://github.com/johnpapa/http-interceptors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
