---
trigger: always_on
description: > Para cualquier **agente de código** que trabaje en este repo (Claude Code, Codex… **NO Grok Build**: ver el muro). Compañero de `CLAUDE.md`. Léelo entero antes de tocar nada.
---

# AGENTS.md — Beyond the Protocol (caso {{TITULAR}})

> Para cualquier **agente de código** que trabaje en este repo (Claude Code, Codex… **NO Grok Build**: ver el muro). Compañero de `CLAUDE.md`. Léelo entero antes de tocar nada.

## Qué es esto
Sistema personal de **{{TITULAR}} {{APELLIDO}}** (ingeniera + paciente de {{DIAGNOSTICO}}) para alcanzar el mejor tratamiento basado en evidencia. **No es software comercial:** es su "gabinete" de IA. El objetivo es ayudarla a ella, con rigor y cuidado.

## Empieza por aquí
- `00_FUENTE-DE-VERDAD/EMPIEZA-AQUI.md` — puerta de entrada.
- `00_FUENTE-DE-VERDAD/COMO-FUNCIONA.md` — cómo está montado todo.
- `00_FUENTE-DE-VERDAD/` — la **fuente de verdad** (todo el contexto). Búscala con `tools/kb.py ask "…"` antes de afirmar nada del caso.

## Reglas innegociables (el "muro") — válidas para TODO agente
- **Apoyo a la decisión, NO consejo médico.** Deciden sus médicos. No reproduzcas cifras clínicas sin verificarlas contra la fuente.
- **Nada hacia fuera sin su OK explícito:** no enviar correos, no publicar, no pagar ni mover dinero, no contactar a su oncóloga **{{CONTACTO}}** hasta confirmar candidatura. Todo queda en **BORRADOR**.
- **Privado por defecto.** No expongas datos clínicos / PII / contraseñas en claro. **Nunca menciones "{{CONTACTO}}"** en nada público.
- {{CONTACTO}} y {{CONTACTO}} son **pares de confianza, no sus médicos**.
- 🚫 **Ningún agente de código que SUBA el repo a un servidor puede correr aquí (regla dura, 25-jul-2026).** Este repo lleva **historial clínico** en su git, así que subir «el workspace» equivale a exfiltrar el informe de una paciente identificable: es el peor egress que este sistema puede sufrir. Y no es hipotético — **Grok Build lo hacía**: el CLI `grok` v0.2.93 subía **el repositorio completo, con todo el historial de git**, a un bucket de xAI (`grok-code-session-traces`) **al margen de qué ficheros leyera el agente**, y transmitía verbatim el contenido de los ficheros leídos, incluido un `.env`; el interruptor «Improve the model» **no** lo desactivaba. Está documentado con captura de tráfico real y bundles clonables (gist de `cereblab`, 12-14 jul-2026, verificado el 25-jul: el gist existe y su contenido coincide; recogido también por The Hacker News). xAI abrió el harness el 15-jul (`github.com/xai-org/grok-build`, existe) y, según reportes secundarios, desactivó la subida en servidor — **pero el código de subida sigue en el binario y se puede reactivar desde el servidor sin actualizar el cliente** (esto último: sello *inferido*, no verificado por mí).
  **Qué significa en la práctica:** Grok Build queda **FUERA de este repo** hasta que (1) corra con la subida desactivada de forma **verificable** (flags de config/entorno, comprobado mirando el tráfico, no confiando en un interruptor de la interfaz) y (2) alguien lo audite (`acceso-herramientas` + `verificacion`). Lo mismo vale para cualquier otro agente o extensión que empaquete el workspace «para mejorar el modelo». Si un agente necesita contexto, se le da el fichero concreto, no el repo. **Comprobado el 25-jul-2026: no hay ni rastro de que Grok Build se haya ejecutado en esta máquina** (sin binario `grok`, sin `~/.grok*`, sin coincidencias en el historial de shell) y **el repo no tiene remoto**.

## Herramientas (`tools/`)
`kb.py` (buscador RAG) · `archivar.py` (Descargas→fuente de verdad) · `transcribe_audios.py` (voz→texto) · `enviar_hoy.py` (Telegram) · `grok.py` (Grok API). **Reindexa** con `python3 tools/kb.py index` tras añadir ficheros.

## Cómo coordinarte con los demás agentes
- Compartimos **esta carpeta = el hub**. **No edites a la vez que otro agente el mismo fichero.**
- Si el repo tiene **git**, trabaja en una **rama** y deja el historial limpio (no toques `main` directo).
- Lo que produzcas, **déjalo en la fuente de verdad e indézalo** (`kb.py index`).
- Subagentes y rutinas existentes en `.claude/agents/` y en las tareas programadas — respétalos, no los dupliques.

## Disciplina de edición (lección de la skill `agent-workflow` de Alby, 28/6)
- **Lee antes de escribir.** No edites código que no has leído entero. Para cambios no triviales, plan corto primero (qué tocas, en qué orden, cómo sabrás que funcionó).
- **Edición quirúrgica.** Usa find-and-replace exacto (match con indentación literal); reserva reescribir-fichero-entero para cuando de verdad reescribes todo. **Ante un mismatch de `edit`, re-lee el estado real del fichero antes de reintentar** — no insistas a ciegas.
- **Cambios pequeños y verificables.** No 20 ficheros de golpe; verifica tras cada unidad lógica (build/tests/`test_all.sh`). Avisa antes de comandos con efectos.
- **Déjalo más limpio de lo que estaba** — corrige el typo o la inconsistencia que pillas al pasar, proporcional (no reformatees todo).
- **Sé honesto con el estado.** `git status`/`git diff` antes de declarar éxito; comprobar > suponer, siempre.

## Gotchas técnicos (lo que muerde si no lo sabes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BeyondTheProtocol/polaris](https://github.com/BeyondTheProtocol/polaris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
