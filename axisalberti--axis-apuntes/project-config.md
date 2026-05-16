---
trigger: always_on
description: Este repositorio contiene material educativo para módulos de formación profesional. La documentación está dirigida a **alumnado** y debe ser **didáctica, clara y pedagógica**.
---

# Guía del Repositorio para Agentes

Este repositorio contiene material educativo para módulos de formación profesional. La documentación está dirigida a **alumnado** y debe ser **didáctica, clara y pedagógica**.

## Skills útiles para este proyecto

Este repositorio está orientado a la generación de contenido educativo. Solo se consideran útiles las skills que ayudan a crear, mejorar o enriquecer materiales didácticos.

- **imagegen**: Usar cuando se necesiten imágenes rasterizadas para apuntes, portadas, recursos visuales, mockups o ilustraciones educativas.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/.system/imagegen/SKILL.md`
- **openai-docs**: Usar únicamente cuando se creen contenidos educativos relacionados con productos, APIs o modelos de OpenAI y sea necesario consultar documentación oficial actualizada.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/.system/openai-docs/SKILL.md`
- **clonezilla-helper**: Usar cuando se generen apuntes, prácticas o guías educativas sobre Clonezilla, clonación de discos, creación de imágenes, restauración o despliegue en laboratorio.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/clonezilla-helper/SKILL.md`
- **content-research-writer**: Usar cuando se necesite crear, ampliar o revisar contenidos educativos con estructura didáctica, mejores introducciones, citas, esquemas y revisión por secciones.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/content-research-writer/SKILL.md`
- **research**: Usar cuando se necesite investigación estructurada con validación en varias fuentes y referencias completas para apuntes, unidades o recursos técnicos.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/research/SKILL.md`
- **images-search**: Usar cuando se necesiten buscar imágenes reales en la web para incluirlas en documentación educativa. Requiere `BRAVE_SEARCH_API_KEY`.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/images-search/SKILL.md`
- **web-search**: Usar cuando se necesite buscar, contrastar o extraer información web actualizada para enriquecer contenidos educativos, verificar conceptos técnicos o recopilar fuentes de referencia.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/web-search/SKILL.md`
- **scrape**: Usar cuando se necesite extraer contenido de páginas concretas o listas de URLs para convertirlo en Markdown, HTML o JSON reutilizable en documentación educativa.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/scrape/SKILL.md`
- **mkdocs**: Usar cuando se modifique la estructura del sitio, navegación, configuración, tema Material, plugins, búsqueda o publicación de la documentación MkDocs.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.codex/skills/mkdocs/SKILL.md`
- **frontend-design**: Usar cuando se creen o mejoren recursos visuales web del proyecto, páginas HTML, componentes, portadas o interfaces educativas con diseño cuidado y responsive.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/frontend-design/SKILL.md`
- **navigation-menu-generator**: Usar cuando se reorganice la navegación del sitio, menús, estructura de unidades, accesos a recursos o enlaces principales del módulo.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/navigation-menu-generator/SKILL.md`
- **html-ppt**: Usar cuando se necesiten crear presentaciones HTML estáticas tipo PPT para contenidos educativos, con temas, plantillas, animaciones, navegación por teclado y modo presentador. Es especialmente útil para generar slides de unidades didácticas, exposiciones de aula o materiales de apoyo visual.  
  Ruta: `/Users/josemanuelgonzalezcastillo/.agents/skills/html-ppt/SKILL.md`

## Normas de redacción de apuntes

- Los apuntes deben ser claros, didácticos y orientados al aprendizaje del alumnado.
- No se permitirán faltas de ortografía en ningún contenido generado o editado.
- Todas las frases y todas las preguntas deben comenzar por mayúscula.
- Se deben seguir las reglas ortográficas del español (acentuación, puntuación y uso correcto de signos de interrogación y exclamación de apertura y cierre).
- Cada vez que se edite un fichero `.md`, se añadirá al final una línea con la fecha de actualización, con el formato: `**Fecha de actualización:** 31/01/2026`.
- Las cuestiones generadas se guardarán en un fichero en la raíz del repositorio. El nombre del fichero debe incluir el nombre del módulo y el nivel de dificultad.
- Cuando se solicite generar cuestiones, se generará directamente el fichero `.gift` final; no se crearán módulos ni scripts en Python para esa tarea.
- Siempre se generarán 30 cuestiones de tipo cuestionario en formato GIFT.
- Al finalizar cada generación de cuestiones, se validará el fichero con las reglas incluidas en este AGENTS.md.
- En cuestiones tipo test y de desarrollo, no se incluirán preguntas que obliguen a memorizar datos numéricos concretos (por ejemplo, nº de núcleos, tasas, latencias u otros valores específicos de dispositivos o conceptos).
- En preguntas tipo test y de desarrollo, los enunciados deben empezar por ¿ y terminar en ? y respetar tildes y ortografía correcta.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AxisAlberti/Axis_Apuntes](https://github.com/AxisAlberti/Axis_Apuntes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
