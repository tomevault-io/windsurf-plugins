---
trigger: always_on
description: Monorepo pnpm + Turborepo: `apps/backend` (NestJS), `apps/web` (Next.js 14), `apps/mobile` (Vite + Capacitor). Auth via Supabase JWT, datos en PostgreSQL con Drizzle ORM. UI con shadcn/ui + estetica Tabler.
---

# CLAUDE.md — Objetiva Comercios Admin

## Proyecto

Monorepo pnpm + Turborepo: `apps/backend` (NestJS), `apps/web` (Next.js 14), `apps/mobile` (Vite + Capacitor). Auth via Supabase JWT, datos en PostgreSQL con Drizzle ORM. UI con shadcn/ui + estetica Tabler.

## Regla principal

Antes de responder cualquier mensaje, evaluar si alguna skill, MCP server o plugin aplica. Si hay aunque sea un 1% de probabilidad de que una herramienta sea util, invocarla. No racionalizar para evitarla.

---

## MCP Servers

### Playwright (`playwright`)

Automatizacion de navegador. Navegar paginas, hacer click, llenar formularios, tomar screenshots, generar PDFs, grabar video.

**Usar cuando:**

- El usuario pide probar la app web visualmente o verificar que una pagina se ve bien
- Necesitas hacer screenshot de un componente o pagina para validar UI
- Testing de integracion E2E en el navegador (login flow, formularios, navegacion)
- Scraping de datos de paginas web
- Verificacion visual de cambios de CSS/layout despues de modificar componentes
- El usuario dice "abrí", "mostrá", "probá en el navegador", "screenshot", "captura"
- Debugging visual: "no se ve bien", "el boton no aparece", "la tabla se rompe"

### Context7 (`context7`)

Documentacion actualizada de librerias. Dos herramientas: `resolve-library-id` (buscar libreria) y `query-docs` (consultar docs).

**Usar cuando:**

- Necesitas la API actual de una libreria (Next.js, NestJS, Drizzle, Supabase, shadcn/ui, Radix, TanStack, Zod, Capacitor, React Hook Form, Recharts, Tailwind)
- No estas seguro de la sintaxis correcta de una funcion o componente
- Trabajas con una version especifica y necesitas docs de esa version
- El usuario pregunta "como se usa X" o "cual es la API de Y"
- Antes de implementar algo con una libreria que no usas frecuentemente
- Para evitar inventar APIs que no existen (alucinaciones)

**Flujo:** Primero `resolve-library-id` con el nombre, luego `query-docs` con el ID resuelto y la pregunta especifica.

### Supabase (`supabase`)

Gestion del proyecto Supabase. SQL, migraciones, Edge Functions, logs, tipos TypeScript.

**Usar cuando:**

- El usuario pide cambios en la autenticacion (Supabase Auth)
- Necesitas ejecutar SQL en Supabase (crear usuarios, asignar roles, modificar auth.users)
- Generar tipos TypeScript desde el schema de Supabase
- Consultar logs de Supabase (auth, API, edge functions)
- Deploy de Edge Functions
- El usuario menciona "supabase", "auth", "login", "signup", "roles", "JWT"
- Diagnostico de problemas de autenticacion ("no puedo loguear", "token invalido")
- Consultar la documentacion oficial de Supabase (knowledge base)

### shadcn-tabler-mcp (`shadcn-tabler-mcp`)

Transformacion de componentes shadcn/ui a estetica Tabler. Tres herramientas: `query-aesthetic` (consultar patron), `transform-component` (leer componente transformado), `map-tokens` (mapear variables CSS).

**Usar cuando:**

- Modificas o creas componentes UI en `apps/web/src/components/ui/`
- Necesitas saber como transformar un componente shadcn a estetica Tabler
- Aplicas estilos a nuevos componentes y necesitas consistencia con el sistema de diseno
- Mapeas variables CSS de Tabler a aliases de shadcn
- El usuario pide "mejorar el diseno", "que se vea como Tabler", "ajustar estilos"
- Cualquier cambio visual en componentes base (button, card, input, table, select, etc.)

**Reglas Tabler clave:** border-radius xl→md, lg→sm. Alturas h-10→h-9, h-9→h-8. Sombras reducir 1 nivel. Padding/gaps reducir (py-6→py-4, gap-6→gap-4). Texto base 14px (text-sm). Controles de formulario necesitan bg explicito.

---

## Plugins

### Superpowers (`superpowers`)

Framework de disciplina de desarrollo. Contiene 14 skills que cubren todo el ciclo: brainstorming, planificacion, implementacion, testing, debugging, code review y finalizacion.

#### superpowers:brainstorming

**Usar cuando:** Antes de cualquier trabajo creativo — crear features, construir componentes, agregar funcionalidad, modificar comportamiento. Explora la intencion del usuario y produce un documento de diseno antes de escribir codigo. Obligatorio antes de implementar.

#### superpowers:writing-plans

**Usar cuando:** Tienes spec o requisitos para una tarea multi-paso. Produce plan con tareas de 2-5 minutos cada una, con paths exactos, codigo completo y pasos de verificacion. Se invoca despues de brainstorming.

#### superpowers:subagent-driven-development

**Usar cuando:** Ejecutas un plan en la sesion actual con tareas independientes. Despacha un subagente fresco por tarea con review de dos fases (spec compliance + code quality).

#### superpowers:executing-plans

**Usar cuando:** Ejecutas un plan escrito en una sesion separada. Ejecuta tareas en lotes de 3 con checkpoints de feedback entre lotes.

#### superpowers:test-driven-development

**Usar cuando:** Implementas cualquier feature o bugfix. Ciclo RED-GREEN-REFACTOR. Regla de hierro: NO escribir codigo de produccion sin un test fallando primero.

#### superpowers:systematic-debugging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [objetiva-comercios/objetiva-comercios-admin](https://github.com/objetiva-comercios/objetiva-comercios-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
