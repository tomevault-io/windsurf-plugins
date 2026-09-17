---
trigger: always_on
description: Grabador **multicam** de escritorio (macOS / Electron) que captura **pantalla + webcam/micro**
---

# record-studio — guía para Claude Code

Grabador **multicam** de escritorio (macOS / Electron) que captura **pantalla + webcam/micro**
sincronizadas y entrega una carpeta de proyecto lista para que la skill **`video-use`**
(headless Claude Code) componga el vídeo final: planos automáticos (fullcam / fullscreen / pip),
gráficos **HyperFrames**, SFX de la librería de HeyGen y subtítulos.

```
record-studio (graba N clips)  ──►  video-use (decide planos, corta, gráficos, SFX, subs)  ──►  edit/final.mp4 (+ final_9x16.mp4)
```

Lee también `README.md` (uso) y `HANDOFF.md` (contrato del EDL multicam entre grabador y editor).

---

## Arquitectura (tres piezas)

1. **Grabador** — esta app Electron (`electron/`, `src/`). Graba clips y lanza el editor headless.
2. **Editor** — la skill **`video-use`**. Fuente canónica en `~/Developer/video-use`
   (symlink `~/.claude/skills/video-use` → ahí). **Copia vendorizada** en `video-use/` de este repo
   (hay que mantener ambas sincronizadas al tocar `helpers/render.py` o `SKILL.md`).
3. **HyperFrames** — `npm` package `hyperframes` (gráficos/animaciones). Se usan sus **ejemplos
   predefinidos** (`hyperframes init --example <nombre>`), no gráficos hechos a mano.

### Layout del repo
- `electron/main.js` — proceso principal: ventanas, IPC (proyectos, clips por chunks, guiones),
  guardas de ruta (`guardPath`/`guardRoot`: el renderer solo toca carpetas raíz elegidas).
- `electron/agent.js` — `runAgentJob()`: spawn de `claude -p` headless, timeouts, kill de grupo,
  coste, registro de huérfanos, log en `edit/_agent.log`.
- `electron/prompts.js` — todos los prompts (compose/iterate/montageBrief + analyze/generate/
  rewrite/hooks de guiones). Funciones puras, testeadas.
- `electron/media-protocol.js` — `rsmedia://` con Range, restringido a raíces permitidas.
- `electron/uploadpost.js` — cliente de Upload-Post: publica `edit/final*.mp4` en YouTube y
  compañía. La clave (`UPLOAD_POST_API_KEY`) se lee del `.env` en el proceso principal y NO llega
  al renderer; el fichero se transmite con `fs.openAsBlob` para no cargar cientos de MB en memoria.
  La subida es asíncrona: `POST /api/upload` devuelve `request_id` y se consulta en
  `GET /api/uploadposts/status?request_id=`. Los títulos y la descripción con capítulos los escribe
  el agente en `edit/publish.json` (`prompts.publishMetaPrompt`) A PARTIR DEL `.srt` del montaje:
  si los timestamps se inventan, los capítulos de YouTube caen a mitad de frase.
- `electron/settings.js` — ajustes durables en `userData/settings.json`.
- `electron/awake.js` — impide que el Mac apague la pantalla o se bloquee por inactividad
  mientras hay una sesión de grabación (`prevent-display-sleep`, se suelta al terminar o si la
  ventana se va).
- `electron/util.js` — helpers puros (stamp, slugify, parseRange, isUnder, writeJson atómico…).
- `electron/preload.js` — puente `contextBridge` (`window.studio.*`).
- `electron/{float,tp}-preload.js` — overlays flotantes (cámara flotante / teleprompter).
- `src/renderer.js` — núcleo UI (estado, navegación, proyectos, editor/feed, terminal);
  `src/recording.js` — fuentes/dispositivos/blur/crop/MediaRecorder por chunks;
  `src/scripts-view.js` — vista Guiones; `src/wire.js` — cableado (se carga el último).
- `src/campipe.js` — selección del motor de cámara: MatAnyone2 local en Apple Silicon, LiveKit en otros equipos.
- `src/index.html`, `src/styles.css` — UI.
- `src/{float,teleprompter}.{html,js}` — ventanas overlay.
- `_scripts/` — utilidades sueltas.
- **Carpetas de proyecto/grabaciones ignoradas por git** (`avatar-mix*`, `alternativa-a-holded*`,
  `rec_*`, `avatar2`): son datos generados (GBs de media), NO código.

### Estructura de un proyecto grabado
```
<raíz>/<proyecto>/
├── project.json            ← metadatos, lista de clips, agentSession (id de conversación)
├── clips/clip_NN/{screen.webm, webcam.webm, sync.json}
└── edit/                   ← lo escribe video-use (final.mp4, final_9x16.mp4, EDLs, srt, posters)
```
- `screen.webm` = pantalla sin audio. `webcam.webm` = cámara **+ micro** (única fuente de audio).
- `sync.json`: `offset_ms = cam_start − screen_start`, duración y dims reales.

---

## Montaje headless ("Montar vídeo" / "Editar con Claude Code")

Al pulsar **✨ Componer** o iterar, la app hace `spawn` de un **Claude Code headless**:

```
claude -p "<prompt>" --add-dir <projDir> \
  --permission-mode bypassPermissions \
  --output-format stream-json --verbose [--resume <session_id>]
```

- **Modelo**: hereda de `~/.claude/settings.json` (actualmente `"model": "opus[1m]"`,
  `"effortLevel": "high"` → **Opus 4.8 high, contexto 1M**). No está fijado con `--model` en el
  spawn (se puede fijar si se quiere independencia del config global).
- **Prompts**: se construyen en `electron/main.js` → `composePrompt()` (montaje inicial) y
  `iteratePrompt()` (modificaciones). Opciones por defecto en `DEFAULT_OPTS` (`sfx: true`).
- **Continuidad de conversación**: `runAgentJob()` captura `session_id` de los eventos stream-json
  y lo guarda en `project.json` (`agentSession`) vía `saveAgentSession()`. Al editar, el usuario
  puede **continuar la misma conversación** (checkbox "🧠 Continuar…", pasa `--resume <id>`) o

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mutonby/vibetube](https://github.com/mutonby/vibetube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
