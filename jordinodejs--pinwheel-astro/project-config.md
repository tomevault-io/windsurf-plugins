---
trigger: always_on
description: Estas reglas guían a la AI de Cursor en este repositorio `pinwheel-astro` para producir código consistente, seguro y alineado con las mejores prácticas de Astro 5, Tailwind CSS 4 y el stack moderno de desarrollo web. Mantén estas prácticas en cada cambio.
---

# Reglas de Cursor para Pinwheel Astro

Estas reglas guían a la AI de Cursor en este repositorio `pinwheel-astro` para producir código consistente, seguro y alineado con las mejores prácticas de Astro 5, Tailwind CSS 4 y el stack moderno de desarrollo web. Mantén estas prácticas en cada cambio.

## Stack Tecnológico y Versiones

### Framework Principal
- **Astro**: 5.x (Vite 7) con integraciones `@astrojs/react`, `@astrojs/mdx`, `@astrojs/sitemap` y `astro-auto-import`
- **UI**: Tailwind CSS 4 con plugin `@tailwindcss/vite` y Bootstrap Grid System personalizado
- **Tipado**: TypeScript estricto (`extends: "astro/tsconfigs/strict"`)
- **Contenido**: `astro:content` v5 (Content Layer API) con colecciones en `src/content` y esquemas en `src/content.config.ts`
- **Gestión de paquetes**: PNPM (requerido) - NO uses npm o yarn
- **Fuentes**: `astro-font` para optimización automática de fuentes
- **Analytics**: `@digi4care/astro-google-tagmanager` controlado por `src/config/config.json`

### Arquitectura de Renderizado
- **Hosting**: Sitio completamente estático (SSG) - puede subirse a hosting básico como Hostinger
- **Componentes React**: Se hidratan en cliente con directivas apropiadas (`client:*`)
- **Sin SSR**: No usa `output: 'server'` ni adaptadores de servidor

## Estructura y Convenciones de Archivos

### Organización de Directorios
```
src/
├── pages/           # Rutas de Astro (file-based routing)
├── layouts/         # Layouts base y componentes de layout
│   ├── components/  # Componentes Astro reutilizables
│   ├── partials/    # Header, Footer, CTA
│   └── shortcodes/  # Componentes MDX auto-importados
├── content/         # Colecciones de contenido (blog, careers, etc.)
├── lib/             # Utilidades y helpers
├── styles/          # CSS layers y utilidades personalizadas
└── config/          # Archivos de configuración JSON
```

### Alias de Importación (tsconfig.json)
SIEMPRE usa estos alias en lugar de rutas relativas:
- `@/components/*` → `src/layouts/components/*`
- `@/shortcodes/*` → `src/layouts/shortcodes/*`
- `@/partials/*` → `src/layouts/partials/*`
- `@/*` → `src/*`

### Convención de Nomenclatura
- **Archivos**: PascalCase para componentes (`Banner.astro`), kebab-case para utilidades (`text-converter.ts`)
- **Componentes**: Nombres descriptivos y específicos (`BlogPostCard`, `ServiceFeatures`)
- **Variables/funciones**: camelCase con nombres explícitos, evita abreviaturas
- **Constantes**: UPPER_SNAKE_CASE
- **Contenido**: `-index.md(x)` para páginas índice de colección

## Reglas de Codificación TypeScript

### Tipado Estricto
- **NO uses `any`** - usa tipos específicos o `unknown` si es necesario
- **Props en .astro**: Define `interface Props` y desestructura con valores por defecto
- **Content Collections**: Usa `CollectionEntry<"collection">` para props de entrada
- **Funciones exportadas**: Siempre con tipos de retorno explícitos

```astro
---
interface Props {
  title: string;
  description?: string;
  featured?: boolean;
}

const { title, description = "Default description", featured = false } = Astro.props;
---
```

### Importaciones y Módulos
- **Extensiones de archivo**: Incluye `.js` para archivos TypeScript importados
- **Importaciones**: Agrupa por tipo (tipos, utilidades, componentes)
- **Re-exportaciones**: Evita exportaciones masivas (`export *`)

```ts
// Correcto
import { markdownify, plainify } from "@/lib/utils/textConverter.js";
import type { CollectionEntry } from "astro:content";
```

## Astro 5 - Mejores Prácticas

### Content Collections (Content Layer API)
- **Ubicación**: Define colecciones en `src/content.config.ts`
- **Loaders**: Usa `glob()` y `file()` loaders nativos
- **Schemas**: Siempre define schemas con Zod para validación
- **Consultas**: Usa `getCollection()` y `getEntry()` con tipos apropiados

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: "**/*.md", base: "./src/content/blog" }),
  schema: z.object({
    title: z.string(),
    description: z.string().optional(),
    pubDate: z.coerce.date(),
    featured: z.boolean().default(false),
  })
});
```

### Componentes y Layouts
- **Props tipadas**: Siempre usa interfaces TypeScript para props
- **Layout base**: Usa `src/layouts/Base.astro` como wrapper principal
- **Metadatos**: Pasa `title`, `meta_title`, `description`, `image` cuando sea necesario
- **Slots**: Usa slots nombrados para contenido estructurado

### Islas de React
- **Principio**: Astro renderiza HTML por defecto, usa React SOLO para interactividad
- **Directivas de hidratación**:
  - `client:load` - Para interacción inmediata (formularios críticos)
  - `client:idle` - Para componentes diferibles (widgets)
  - `client:visible` - Para contenido que entra en viewport (carruseles)
- **Importaciones**: Asegúrate que componentes React estén en `layouts/function-components/`

```astro
---

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JordiNodeJS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
