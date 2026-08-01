---
trigger: always_on
description: Project development guidelines (advanced)
---

Project development guidelines (advanced)

Documentation
- **Before starting work on any feature or component, always check the docs/ directory first**
  - The docs/ folder contains comprehensive technical specifications and architecture documentation
  - Each major feature has its own documentation file (e.g., docs/vault.md for the Vault system)
  - Documentation includes:
    - Database schemas and data models
    - Business logic and workflows
    - API specifications and integration points
    - Constraints, rules, and edge cases
  - Reading the relevant documentation before implementation helps avoid mistakes and ensures consistency with the existing architecture
  - If documentation is missing or outdated for a feature you're working on, consider updating it as part of your work
- **After completing any task, always update the relevant documentation in docs/ directory**
  - When adding new methods, functions, or services, document their signatures, parameters, algorithms, and error handling
  - When modifying existing functionality, update the corresponding documentation to reflect the changes
  - When adding new database tables or models, update the schema documentation and model descriptions
  - Documentation updates are a mandatory part of task completion, not an optional step

Build and configuration
- Toolchain versions
  - Go: go 1.24.x (module file declares 1.24.5). The project is a single Go module: github.com/webtor-io/web-ui.
  - Node: Node.js 22.x is used to build frontend assets (see Dockerfile stage "node:22").
  - Package managers: npm (package-lock.json is present). Yarn is not used.

- Build paths and outputs
  - Frontend assets are emitted to assets/dist by webpack (see webpack.config.js). Static assets are exposed by the server at /assets (handlers/static).
  - Public static files from pub are mounted at / and /pub.
  - Go server binary: web-ui during local go build; Docker build stage outputs /app/server for Alpine image.

- Makefile targets
  - make build: runs npm run build then go build .
  - make run: runs the locally built binary ./web-ui s (serve).
  - make forward-ports: uses kubefwd to forward selected services in the webtor namespace (claims-provider, supertokens, rest-api, abuse-store) for local development.

- Docker build
  - docker build . produces a minimal Alpine image. The build performs:
    1) npm install && npm run build in a Node 22 builder;
    2) go build in a golang:latest builder with CGO_ENABLED=0, GOOS=linux and additional -ldflags "-w -s -X google.golang.org/protobuf/reflect/protoregistry.conflictPolicy=ignore" to relax protobuf symbol conflicts;
    3) runtime image copies server, templates, migrations, pub, and built assets.
  - The container exposes ports 8080 and 8081 and runs ./server serve with GIN_MODE=release.

- Local development (hot reload)
  - npm start runs two processes with concurrently: "air" (Go live-reload) and webpack-dev-server.
  - air is expected to be installed on the developer machine. If missing, install via: go install github.com/cosmtrek/air@latest, then ensure GOPATH/bin is on PATH. An explicit .air.* config is not present; defaults are sufficient for this repo structure.

- Minimum runtime configuration
  - Web listener: WEB_HOST (default ""), WEB_PORT (default 8080). Flags: --host, --port (services/web).
  - REST API access (one of):
    - Direct Webtor REST-API: REST_API_SERVICE_HOST, REST_API_SERVICE_PORT (default 80), REST_API_SECURE (enable https), WEBTOR_API_KEY/WEBTOR_API_SECRET (if required by backend). Flags in services/api.
    - RapidAPI: RAPIDAPI_HOST and RAPIDAPI_KEY; when set, API calls are redirected via RapidAPI and HTTPS:443 is enforced.
  - Sessions: SESSION_SECRET (default "secret123"). Optional Redis-backed session store if REDIS_MASTER_SERVICE_HOST/REDIS_SERVICE_HOST and REDIS_MASTER_SERVICE_PORT/REDIS_SERVICE_PORT are provided; REDIS_PASS for password (handlers/session).
  - Static assets: ASSETS_PATH (default ./assets/dist), WEB_ASSETS_HOST (handlers/static). For production, ensure assets/dist exists (via npm run build).
  - Optional integrations (feature toggles and endpoints):
    - Umami analytics: USE_UMAMI, UMAMI_WEBSITE_ID, UMAMI_HOST_URL (services/umami).
    - GeoIP API cache: USE_GEOIP_API, GEOIP_API_SERVICE_HOST, GEOIP_API_SERVICE_PORT (services/geoip).
    - Claims-provider (user tiers/limits): USE_CLAIMS, CLAIMS_PROVIDER_SERVICE_HOST, CLAIMS_PROVIDER_SERVICE_PORT (services/claims). When enabled, claims are fetched via gRPC and cached with lazymap (1m success TTL, 10s error TTL).
    - Stremio addon HTTP client: STREMIO_ADDON_USER_AGENT (custom user agent for addon requests), STREMIO_ADDON_PROXY (proxy URL for addon HTTP client, supports http:// and socks5:// schemes). Flags: --stremio-addon-user-agent, --stremio-addon-proxy (services/stremio).

- Database and migrations
  - PostgreSQL configuration flags are registered by github.com/webtor-io/common-services (RegisterPGFlags). On server start (serve.go), migrations in migrations/ are applied automatically. Ensure DB connectivity is configured via the common services environment variables (see common-services docs; typical PG_HOST/PG_PORT/PG_USER/PG_PASSWORD/PG_DATABASE and SSL flags).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webtor-io/web-ui](https://github.com/webtor-io/web-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
