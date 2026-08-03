---
trigger: always_on
description: Archivo de referencia para cualquier agente de codificación que trabaje en este proyecto.
---

# CLAUDE.md

Archivo de referencia para cualquier agente de codificación que trabaje en este proyecto.
Lee este archivo completo antes de hacer cualquier cambio.

## Estado del proyecto y arranque

Antes de hacer cualquier cosa, comprueba el estado del repositorio:

1. Lee todos los archivos de `docs/`
2. Comprueba si existe la carpeta `.template/`. Si existe, este repo sigue siendo la plantilla
   sin inicializar: hay andamiaje, todavía no hay proyecto.
3. Si los documentos están vacíos o incompletos (solo tienen comentarios, sin contenido real):
   - No escribas código
   - No rellenes nada todavía
   - Empieza con esta pregunta: "¿Qué quieres construir y para quién?"
   - A partir de la respuesta, haz las preguntas necesarias para completar 
     los documentos de docs/ en este orden: prd.md → business.md → 
     design-system.md → architecture.md → data-model.md → roadmap.md → user-flows.md
   - Confirma con el usuario antes de pasar al siguiente documento
   - Cuando todos estén rellenos, ejecuta la **inicialización del proyecto** (sección
     siguiente) y solo después pregunta: "¿Empezamos a construir?"

4. Si los documentos ya tienen contenido: lee todo lo que haya en `docs/` antes de actuar.
   Si además `.template/` sigue existiendo, la inicialización quedó a medias: avisa al usuario
   y ofrécete a completarla antes de seguir.

---

## Inicialización del proyecto (una sola vez)

Esta plantilla se distribuye con documentación que habla **de la plantilla**, no del proyecto.
En cuanto los documentos de `docs/` estén rellenos, conviértela en el repo de *este* proyecto.
Hazlo por iniciativa propia, sin esperar a que el usuario lo pida.

Puedes lanzar el proceso completo con `/init-proyecto`.

**Checklist de inicialización:**

1. **`README.md`** — reescríbelo entero para el proyecto, a partir de lo que hay en `docs/`.
   Debe explicar el producto, no la plantilla. Estructura sugerida: nombre y descripción de
   una línea, qué problema resuelve, requisitos previos, variables de entorno (referencia a
   `.env.example`), instalación y desarrollo (`pnpm install`, `pnpm dev`), estructura de
   carpetas, cómo contribuir (referencia a `CLAUDE.md` y al protocolo) y estado del proyecto.
2. **`CLAUDE.md`** — rellena los placeholders de este mismo archivo: nombre, descripción,
   estado, stack tecnológico, estructura de carpetas, convenciones de código y "Qué NO hacer".
   Borra los comentarios `<!-- ... -->` que ya no apliquen, esta sección de inicialización
   (deja de tener sentido una vez hecha), el comando `.claude/commands/init-proyecto.md` y las
   referencias a `.template/` del arranque y del protocolo de changelog. El "Protocolo de MCPs"
   se queda: sigue aplicando cada vez que entre una integración nueva.
3. **`LICENSE`** — sustituye `[YEAR]` y `[AUTHOR]` por los valores reales. Pregunta el nombre
   del autor si no lo sabes.
4. **`.env.example`** — deja solo las variables que el stack elegido necesita de verdad.
5. **MCPs** — con el stack ya decidido, pregunta al usuario qué servidores MCP quiere y con qué
   alcance, siguiendo el "Protocolo de MCPs" (o lanza `/mcp-setup`).
6. **`changelog/`** — debe quedar sin entradas heredadas. Crea la primera entrada real del
   proyecto (tipo: Configuración) describiendo la inicialización, y quita de
   `changelog/README.md` la referencia a la plantilla (o borra el archivo).
7. **`mejoras/backlog.md`** — borra el ejemplo comentado y déjalo listo para entradas reales.
8. **`.template/`** — bórrala entera (`rm -rf .template`). Es el historial de la plantilla, no
   del proyecto.
9. **Verificación final** — busca referencias sobrantes:
   `grep -ril "plantilla\|template" . --exclude-dir=.git --exclude-dir=node_modules`.
   Revisa cada resultado y corrígelo si habla de la plantilla en lugar del proyecto.

**Regla general:** después de la inicialización, ningún archivo del repo debe describirse a sí
mismo como plantilla ni explicar cómo usar la plantilla. Toda la documentación habla del
producto que se está construyendo. Si más adelante encuentras un resto de la plantilla en
cualquier archivo, corrígelo en esa misma sesión.

---

## Protocolo de MCPs

Muchos servicios del stack (Supabase, Resend, Stripe, Vercel, Sentry, Figma, Linear…) publican un
servidor MCP que te deja operarlos directamente en vez de trabajar a ciegas. Configurarlos es
decisión del usuario, no tuya: **pregunta, no instales por tu cuenta**.

### Cuándo preguntar

- Al terminar `docs/architecture.md`, cuando el stack ya está decidido (forma parte de la
  inicialización del proyecto).
- Cada vez que se añada una integración nueva al stack más adelante.

Fuera de esos dos momentos, no saques el tema.

### Cómo preguntar

1. **Mira qué hay ya configurado** con `claude mcp list` antes de proponer nada. Si un servidor
   del stack ya está disponible a nivel global, dilo y no propongas duplicarlo.
2. **Averigua qué existe de verdad.** Si no sabes con certeza si un servicio tiene servidor MCP,
   cómo se llama el paquete, qué transporte usa o qué credenciales pide, **búscalo en la
   documentación oficial del servicio antes de proponerlo**. No inventes comandos ni nombres de
   variables: un `claude mcp add` mal copiado deja el proyecto con un servidor que no arranca.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polmarza/project-template](https://github.com/polmarza/project-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
