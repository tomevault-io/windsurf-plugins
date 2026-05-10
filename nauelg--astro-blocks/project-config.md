---
trigger: always_on
description: Copyright (c) 2026 Nauel Gómez Gamero
---

<!--
Copyright (c) 2026 Nauel Gómez Gamero
Licensed under the Business Source License 1.1
-->

# Guía para agentes – AstroBlocks

Documento de referencia para iteraciones futuras sobre el CMS. Describe la estructura, convenciones y puntos de extensión.

---

## 1. Estructura del directorio

```
lib/astro-blocks/
├── plugin/           # Integración Astro (injectRoute, runtime, config)
│   └── index.ts      # Entry del plugin; hook astro:config:setup
├── contract/         # Contrato de componentes (defineBlockSchema)
│   └── index.ts
├── api/              # Capa de datos y handlers HTTP
│   ├── data.ts
│   └── handlers.ts
├── routes/           # Entrypoints inyectados (no están en src/pages)
│   ├── admin/        # Panel: layout.astro, components/, index, pages, redirects, configs, users, settings, menus, languages, cache
│   │   └── components/
│   │       ├── DetailModal.astro   # Modal reutilizable para crear/editar (mismo diseño que formularios)
│   │       ├── ConfirmDialog.astro # Diálogo de confirmación (overlay + panel centrado); expone window.cmsConfirm()
│   │       └── AlertDialog.astro    # Diálogo de aviso (mismo estilo); expone window.cmsAlert()
│   │   └── client/   # Helpers cliente compartidos y módulos para scripts del admin
│   └── api/          # catchall.ts
├── styles/           # Estilos del panel (white-label)
│   └── cms-admin.css # Overrides Pico, layout, footer (.cms-footer, .cms-footer-logo), componentes, .cms-detail-modal, .cms-dragging, .cms-dropzone
├── img/              # Assets del paquete (logo para footer y README)
│   └── blocks_logo.jpg
├── utils/            # Utilidades compartidas (bloques, slugs, paths, menús)
├── types/            # Tipos compartidos del dominio
├── scripts/          # Build local del paquete
├── meta/             # Metadatos internos del paquete (p. ej. catálogo de features para la web informativa)
│   └── features.json
├── dist/             # Artefacto distribuible generado por tsc + copia de assets
├── playgrounds/
│   └── basic/        # Proyecto Astro consumidor para validar el paquete
├── package.json      # exports tipados a dist/, scripts y workspaces
├── README.md         # Solo consumidor
├── DEVELOPING.md     # Solo mantenedor del paquete
├── LOCAL_PACKAGE_TESTING.md # Flujo temporal para probar el tarball local
└── AGENTS.md
```

- **Datos del proyecto (fuera del paquete):** `data/` y `public/uploads/` en la **raíz del proyecto**. El plugin usa `projectRoot` (p. ej. `process.cwd()` o `ASTRO_BLOCKS_PROJECT_ROOT`).

---

## 2. Rutas y prefijos

- **Panel:** todo bajo **`/cms`**: `/cms`, `/cms/pages`, `/cms/redirects`, `/cms/configs`, `/cms/users`, `/cms/settings`, `/cms/menus`, `/cms/languages`, `/cms/cache`. El detalle (crear/editar) se hace en modal en la propia lista, no hay rutas `/new` ni `/[id]`.
- **API:** bajo **`/cms/api`**: `/cms/api/pages`, `/cms/api/pages/[id]`, `/cms/api/site`, `/cms/api/menus`, `/cms/api/redirects`, `/cms/api/configs`, `/cms/api/languages`, `/cms/api/users`, `/cms/api/upload`, `/cms/api/cache/invalidate`.
- **Páginas del sitio:** ruta inyectada **`/[...slug]`**. En alpha, el modo por defecto es SSR con cache experimental de Astro (`routes/page.astro`). Si el consumidor fuerza `publicRendering: 'static'`, el plugin inyecta `routes/page-static.astro`. Home = slug vacío o `/`.
- **Sitemap / robots:** `/sitemap-index.xml`, `/robots.txt` (endpoints con `prerender = false`).

Al añadir rutas nuevas del panel o de la API, mantener estos prefijos y actualizar enlaces y `fetch()` en los .astro del admin.

**Estructura del panel:** `layout.astro` incluye topbar mínima y contextual (título de vista, pill del sitio si aplica, CTA `Ver sitio` y perfil con dropdown reducido a acciones realmente necesarias, actualmente `Salir`), sidebar con menú agrupado (Dashboard; Contenido: Páginas, Menús; Configuración: Ajustes, Caché), y **footer fijo** (`.cms-footer`) con el logo de AstroBlocks (`img/blocks_logo.jpg`, optimizado con `astro:assets`), nombre y versión. El contenido hace scroll entre topbar y footer. Iconos con `@lucide/astro`. El branding lateral debe mantenerse deliberadamente sobrio: logo + texto `Content platform`, sin duplicar el nombre del producto. La pantalla `/cms/cache` se usa para invalidación total de caché; no lanza builds ni va en el formulario de edición de página.

---

## 3. Estilos del panel (Pico CSS, Animate.css, tema white-label y design system)

- **Tailwind eliminado.** El panel no usa Tailwind ni ninguna integración de estilos inyectada desde el plugin.
- **Base UI:** Pico CSS (`@picocss/pico`). Se importa en `routes/admin/layout.astro` junto con Animate.css y `styles/cms-admin.css`. Orden: Pico → Animate.css → cms-admin.css para que los overrides del CMS tengan prioridad.
- **Tema white-label:** En el layout se inyectan en `<body class="cms-root">` las variables `--cms-primary` y `--cms-secondary` desde `site.primaryColor` y `site.secondaryColor` (Settings). En `cms-admin.css`, `.cms-root` redefine `--pico-primary` (y variantes) con `var(--cms-primary)` para que Pico use el color del tema.

### 3.1. Principios del design system

El panel debe seguir siempre estos principios visuales:

- **90% neutro, 10% color de acento.** El color configurable (`--cms-primary`) es un acento, no el color dominante del layout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NauelG/astro-blocks](https://github.com/NauelG/astro-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
