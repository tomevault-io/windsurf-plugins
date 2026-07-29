---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Apache Airavata — a unified middleware for distributed computing resource management. Single Spring Boot JVM serving gRPC + REST via Armeria on port 9090. Protobuf defines the API contracts; HTTP/JSON transcoding provides REST automatically from gRPC definitions.

## Build & Dev

```bash
tilt up                                           # Start everything (infra + server). ALWAYS use this.
mvn install -DskipTests -T4                       # Build only (no server start)
mvn test -T4                                      # Unit tests (parallel)
mvn test -pl airavata-api -Dgroups=runtime        # Integration tests (needs Docker via tilt)
mvn test -pl airavata-api/research-service        # Single module tests
```

Health check: `https://api.airavata.host/actuator/health`
API docs: `https://api.airavata.host/docs`

### Verifying changes

Compile-green is not enough — the running Tilt-managed server must actually stay up. After each batch of changes:

```bash
tilt logs airavata-server 2>&1 | tail -40        # scan for stack traces
curl https://api.airavata.host/actuator/health
```

Do not run `mvn clean` during an active session: it deletes `airavata-server/target/*.jar` while the JVM is running, and lazy classloaders (e.g. the MariaDB JDBC ServiceLoader) then fail at runtime with `NoSuchFileException`, cascading into HikariCP connection timeouts. Prefer incremental `mvn compile`. If a clean was unavoidable, rebuild and force a restart: `mvn package -DskipTests -pl airavata-server -am` then `tilt trigger airavata-server` (Tilt may watch sources but not the jar artifact).

### Infrastructure (started by `tilt up`)

Services run directly in the shared colima VM (`airavata` profile) on the `airavata-devstack`
docker network. A single shared Traefik ingress (managed by `devstack/`) binds `127.0.0.1:80/443`,
routes by Host header with a mkcert-trusted wildcard cert for `*.airavata.host`, and forwards to
containers by label. State is ephemeral: `tilt down` stops containers but volumes persist until
you run `./devstack/devstack reset`.

**One-time host setup (run once per machine):**

```bash
./devstack/devstack setup     # installs colima/dnsmasq/mkcert, starts VM, starts Traefik, sets up DNS
```

After that, `tilt up` is all you need.

Reach services by hostname (`*.airavata.host` → `127.0.0.1` via dnsmasq; HTTPS via shared Traefik;
no `/etc/hosts` needed):

| Hostname | Service | Creds |
|----------|---------|-------|
| `api.airavata.host` | Airavata server (gRPC + REST) | — |
| `auth.airavata.host` | Keycloak | `admin` / `admin` |
| `adminer.airavata.host` | Adminer (`--profile tools`) | — |
| `gateway.airavata.host` | Django portal (separate `tilt up --port 10351` in `airavata-portals`) | `default-admin` / `ade4#21242ftfd` |

### Web portals (separate `tilt up`)

The web portals run from the sibling `airavata-portals` repo. Both repos share the same
colima VM and `airavata-devstack` network, so the portal container reaches the server at
`airavata-server:9090` (in-network plaintext gRPC).

```bash
tilt up                                          # 1. this repo — infra + server
cd ../airavata-portals && tilt up --port 10351   # 2. portals (distinct Tilt UI port)
```

MariaDB is published on `127.0.0.1:3306` (replacing the old `13306`) and reachable
inside the VM as `db.airavata.host:3306`. Internally the server reaches infra by
service name (`kafka:9092`, `keycloak:18080`, `sftp:22`; SFTP creds `airavata`/`pass`).

**Restart matrix:**

| Scenario | Action |
|----------|--------|
| Machine reboot | `./devstack/devstack ensure` (idempotent; restarts VM + ingress if stopped), then `tilt up` |
| Colima stopped manually | `colima start -p airavata`, then `tilt up` |
| Wipe all data | `./devstack/devstack reset` (global — destroys the shared VM) |

Gotchas:
- Connection settings are injected as `-D` system properties via `JAVA_TOOL_OPTIONS` in the
  `airavata-server` service. Airavata's `ApplicationSettings` resolves *dotted* keys
  (`system-property > env-var-with-exact-dotted-key > file`), so `SPRING_DATASOURCE_URL`-style
  env vars do **not** override `kafka.broker.url` / `spring.datasource.url` / etc.
- The Keycloak issuer is `https://auth.airavata.host/realms/default` for both browser and
  in-network server (TLS terminates at Traefik, which forwards `X-Forwarded-Proto`).
- `docker` must be on `PATH` for Tilt's shell-outs (colima installs the CLI keg-only — run
  `brew link docker` if `which docker` is empty).
- The mkcert root CA is mounted at `/certs/rootCA.pem` in the server container and imported
  into the JVM truststore at startup (alias `mkcert-airavata`, idempotent).

## Database schema & seeding

Schema and seed data are applied **outside the JVM**, before the server starts — the server
only validates. There is no Flyway runtime and no Java seeder beans.

- **Schema**: `airavata-server/src/main/resources/db/migration/airavata/V1__Baseline_schema.sql`
  is a single collapsed baseline (the JPA `@Entity`-derived DDL). The `db` container mounts it as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/airavata](https://github.com/apache/airavata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
