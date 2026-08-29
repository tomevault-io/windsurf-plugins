---
trigger: always_on
description: Widget de escritorio Windows (Electron) estilo Clippy que muestra el uso de
---

# Claude Usage Pet — estado del proyecto

Widget de escritorio Windows (Electron) estilo Clippy que muestra el uso de
Claude (límite de 5h, semanal, créditos) con una mascota flotante
personalizable. No oficial, no afiliado a Anthropic. Funciona con Claude
Code O con la app de escritorio de Claude "normal" (ver "Fuente de datos"
más abajo) — no hace falta ser desarrollador para usarlo. Plan original en
`C:\Users\f_via\Documents\FV\Job Search 2026\Claude Usage Pet\plan-claude-usage-pet.md`.

Repo público: https://github.com/fvialeleyva/claude_usage_pet (MIT). Local
y GitHub están sincronizados vía `git remote origin` + `gh` CLI (instalado
por Claude Code con `winget install --id GitHub.cli`, autenticado como
`fvialeleyva`). Primer release (`v0.1.0`, marcado pre-release por no estar
firmado) ya está publicado con el instalador de la Fase 5.

## Cómo correr la app

```bash
cd "C:\Users\f_via\Documents\FV\Vibe-Coding\claude-usage-pet"
npm start
```

Es dev-mode puro (`electron .`). Para reiniciar limpio durante desarrollo:
matar todos los `electron.exe` primero (`taskkill //F //IM electron.exe`),
porque el lock de instancia única (ver abajo) hace que una segunda
instancia no haga nada si ya hay una viva.

Para generar el instalador (Fase 5, ver sección dedicada más abajo):

```bash
npm run build
```

Produce `dist/Claude Usage Pet Setup 0.1.0.exe` (NSIS, sin firmar todavía)
y `dist/win-unpacked/Claude Usage Pet.exe` (la app ya armada, sin pasar
por el instalador — útil para smoke-test rápido). Matar los
`Claude Usage Pet.exe`/`electron.exe` corriendo antes de un build nuevo.

## Estado por fase

- **Fase 0** — spike de datos: confirmado que se puede leer el % de uso
  desde `~/.claude/.credentials.json` (token OAuth) contra
  `api.anthropic.com/api/oauth/usage` (endpoint no documentado, el mismo
  que usa `/usage` en Claude Code). Ver `spike/check-usage.js`.
- **Fase 1** — tray icon + panel de detalle (clona el formato de barras de
  5h/semanal/créditos).
- **Fase 2** — mascota flotante arrastrable, siempre-encima.
- **Fase 3** — notificaciones nativas al cruzar 50%/90% de cada límite.
- **Fase 4** — personalización (skins + accesorios). **Completa** — 7
  skins funcionando, ver sección dedicada abajo. Solo queda pulir detalles
  menores si Franco los pide (ver "Pendiente / ideas sueltas").
- **Fase 5** — empaquetado. **En progreso**, ver sección dedicada más
  abajo para el detalle de qué está hecho y qué falta (firma de código,
  repo público + auto-updater son las piezas que dependen de que Franco
  tome decisiones/haga trámites — no bloquean lo demás).

## Arquitectura

```
src/
  data/
    usage.js              — capa de datos primaria (Claude Code): lee el
                            token, llama al endpoint, nunca tira excepción
                            sin contexto ({ok:false,...}). Si Claude Code
                            falla por lo que sea, cae sola a
                            desktop-usage.js antes de reportar error — ver
                            sección dedicada "Fuente de datos" más abajo.
    desktop-usage.js       — capa de datos de respaldo (app de escritorio
                            de Claude "normal"): lee un historial local
                            que esa app ya escribe sola, sin token/OAuth.
  main/
    main.js               — proceso principal: tray, todas las ventanas,
                             polling, IPC
    preload.js             — puente contextBridge, compartido por las 3
                              ventanas renderer (pet, panel, customize)
    pet-state.js            — persiste posición/visibilidad de la mascota
                               (%APPDATA%/claude-usage-pet/pet-window-state.json)
    pet-appearance.js       — persiste skin elegido + accesorios
                               (pet-appearance.json, mismo directorio)
    custom-skin.js           — skin propio del usuario (ver sección
                                dedicada en Personalización)
    notifications.js        — umbrales 50/90, con reseteo cuando el % baja
    settings.js              — preferencias de la app (hoy: autostart),
                               pet-settings.json en el mismo directorio
  renderer/                — panel de detalle (click/hover en el tray)
  renderer-pet/             — la mascota flotante en sí
  renderer-customize/        — ventana "Personalizar mascota…"
assets/
  tray-icon.png, tray-icon-warning.png  — ícono del tray (32px)
  icon.ico                — ícono de la app/instalador (multi-resolución,
                             16 a 256px), generado desde app-icon.svg
  app-icon.svg             — fuente del ícono de app: el diseño de Smiley
                             con el color de severidad fijado en el
                             naranja "normal" (no es un skin, es el ícono
                             de marca de la app en sí)
  skins/                  — floppy-o.png, monitor-max.png, forbino-max.png,
                             calc-a-tron.png, action.png, mug.png (más el
                             skin "smiley" que es SVG a mano, sin imagen)
scripts/
  generate-icon.js         — ícono del tray (PNG puro, sin dependencias)
  generate-app-icon.js     — ícono de la app/instalador (`npm run

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fvialeleyva/claude_usage_pet](https://github.com/fvialeleyva/claude_usage_pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
