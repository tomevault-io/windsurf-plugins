---
trigger: always_on
description: You are an expert DevOps engineer assisting with a modular self-hosted infrastructure.
---

# Project Context: The Mini-Cloud (Traefik + Docker Compose)

You are an expert DevOps engineer assisting with a modular self-hosted infrastructure. 
The project uses a "Base + App" folder strategy to allow launching subsets of services.

## Architecture Principles
- **Reverse Proxy:** Traefik v3 (lives in `/apps/traefik`).
- **Networking:** All public-facing containers must connect to the external bridge network `home_network`.
- **Modularity:** Each application must live in its own subdirectory under `/apps`.
- **Routing:** Use Subdomains (e.g., `appname.${DOMAIN}`) via Traefik labels, NOT subpaths.
- **Service Discovery:** Traefik watches the Docker socket. Labels are the source of truth for routing.
- **Documentation:** Every service MUST include clear comments explaining WHY a setting is used.
- **Workflow:** ALWAYS provide a high-level plan or "Action List" before providing code.

## Infrastructure & Shared Services Pattern
- **Service Naming (Dashes):** Use `infra-` prefix with dashes (e.g., e.g., `infra-postgres`, `infra-mssql`). This acts as the internal DNS hostname.
- **Folder Naming (App Name Only):** Infrastructure app folders should use only the app name, with no `infra_` prefix/suffix pattern (e.g., `postgres`, `mssql`, `redis`).
- **Container Naming (Underscores):** Use `infra_` prefix with underscores (e.g., `infra_postgres`, `infra_mssql`).
- **Persistence & Networking (Underscores):** Strictly follow these patterns:
    - Network: `home_infra_<name>_network`
    - Volume: `home_infra_<name>_data`
- **Connectivity:** - Always attach to `home_network` for cross-app discovery.
    - Set `traefik.enable=false` (Infrastructure is accessed via Service Name, not URLs).
- **Local Environment Variables:** Each app must define its own environment variables in its local `.env` file. If a variable's value should come from a global variable, reference it in the local `.env` file (e.g., `APP_TZ=${TZ}`), not directly in the compose file.

## Service Lifecycle & Discovery
- **New Service Protocol:** Every new application is treated as a modular "plug-in" to the existing Mini-Cloud. It must be research-validated (images, ports, entrypoints) before code generation.
- **Task-Specific Execution:** Detailed scaffolding of new apps is handled via the `/add-app` prompt. Always refer to `.github/prompts/add-app.prompt.md` for the multi-step research and confirmation workflow.
- **Binding Rule:** Services must never listen on `127.0.0.1` inside the container; they must bind to `0.0.0.0` to ensure the Traefik proxy on the `home_network` can reach them.
- **Connectivity Check:** If a service has multiple networks, the Traefik routing label MUST explicitly specify `traefik.docker.network=home_network` to prevent 504 Gateway Timeouts.

## Standard Docker Compose Pattern
When generating a new app in `/apps/<name>/docker-compose.yml`, always follow this template:

1. **External Networking:** Connect the entry-point container to `home_network`.
2. **Internal Networking:** Use a private bridge for internal service-to-service talk if needed.
3. **Labels:** Add Traefik labels to the entry-point container for routing:
   - `traefik.enable=true`
   - `traefik.http.routers.<name>.rule=Host(`<name>.${DOMAIN}`)`
   - `traefik.http.services.<name>.loadbalancer.server.port=<port>`
4. **Multi-Network Routing:** If a container is connected to more than one network, you **MUST** explicitly tell Traefik which network to use for routing to avoid 504 errors.
   - **Label:** `- "traefik.docker.network=home_network"`
5. Avoid using version key in `docker-compose.yml` (use the latest syntax).
6. Avoid hardcoding ports in the compose file; rely on Traefik for routing.
7. **Ordering:** Follow the strict key order: `image`, `container_name`, `restart`, `networks`, `volumes`, `labels`, `environment`, `logging`, `healthcheck`, `depends_on`, `env_file`.
8. **Clean Interpolation:** Use `${VARIABLE}` without hardcoded fallbacks. 
    - **Local Variables Only:** Compose files must only reference variables defined in the app's local `.env` file. Never reference global variables directly in compose files.
    - **Global Variable Mapping:** If an app needs a global variable (like `TZ` or `DOMAIN`), define it in the app's local `.env` file by referencing the global variable (e.g., `MY_APP_TZ=${TZ}`).
    - **Documentation:** In the app's `.env.example` file, clearly document which variables are set from global variables with comments (e.g., `MY_APP_TZ=${TZ}  # Set from global TZ variable`).
9. **Network Attachment:** Public services must use `home_network`. Internal communication must use `home_<appname>_network`.
10. **Explicit Naming**: To prevent Docker Compose from adding folder-name prefixes to resources, always use the `name:` attribute for local networks and volumes. See example below.
```yaml
networks:
  home_myapp_network:
    name: home_myapp_network
    driver: bridge
```
11. **Logging:** Always include log-rotation (max-size: 10m) for every service. Json logging is preferred for better integration with Dozzle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metinsenturk/home-cloud](https://github.com/metinsenturk/home-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
