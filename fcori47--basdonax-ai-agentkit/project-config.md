---
trigger: always_on
description: Este archivo lo leen solos los agentes de programación cuando abren el
---

# AGENTS.md — contexto para agentes de IA

Este archivo lo leen solos los agentes de programación cuando abren el
proyecto: **Codex, Claude Code, Cursor, Devin, Jules** y cualquier otro que
siga la convención `AGENTS.md`. Está para que entiendan el repo sin que se lo
tengas que explicar cada vez.

Si sos una persona: leé el `README.md`, es el que está escrito para vos.

---

## Qué es esto

**Agent Kit.** Un agente de IA conversacional que arranca en la máquina
del usuario, sin servidor. Telegram lo atiende desde ahí; WhatsApp, desde un
servidor, con Chatwoot en el medio. Funciona con Claude, OpenAI o Gemini,
intercambiables desde el `.env`.

Construido sobre **LangChain + LangGraph**. La memoria son los *checkpointers*
de LangGraph, indexados por `thread_id`.

Se armó por etapas: primero local, después Telegram, después WhatsApp. Todo
lo que se diseñó acá apunta a que cada etapa nueva no obligue a reescribir
el agente.

**Idioma del código: español.** Nombres de funciones, variables, comentarios,
docstrings y mensajes de error, todo en español rioplatense (voseo: *tenés*,
*podés*, *guardás*). Excepciones: los identificadores que vienen de librerías
(`messages`, `thread_id`, `checkpointer`, `StateGraph`) y los nombres de los
proveedores. **Si escribís código nuevo acá, seguí esa convención.**

---

## Estructura

El árbol de archivos está en el **[README](README.md#qué-hay-adentro)**.
Lo que importa acá es qué hace cada uno:

| Archivo | Qué resuelve |
|---|---|
| `agente.py` | **El agente.** El grafo de LangGraph. Empezá por acá. |
| `herramientas.py` | Lo que el agente puede hacer además de conversar. Hoy: el clima. |
| `modelos.py` | Crea el modelo y le pregunta al proveedor cuáles tiene |
| `memoria.py` | Los checkpointers: `ram` / `sqlite` / `postgres` |
| `prompts.py` | Lee y guarda `prompts/sistema.md` |
| `respuesta.py` | Parte la respuesta en varios mensajes, o la deja en uno (`MENSAJES_POR_RESPUESTA`) |
| `frenos.py` | El tope de mensajes por conversación y por día |
| `avisos.py` | El mail al dueño cuando algo se rompe: por Google (lo recomendado) o por SMTP |
| `gmail.py` | Habla con Google: el permiso de Google Cloud y la Gmail API. Solo biblioteca estándar y **sin importar nada del paquete**: `conectar_gmail.py` lo carga suelto |
| `consola.py` | Que la terminal de Windows no rompa con las tildes |
| `config.py` | Lee el `.env`. Única fuente de configuración. |
| `canales/base.py` | La forma de un canal |
| `canales/telegram.py` | **El bot de Telegram.** Polling, corre en tu máquina. |
| `canales/chatwoot.py` | **El canal de WhatsApp**, con Chatwoot en el medio |
| `canales/buffer.py` | Junta la ráfaga de mensajes cortos y contesta una vez. Una parte puede ser una tarea que todavía se está leyendo (una foto): se espera en su lugar |
| `canales/adjuntos.py` | **Lo que no es texto, a texto:** audios (OpenAI), fotos y stickers (el modelo del agente), ubicaciones, contactos, archivos, videos y citas |
| `canales/whatsapp_meta.py` | El visto azul y el «escribiendo…» en el celular de la persona, pidiéndoselos a Meta (opcional) |
| `web/webhook.py` | **El servidor que atiende WhatsApp.** Es lo que corre en producción. |
| `../webhook_chatwoot.py` | El punto de entrada del webhook |
| `../Dockerfile` | Empaqueta el **webhook** (`webhook_chatwoot.py`); el bot de Telegram queda adentro por si lo querés correr |
| `web/app.py` | La plataforma de pruebas (FastAPI + un solo HTML) — **no es** el webhook |
| `../conectar_gmail.py` | Conecta la cuenta de Google que manda los avisos. Se corre una vez, **en la computadora** de la persona (abre el navegador), y deja los tres `GMAIL_*` en el `.env` sin mostrarlos |
| `../probar_mail.py` | Manda un mail de prueba de los avisos. Es el último paso de la instalación, y se corre **en el servidor** |
| `../n8n/format_chain_v4.js` | La Format Chain nueva, para el que tiene el agente en n8n y la pega a mano. **Es copia exacta** de la que usa el actualizador: un test lo cuida |
| `../.claude/skills/actualizar-agente-whatsapp/` | El actualizador: una skill de Claude Code que adapta al cobro de Meta un agente hecho en n8n, en este kit o que todavía no existe. Sus pruebas (contra un n8n de mentira) están en `tests/actualizador/` |
| `../docs/img/` | Las imágenes del README. Se generan aparte; no son parte del agente |

---

## Las cuatro decisiones de diseño

Entender esto evita romper cosas:

**1. El agente recibe texto y devuelve texto.**
No sabe si lo llaman desde la terminal, la web, Telegram o WhatsApp. Esa
frontera es deliberada: es lo que permite agregar canales sin tocarlo.
`Agente.responder(texto, conversacion) -> Respuesta`.

**2. La memoria es un checkpointer intercambiable.**
`ram()` / `sqlite()` / `postgres()` en `memoria.py`. El `MODO` del `.env`
elige. Cambiar dónde se guardan las conversaciones no toca `agente.py`.

**3. El prompt del sistema vive en un archivo, no en el código.**
`prompts/sistema.md`, leído en **cada** mensaje (no una vez al arrancar).
Por eso se puede editar con el agente corriendo.

**4. Toda la configuración sale del `.env`, vía `config.py`.**
Ninguna credencial en el código, ni una. Las claves se leen únicamente en
`config.py`.

---

## Dónde tocar cada cosa

| Querés… | Archivo | Cómo |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fcori47/basdonax-ai-agentkit](https://github.com/fcori47/basdonax-ai-agentkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
