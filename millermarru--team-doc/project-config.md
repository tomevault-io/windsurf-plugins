---
trigger: always_on
description: > Referencia técnica completa del proyecto. Leer antes de hacer cualquier cambio.
---

# CLAUDE.md – DocHubs

> Referencia técnica completa del proyecto. Leer antes de hacer cualquier cambio.

---

## Descripción general

**DocHubs** es una plataforma de documentación colaborativa construida con Next.js 16 (App Router) + React 19 + TypeScript. Permite a developers escribir documentación en Markdown, publicarla al instante y compartirla. Los documentos se almacenan como archivos `.md` en un repositorio de GitHub externo; el proyecto NO usa base de datos propia.

**Autores:** Miller Zamora & Sam Vasquez  
**Idioma del proyecto:** Español  
**Deploy objetivo:** Vercel

---

## Stack tecnológico

| Capa | Tecnología |
|---|---|
| Framework | Next.js 16.1.6 (App Router) |
| UI | React 19 |
| Lenguaje | TypeScript 5 |
| Estilos | Tailwind CSS 4 + `@tailwindcss/typography` |
| Componentes UI | shadcn/ui (Radix UI base) |
| Iconos | lucide-react |
| Temas | next-themes (dark/light) |
| Markdown | react-markdown + remark-gfm + rehype-slug + rehype-autolink-headings + rehype-pretty-code + shiki |
| Búsqueda | cmdk |
| Auth | jose (JWT HS256) + bcryptjs |
| GitHub API | @octokit/rest |
| Notificaciones | sonner |
| Analytics | @vercel/analytics |
| Fuentes | Geist Sans + Geist Mono (Google Fonts) |

---

## Estructura de directorios

```
app/
  layout.tsx              → Root layout (Navbar, Footer, Providers, Toaster, Analytics)
  page.tsx                → Landing page pública
  not-found.tsx           → 404 global
  globals.css             → Estilos globales Tailwind

  admin/
    page.tsx              → Panel admin (Server Component, requiere sesión)
    login/page.tsx        → Página de login
    _components/
      AdminEditor.tsx     → Editor principal: WYSIWYG Markdown con preview
      LoginForm.tsx       → Formulario de login

  api/
    auth/
      login/route.ts      → POST – verifica credenciales, setea cookie JWT
      logout/route.ts     → POST – borra la cookie de sesión
    docs/
      nav/route.ts             → GET – retorna el index.json (navegación)
      upsert/route.ts          → POST – crea o actualiza un doc en GitHub (requiere auth)
      upload-image/route.ts    → POST – sube imagen al repo GitHub y devuelve URL pública

  docs/
    layout.tsx            → Layout con Sidebar + área de contenido
    [[...slug]]/
      page.tsx            → Página dinámica de docs (/docs o /docs/a/b/c)
      not-found.tsx       → 404 específico de docs

  team/
    page.tsx              → Página del equipo (Miller + Sam)

components/
  Navbar.tsx              → Header sticky (Server Component, carga el index)
  Footer.tsx              → Footer global
  MobileMenu.tsx          → Menú hamburguesa para mobile
  Providers.tsx           → ThemeProvider wrapper

  docs/
    Sidebar.tsx           → Navegación lateral izquierda (secciones + items)
    Markdown.tsx          → Renderizador de Markdown con syntax highlighting
    SearchCmdk.tsx        → Búsqueda de docs con cmdk (modal)
    Toc.tsx               → Tabla de contenidos (anchors de headings)
    MobileSidebar.tsx     → Sidebar en modal para mobile

  ui/                     → Componentes shadcn: button, card, input, label,
                            popover, select, sonner, ThemeToggle

lib/
  auth.ts                 → JWT (crear/leer sesión), bcrypt, lectura de users.json
  github.ts               → Cliente Octokit singleton (leer/escribir archivos en repo)
                            getFileContent, upsertFile (texto)
                            getFileSha, upsertBinaryFile (imágenes/binarios)
                            buildRawUrl (URL pública raw.githubusercontent.com)
  docs.ts                 → getDocsIndex, getDocContent, upsertIndexItem, extractToc
  utils.ts                → cn() helper

types/
  index.ts                → DocItem, DocSection, DocsIndex, UpsertDocBody, SessionPayload

data/
  users.json              → Usuarios admin [{username, passwordHash}]

scripts/
  hash-password.mjs       → Script para generar hash bcrypt de contraseñas

public/
  images/                 → Fotos del equipo (miller.jpeg, sam.jpeg)
```

---

## Variables de entorno requeridas

Archivo: `.env.local` (nunca commitear)

```env
# GitHub donde se almacenan los docs
GITHUB_TOKEN=ghp_...         # Personal Access Token con permisos repo
GITHUB_OWNER=usuario          # Owner del repo
GITHUB_REPO=nombre-repo       # Nombre del repo
GITHUB_BRANCH=main            # Rama (default: main)

# JWT de sesión admin
AUTH_SECRET=string-largo-aleatorio-seguro
```

- Si `GITHUB_TOKEN` no está: el sistema intenta leer desde `raw.githubusercontent.com` (solo lectura, repo público).
- Si `AUTH_SECRET` no está: la app lanza error al intentar crear/leer sesiones.

---

## Sistema de documentación (flujo de datos)

```
GitHub Repo (fuente de verdad)
  ├── index.json              → Índice con todas las secciones e items
  └── docs/
      ├── seccion/slug.md    → Archivos Markdown de cada documento
      └── images/            → Imágenes subidas desde el AdminEditor
          └── nombre-1234567890.png
```

1. `getDocsIndex()` → lee `index.json` del repo (caché ISR 120s en public, sin caché en private).
2. `getDocContent(slug)` → lee el `.md` del repo usando el `path` del índice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MILLERMARRU/team_doc](https://github.com/MILLERMARRU/team_doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
