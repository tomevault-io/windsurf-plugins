---
trigger: always_on
description: Guía para agentes (y humanos) que contribuyen a este repo. Es un proyecto
---

# AGENTS.md — Angular Signals (Interactive Introduction)

Guía para agentes (y humanos) que contribuyen a este repo. Es un proyecto
educativo de **Angular 22**: una sola app standalone, signal-first, `OnPush` en
todos lados, estilada con Tailwind. No es una librería ni un monorepo.

## Cómo trabajar

- Cambios chicos y acotados, que se lean como diffs — no reescrituras masivas.
- **Tests antes que UI**: la lógica de dominio (parsers, helpers, transformación
  de datos en `libs/`) se valida como **función pura sobre fixtures**, sin
  `TestBed` ni navegador. La pantalla se prueba aparte.
- Si tocás lógica de dominio, dejá un test que la cubra.
- Antes de dar por terminado un cambio de código, dejá el build/lint en verde:
  `npm run build`, `npm run lint` y `npm test` sin errores ni warnings nuevos.
- Usá siempre sintaxis Angular moderna: `inject()`, control flow (`@if`/`@for`),
  APIs de signals (`signal`/`computed`/`effect`/`input`/`output`/`model`),
  componentes standalone. Nada de `NgModule`.

## Verificación visual (receta de esta máquina, no re-tantear)

Destilado de la autopsia de sesiones del 09 al 16 de julio 2026: el entorno de
verificación se redescubrió desde cero en 5 sesiones distintas. Esta es la vía
que funciona; no volver a explorar alternativas ya descartadas.

- **NO uses el Browser pane (`mcp__Claude_Browser__computer` screenshot) con
  esta app.** La animación rAF continua + pestaña oculta lo cuelga (timeout de
  30s, reproducido en 5 sesiones). Máximo 1 intento; si falla, cambiá de vía
  sin insistir y sin devolverle la prueba al usuario.
- **Vía que funciona:** `chrome-devtools` o `claude-in-chrome` (extensión
  conectada). Si `claude-in-chrome` da "not connected", pedile a Damian una
  sola vez que conecte la extensión y seguí por ahí.
- **Dev server:** config `signals-play-dev` en `.claude/launch.json` (ya
  existe, no crear otra). Antes de afirmar que corre, chequeá salud
  (`preview_logs` o curl a `localhost:4200`). Se cae silencioso. Y ojo: un
  error de compilación transitorio corta el HMR de la pestaña del usuario
  aunque el server se recupere; si dice "no veo cambios", primero verificá
  server + bundle, no el código.
- **Falso bug conocido:** la pestaña oculta del pane pausa `rAF` y
  `scroll-behavior: smooth`. Un "bug de navegación/scroll" que solo pasa en el
  pane NO es un bug real: verificá en browser real antes de gastar debugging.
- **Cierre de cambio visual = dos chequeos separados y obligatorios:**
  (1) ¿renderiza sin error? y (2) ¿se ve bien compuesto? El segundo exige
  captura real MIRADA (o el agente `design-reviewer`), nunca solo mediciones
  de DOM. Para colisiones/superposición, además de mirar: medí overlap
  numérico con `getBoundingClientRect()` contra TODOS los vecinos (título,
  contador, dots, órbita), no contra uno solo.
- **Gate de diseño:** todo cambio que toque el motor visual (`molecule-engine`,
  CSS de `integrada-vista/`) cierra con una pasada de `design-reviewer` contra
  `DESIGN-CHECKLIST.md` ANTES de declararlo bueno. La palabra del propio
  agente ("quedó hermoso") no es veredicto.
- **NO verifiques la navegación con eventos SINTÉTICOS** (2026-07-24, costó ~15
  round-trips). `btn.click()` y `dispatchEvent(new MouseEvent/KeyboardEvent)`
  disparan el handler pero `goToUnit` no avanza, y las mediciones carrean con
  las animaciones de scroll: dan FALSOS NEGATIVOS ("el índice no navega") y
  dejan la instancia en un estado inconsistente. Con **input REAL** (click de
  puntero de chrome-devtools/claude-in-chrome, o rueda real) anda a la primera.
  Ojo también con las coordenadas: el screenshot fluctúa (1456/1512/1520 px)
  mientras el viewport real es 1920 → clickeá por `uid`/`ref`, o escalá por
  `screenshotW / innerWidth`, si no le errás al elemento.
- **Al terminar una review, actualizá el `DESIGN-CHECKLIST.md`** con cada defecto
  detectado Y resuelto (es la regla del propio archivo, y es lo que evita que la
  familia de defectos vuelva). Si la review se entregó como artefacto, ese
  artefacto también se actualiza al aplicar los fixes: un reporte que quedó en
  "roto" cuando ya está arreglado desinforma.

## Antes de construir features visuales grandes

- **Alcance completo primero:** si el pedido es "integrar/migrar X", inventariá
  el universo real (p.ej. `app.routes.ts` para los sub-niveles) y confirmá el
  alcance en una frase ANTES de programar el primer caso. Un ejemplo-muestra no
  define el alcance.
- **Referencia visual externa** (neal.fun, ncase.me, etc.): desambiguá elemento
  por elemento qué se traslada, con `AskUserQuestion` si hace falta.
- **"Que se sienta X" se traduce a lista escrita de anti-patrones** antes de
  implementar (ver la sección de gramática de modal en `DESIGN-CHECKLIST.md`).
- **Syncs bidireccionales** (URL↔estado, navegación↔UI): especificá y testeá
  AMBAS direcciones antes de declarar cerrado; un test mínimo por dirección.

## Mapa del repo → scope de commit

Cada área de `src/app/` mapea a un scope de commit. Usá el scope del área que
realmente tocás; no inventes scopes nuevos.

| Directorio                 | Qué contiene                                                                          | Scope        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damiansire/angular-signals-introduction](https://github.com/damiansire/angular-signals-introduction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
