---
trigger: always_on
description: > Este archivo rige el comportamiento de cualquier agente de IA (Claude, GPT, Copilot, Cursor, etc.) que trabaje en este repositorio. Las reglas son **obligatorias** y prevalecen sobre las preferencias del agente. El objetivo es mantener la calidad, coherencia y estabilidad del proyecto entre múltiples contribuyentes.
---

# AGENTS.md — Reglas de colaboración para agentes de código

> Este archivo rige el comportamiento de cualquier agente de IA (Claude, GPT, Copilot, Cursor, etc.) que trabaje en este repositorio. Las reglas son **obligatorias** y prevalecen sobre las preferencias del agente. El objetivo es mantener la calidad, coherencia y estabilidad del proyecto entre múltiples contribuyentes.

---

## 0. Principio rector

**Mejora sin romper.** Toda intervención debe dejar el repositorio en un estado igual o mejor del que se encontró. Si un cambio rompe el build, las pruebas, el fallback offline o el comportamiento existente, **se revierte**. No se mergea nada en rojo.

---

## 1. Stack tecnológico — no se negocia

| Capa | Tecnología | Prohibido |
|------|------------|-----------|
| Framework | Astro 7 (static output, adapter Vercel) | No migrar a Next.js, Remix, SvelteKit |
| Backend de datos | Supabase (Postgres + JS client `@supabase/supabase-js`) | No cambiar a MongoDB, Firebase, o Prisma |
| Estilos | CSS modular plano (`src/styles/*.css`) importado por componente | No introducir Tailwind,styled-components, CSS Modules, Stitches, vanilla-extract |
| Iconos | SVG inline (Lucide-style, `stroke-width="2"`), sin librerías | No mezclar Heroicons + Phosphor + Lucide; no usar emoji como icono de feature |
| JS del cliente | Vanilla JS en `<script>` dentro de `.astro`, sin build step | No introducir React, Vue, Svelte, Alpine, Stimulus |
| Package manager | pnpm ( existe `pnpm-lock.yaml` y `pnpm-workspace.yaml`) | No usar npm ni yarn para instalar; nunca mezclar lockfiles |
| Despliegue | Vercel (`@astrojs/vercel`) | No cambiar a Netlify, Cloudflare Pages sin aprobación explícita |

**Regla:** Si el proyecto ya usa una tecnología, se mantiene. No se introducen dependencias nuevas sin justificación documentada y sin romper el bundle actual.

---

## 2. Comandos del proyecto

```bash
pnpm install      # Instala dependencias
pnpm dev          # Servidor de desarrollo (http://localhost:4321)
pnpm build        # Build de producción — DEBE pasar sin errores ni warnings
pnpm preview      # Previsualiza el build
```

**Antes de declarar una tarea completa, el agente DEBE ejecutar `pnpm build` y confirmar que termina con `Complete!` y sin warnings.** Si hay warnings, se arreglan antes de entregar.

No existe suite de pruebas automatizadas ni linter configurado — el build de Astro es la verificación mínima y luego la revisión manual por parte del usuario. Si se añaden pruebas o linter en el futuro, el agente debe ejecutarlos también.

---

## 3. Reglas de código — obligatorias

### 3.1 Estilo general

- **Indentación:** 2 espacios. Sin tabs.
- **Comillas:** Doble (`"`) para strings en JS, simple (`'`) en CSS.
- **Punto y coma:** Sí, al final de cada sentencia JS.
- **Trailing comma:** Sí, en objetos y arrays multilínea.
- **Longitud de línea:** 100 caracteres máximo (aprox.). No es un muro, pero justifica excepciones.
- **Comentarios:** **CERO comentarios** en el código a menos que el usuario los pida explícitamente. El código se documenta con nombres claros y estructura — no con comentarios explicativos.
- **Idioma del código:** Variables y funciones en inglés o español según el archivo circundante. El proyecto mezcla ambos — **mira el archivo antes de elegir** y mantén coherencia con el archivo que editas.

### 3.2 Componentes Astro (`.astro`)

- Frontmatter: `---` con imports al inicio, lógica después, y se exporta Props interface tipada cuando recibe props.
- Un componente por archivo. El nombre del archivo coincide con el nombre del componente (PascalCase).
- **No se inlinean estilos en `<style>` dentro de los componentes.** Los estilos viven en `src/styles/*.css` y se importan: `import "../styles/foo.css";`.
- HTML semántico: `nav`, `main`, `header`, `footer`, `section`, `article`, `dialog` antes que `div`.

### 3.3 CSS — reglas estrictas

1. **Un archivo CSS por UI domain** (`navbar.css`, `hero.css`, `index.css`, etc.). No se crea un único CSS gigante; no se inlinean estilos globales en componentes.
2. **Variables CSS en `:root`** dentro de `global.css`. **NunCA** introducir colores nuevos como literales hex si ya existe una variable para ese rol (ver §5).
3. **Mobile-first.** Breakpoints con `@media (max-width: Npx)` donde N sea 480 / 640 / 768. El proyecto usa este patrón — manténgo.
4. **`transition: all` está prohibido.** Listar propiedades específicas: `transition: transform .2s, box-shadow .2s, border-color .2s`.
5. **Animaciones GPU-only.** Solo `transform` y `opacity` se animan. `width`, `height`, `top`, `left`, `margin`, `padding` no se animan.
6. **`prefers-reduced-motion` se respeta.** Todo el código de animación/transición está cubierto por el reset global en `global.css`. Animaciones nuevas deben probar que no lo rompen. Animaciones pulsadas infinitas (ej. `nec-pulse`) deben incluirse en el reset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Projects-for-Venezuela/infoayudavenezuela](https://github.com/Projects-for-Venezuela/infoayudavenezuela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
