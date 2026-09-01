---
trigger: always_on
description: Este archivo es para el próximo agente (o humano) que retome el trabajo en
---

# AGENTS.md — guía de traspaso para el siguiente agente

Este archivo es para el próximo agente (o humano) que retome el trabajo en
este repo sin haber estado presente en las conversaciones anteriores. No
repite lo que ya está en `CLAUDE.md` (arquitectura, convenciones, comandos);
esto es específicamente **qué se hizo, por qué, y qué falta**, para no volver
a derivar el mismo contexto desde cero.

Si algo de lo escrito acá deja de ser cierto (una migración que se completó,
un pendiente que se resolvió), **actualiza este archivo en el mismo commit**
que lo cambie. Un AGENTS.md desactualizado es peor que no tenerlo.

## Cómo trabaja el usuario (léelo antes de tocar nada)

- Habla en español mexicano, directo, a veces con erratas de tecleo rápido
  ("son andie" = "sin nadie", etc.) — no le pidas que aclare, interpreta por
  contexto y confirma con lo que construiste.
- Manda **capturas de pantalla anotadas** (flechas, círculos, colores) como
  especificación. Míralas con cuidado línea por línea: el detalle que
  importa suele estar en una esquina de la imagen, no en el texto que la
  acompaña.
- Cuando dice "continua" o "ok", es luz verde para seguir con el plan que ya
  se explicó — no vuelvas a preguntar.
- Es exigente con el acabado visual ("estamos en 2026, no eso básico de los
  90") y con que las cosas funcionen de verdad, no que aparenten funcionar.
  Prueba cada cambio de UI describiendo lo que hiciste con evidencia (HTML
  renderizado, simulación en Node, captura), nunca solo "ya debería
  funcionar".
- **No subas nada a GitHub sin que lo pida explícitamente.** Commits locales
  sí, `git push` no, salvo que el usuario lo pida con esas palabras ("sube",
  "guarda y sube") — en esta sesión pasó exactamente una vez, después de
  varias tandas de commits locales, y ahí sí se subió core + el plugin de
  Spoolman. Que ya se haya subido una vez no es autorización permanente:
  vuelve a preguntar (o a esperar la palabra) la próxima vez.
- El servidor de producción real corre en `/home/jcjc/nopal` (fuera de
  cualquier worktree). Reiniciarlo es `sudo systemctl restart nopal.service`
  — el agente no tiene sudo interactivo, así que **quien lee esto tiene que
  pedirle al usuario que lo corra**, no intentarlo con `Bash`.

## Flujo de trabajo establecido en esta sesión

1. Se trabaja en un worktree (`.claude/worktrees/nopal-intelligence-layer-*`
   o el que corresponda), nunca directo en `/home/jcjc/nopal`.
2. Cada cambio se prueba (pytest para Python; para JS/CSS, `node --check`
   más una simulación real del render en Node cuando el cambio es de UI —
   ver ejemplos de esta técnica en el historial de commits, es el patrón
   `require('.../app.js')` con un DOM de mentira armado a mano).
3. Se sube el cachebuster `?v=N` de **cada** archivo estático que cambió
   (`app.js`, `style.css`, `translations.js`) en `backend/templates/index.html`
   — si no, el navegador sirve la copia cacheada y el usuario ve "no cambió
   nada" aunque el código sí cambió.
4. Commit en el worktree con mensaje largo explicando el *porqué*, no el
   *qué* (el diff ya dice el qué).
5. `git -C /home/jcjc/nopal merge --no-ff <rama-del-worktree>` para llevarlo
   a producción. En algún punto de esta sesión hacía falta un `git stash`
   porque el usuario tenía cambios sin commitear en `style.css`/
   `index.html` (el reordenamiento de la barra SD del láser) — **eso ya se
   resolvió y está commiteado**, así que los merges recientes son avance
   directo sin stash. Si vuelve a aparecer un working tree sucio en
   `/home/jcjc/nopal`, revisa primero de qué se trata antes de tocarlo
   (puede ser trabajo nuevo del usuario, no el mismo caso de antes).
6. Verificar el merge con `pytest` corrido *dentro de* `/home/jcjc/nopal`
   (no solo en el worktree) y confirmando que `index.html` en producción
   trae los `?v=` nuevos.
7. Nunca se le pide al usuario reiniciar sin haber verificado 1-6 primero.

**Nota de entorno**: no des por sentado que `node`/`pytest` están en el
`PATH` del worktree — en algún entorno de ejecución de esta sesión no lo
estaban. `pytest` vive en `/home/jcjc/nopal/.venv/bin/pytest` (el venv de
producción; el worktree no tiene uno propio, se puede correr contra ese
mismo sin problema ya que solo lee el repo). Un `node` suelto apareció en
`~/.local/nodejs/bin/node` — sirve para `node --check` y las simulaciones
de render. Si ninguno de los dos aparece, dilo explícitamente en vez de
saltarte la verificación en silencio.

## Qué es NOPAL Intelligence (la capa de IA)

Todo lo construido en esta sesión gira alrededor de esto. Resumen rápido:

- Capa de IA **opcional y apagada por omisión**. `backend/services/ai_*.py`
  (config, provider, tools de solo lectura, actions con confirmación,
  router multi-modelo, conversaciones).
- **Regla de oro que no se negocia**: láser y CNC nunca arrancan por esta
  vía. Hay un test que falla si alguien agrega esa herramienta.
- Cada herramienta que la IA puede llamar (`get_workshop_status`,
  `get_library`, `get_led_matrix`, etc.) tiene que corresponder a un dato
  real medido — nunca se le da a la IA algo que no pueda contestar de
  verdad. Esto es relevante para el trabajo más reciente (la cortinilla de
  comandos, ver abajo): cada chip sugerido se apoya en una tool o action que

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charlymigenes-ux/nopal](https://github.com/charlymigenes-ux/nopal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
