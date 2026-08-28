---
trigger: always_on
description: Sitio web del Challenge JAR (desafío de comportamiento robótico con una tarea a revelar) para la JAR 2026 — Jornada Argentina de Robótica (Rosario, 3 al 6 de noviembre), organizado por el AIR Club UdeSA. Es un sitio estático en Astro con información del desafío, guía de setup y workshops asincrónicos, deployeado a GitHub Pages.
---

# Contexto

Sitio web del Challenge JAR (desafío de comportamiento robótico con una tarea a revelar) para la JAR 2026 — Jornada Argentina de Robótica (Rosario, 3 al 6 de noviembre), organizado por el AIR Club UdeSA. Es un sitio estático en Astro con información del desafío, guía de setup y workshops asincrónicos, deployeado a GitHub Pages.

Ecosistema de 3 repos de la org [AIRclub-UdeSA](https://github.com/AIRclub-UdeSA):

- [`yahboom_rosmaster`](https://github.com/AIRclub-UdeSA/yahboom_rosmaster): simulador del ROSMASTER X3 (fork de automaticaddison adaptado a Humble + Gazebo Fortress).
- `jar_site` (este repo): el sitio web.
- [`jar_workshops`](https://github.com/AIRclub-UdeSA/jar_workshops): código de los workshops semanales.

# Stack y decisiones tomadas

- Astro puro, sin framework de UI ni Tailwind ni Starlight: decisión deliberada para tener control total del diseño.
- Identidad visual: **"Black Void & Sculptural Scale"** (estilo Dala / Linear / Mission Control). Canvas negro profundo (`#050807` / `#000000`), sin cajas grises repetitivas; elementos flotando sobre el vacío con espaciado generoso y separadores hairline de 1px (`rgba(255, 255, 255, 0.07)`).
- La landing suma una capa de **journal técnico editorial**: la tipografía lleva la jerarquía y el chrome aparece solo cuando representa una acción o un estado real. Evitar HUDs, terminales decorativas, glows, marcos de visor, cards repetidas y etiquetas ornamentales.
- Tipografía: Space Grotesk (display a gran escala con tracking ajustado `-0.035em`), Archivo (cuerpo ligero y legible) y JetBrains Mono (etiquetas de sistema, telemetría y terminales).
- Acentos cromáticos: Verde `#4ade80` como acento funcional primario (color del ROSMASTER, indicador de estados activos y telemetría viva) y Ámbar `#fbbf24` como secundario (callouts de misión y avisos).
- Dinámica de equipos: Postulación libre y gratuita, sin límite de integrantes por equipo; selección previa por cupos físicos en pista. Al menos un integrante (capitán/representante) debe viajar a Rosario.
- Tema único oscuro: no hay toggle ni tema claro. Ver `src/styles/global.css` — los tokens CSS variables están al inicio del archivo.
- La documentación vive en markdown vía la colección `docs`; la landing usa componentes Astro con copy explícito. El shell (nav, sidebar, footer) son componentes propios en `src/components/`.
- Solo español rioplatense con voseo. Sin emojis.

# Estructura

- `src/pages/`: `index.astro` compone la landing y `[...slug].astro` sirve los docs.
- `src/layouts/`: layouts Base y Doc.
- `src/components/`: Nav, Footer, Sidebar y shell compartido.
- `src/components/docs/`: tabla de contenidos e índices editoriales generados desde la colección.
- `src/components/landing/`: capítulos de la landing (`LandingHero`, `DonatelloProfile`, `ChallengeOverview`, `ObservationPlate`, `Roadmap`, `EventDetails`, `Resources`, `ApplicationBand`, `Closing`).
- `src/config/site.ts`: navegación del sidebar (secciones fijas; los workshops se autogeneran desde archivos `semana-*.md`) y metadatos.
- `src/content/docs/`: contenido markdown en subcarpetas `setup/`, `workshops/`.
- `src/templates/`: plantillas fuera de la colección (ej.: `plantilla-semana.md` para nuevas semanas).
- `src/styles/global.css`: tokens, shell y estilos compartidos de documentación.
- `src/styles/landing.css`: estilos exclusivos de la landing, importados solo desde `index.astro`; prefijo de selectores `landing-`.
- `public/models/rosmaster_unified.glb`: modelo CAD optimizado que usa el visor actual.
- `public/models/rosmaster/`: URDF de referencia; ya no es el asset cargado por la landing.
- `public/media/`: videos y posters optimizados de Gazebo/RViz usados por `ObservationPlate`; `public/media/docs/` contiene figuras SVG propias de las guías.
- `scripts/sync-robot.sh`: regenera el modelo 3D desde un workspace ROS local.

# Confidencialidad

**No revelar la temática del desafío (búsqueda y rescate, víctimas, conos) en ningún contenido público hasta que el club lo anuncie oficialmente.** Usar lenguaje genérico: "desafío de comportamiento robótico", "una tarea que se revelará en la JAR". La carpeta `src/content/docs/competencia/` se volverá a agregar cuando se revele la temática.

# Convenciones de contenido

- Frontmatter schema: `title` requerido; `description`, `duration`, `outcome` y `prerequisites` opcionales; `level`: `inicial` | `intermedio` | `avanzado`; `status`: `listo` | `proximamente` (default `listo`). Los índices de Setup y Workshops consumen estos datos directamente de la colección, por lo que no se duplican a mano.
- Links entre páginas markdown SIEMPRE relativos (`./x/` o `../seccion/x/`), porque el sitio vive bajo `/jar_site`.
- Archivos nuevos de workshop: prefijo `semana-NN-` con dos dígitos → entran solos al sidebar, ordenados alfabéticamente.
- Comandos de código siempre probados para Ubuntu 22.04 + ROS 2 Humble + Gazebo Fortress.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIRclub-UdeSA/jar_site](https://github.com/AIRclub-UdeSA/jar_site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
