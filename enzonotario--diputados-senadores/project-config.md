---
trigger: always_on
description: Instrucciones para agentes que trabajan en este repo.
---

# AGENTS.md

Instrucciones para agentes que trabajan en este repo.

## Qué es

Una sola app **Nuxt 4 (SSR)** que sirve **Congreso** (landing), **Diputados** y **Senadores** según el `Host`. Misma codebase, tres sitios.

| Sitio | Host local | Host prod |
|--------|------------|-----------|
| Congreso (landing) | `congreso.localhost:3200` | `congreso.argentinadatos.com` |
| Diputados | `diputados.localhost:3200` | `diputados.argentinadatos.com` |
| Senadores | `senadores.localhost:3200` | `senadores.argentinadatos.com` |

`*.localhost` suele bastar (sin `/etc/hosts`). Alternativa: `*.localhost.test` (ver `vite.server.allowedHosts` en `nuxt.config.ts`).

En **congreso.** solo existe `/` (landing con CTAs a las dos cámaras). Cualquier otra ruta hace 302 a `/`.

## Arranque

```bash
pnpm install
pnpm dev          # :3200 --host
pnpm build        # SSR Node + hybrid SSG (Coolify / Docker)
pnpm start        # node .output/server/index.mjs
pnpm preview
pnpm lint
pnpm lint:fix
```

Package manager: **pnpm**.

## Deploy (Coolify / Docker / VPS) — recomendado

**Tres servicios Coolify** (misma imagen base, distinto tag y `NUXT_PUBLIC_DEFAULT_CHAMBER`):

| Servicio | Dominio | `IMAGE_TAG` | `NUXT_PUBLIC_DEFAULT_CHAMBER` |
|----------|---------|-------------|-------------------------------|
| Diputados | `diputados.argentinadatos.com` | `diputados-latest` | `diputados` |
| Senadores | `senadores.argentinadatos.com` | `senadores-latest` | `senadores` |
| Congreso | `congreso.argentinadatos.com` | `congreso-latest` | `congreso` |

Runtime sigue resolviendo sitio por `Host`; el env fija el **manifiesto SSG** del build (hybrid). Congreso solo prerenderiza `/`.

### Hybrid SSG

En build (`app/lib/prerender-manifest.ts` + hook `prerender:routes`):

- **Congreso:** solo `/` (landing)
- **Cámaras:** índices (`/`, `/actas`, listados) + miembros **activos** (sin `/afinidad`) + actas `fecha >= buildDate − 4 años` (`ACTAS_SSG_YEARS`)

El resto: SSR + `Cache-Control` largo (`max-age=31536000`). Tras deploy, **purge Cloudflare** de esos paths si el CDN cacheó HTML viejo.

Datos en RAM (`*-data.ts`) + mini-API Nitro (`server/api/*`). El browser no baja dumps de `api.argentinadatos.com`. SQLite solo si hay varias réplicas sin RAM compartida.

### Coolify: zero-downtime (evitar `404 page not found`)

El texto plano **`404 page not found`** (sin UI de la app) es Traefik sin backend sano — no es un 404 de Nuxt. Con healthchecks ON y sin rolling, Coolify para el contenedor viejo antes de que el nuevo esté listo → minutos de 404 durante el pull.

En **cada** app (diputados / senadores / congreso):

1. **Rolling Updates: ON** (Advanced / General). Requiere healthcheck válido y **nombre de contenedor por defecto** (sin custom name). Sin port mapping al host.
2. **Healthcheck: ON**
   - Path: `/api/health`
   - Port: `3000` (el de la app, no 80)
   - Expected status: `200`
   - Host: `localhost` (Coolify lo exige)
3. Build pack = **Dockerfile thin** de la raíz (solo pull), no `Dockerfile.build` en el VPS.
4. Tras deploy: el viejo sigue *running* hasta que el nuevo pase a *healthy*. Puede quedar un hueco corto (~segundos) por bugs conocidos Coolify/Traefik; no minutos.

La imagen runtime (`Dockerfile.build`) instala `curl` (la UI de Coolify lo usa) y declara `HEALTHCHECK` Docker contra `/api/health` (`interval=10s`, `start-period=40s`).

### Coolify: el VPS no debe compilar Nuxt

El `Dockerfile` de la raíz **solo hace** `FROM ghcr.io/...` (pull). El build pesado está en `Dockerfile.build` (GitHub Actions, matrix por cámara).

1. Push → Actions buildea **tres** imágenes (`diputados-latest` / `senadores-latest` / `congreso-latest` + `:<site>-<sha>`).
2. Coolify “build” del `Dockerfile` = pull de GHCR (segundos). **No** debe aparecer `RUN pnpm build` en los logs.
3. Si ves `Building docker image` + `pnpm build` + exit 137: todavía está usando el Dockerfile viejo multi-stage; redeployá con el `Dockerfile` thin.
4. Package GHCR privado → en Coolify, Docker Registry: `ghcr.io` + PAT `read:packages`.
5. Build arg `IMAGE_TAG` = `diputados-latest`, `senadores-latest` o `congreso-latest` (o `diputados-<sha>` puntual).
6. Evitá race Git vs Actions: desactivá auto-deploy al push. Secrets GHA: `COOLIFY_API_TOKEN`, `COOLIFY_APP_UUID_DIPUTADOS`, `COOLIFY_APP_UUID_SENADORES`, `COOLIFY_APP_UUID_CONGRESO`.

```bash
# Local (máquina con RAM) — una cámara
docker build -f Dockerfile.build \
  --build-arg NODE_MAX_OLD_SPACE_SIZE=6144 \
  --build-arg NUXT_PUBLIC_DEFAULT_CHAMBER=diputados \
  -t diputados-senadores:diputados .
```

Cuando haya movimiento en las cámaras (datos en RAM):

```bash
curl -X POST https://senadores.argentinadatos.com/api/revalidate \
  -H "Authorization: Bearer $NUXT_REVALIDATE_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"clearData":true}'
```

Eso vacía las caches en RAM; el próximo request vuelve a bajar datos de `api.argentinadatos.com`. **No** invalida HTML SSG ni CDN: hace falta redeploy (o purge CF de paths SSR largos).

Redirects SEO nombre→id: `server/middleware/legacy-seo.ts` (mapa en `assets/legacy-senador-redirects.json`).

## Reglas duras


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enzonotario/diputados-senadores](https://github.com/enzonotario/diputados-senadores) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
