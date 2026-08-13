---
trigger: always_on
description: Este archivo es el **mapa operativo para un agente de IA** (Claude Code, Codex, Cursor o cualquiera) que esté corriendo **dentro de la carpeta de este repo**. Si un usuario te dice "instalámelo", "guiame" o "ponelo a andar", seguí esto al pie de la letra.
---

# AGENTS.md — Guía para agentes de IA

Este archivo es el **mapa operativo para un agente de IA** (Claude Code, Codex, Cursor o cualquiera) que esté corriendo **dentro de la carpeta de este repo**. Si un usuario te dice "instalámelo", "guiame" o "ponelo a andar", seguí esto al pie de la letra.

> 📖 El tutorial humano detallado vive en [`README.md`](./README.md). Cuando el usuario quiera pasos explicados para leer él, mandalo ahí. Este archivo es para **vos, el agente**.

---

## 🎯 Tu objetivo

Dejar a **Hermes + Ollama corriendo 100% local** en la máquina del usuario y el **chat abierto** en su terminal. Nada más. El repo entrega eso: el agente Hermes de Nous Research conectado a un modelo local servido por Ollama, charlando desde la terminal.

> Este repo se maneja con **comandos `docker compose` directos** (no hay scripts de arranque). Vos podés correr los que NO son interactivos; el login, el selector de modelo y el chat los corre el humano.

---

## 🧭 Cómo comportarte

- Hablá en **español simple y directo**. Asumí que el usuario puede no ser técnico.
- **Una cosa a la vez.** No tires 5 comandos juntos: explicá el paso, esperá, seguí.
- **No inventes comandos.** Usá solo lo que existe en este repo: `docker compose ...` y editar `.env`. Si dudás, leé el archivo (`docker-compose.yml`, `.env.example`) antes de afirmar nada.
- **Sé honesto con los límites.** No prometas que "se hace todo solo". Hay cosas que vos NO podés hacer (ver más abajo) y tenés que decirlo claro y pasarle la posta al humano.
- **No alucines features.** Este repo NO arma Telegram, WhatsApp, Discord, Slack, mail, crons, memoria ni skills. Eso es del **agente Hermes de Nous Research** y se activa adentro de Hermes → derivá a [nousresearch.com](https://nousresearch.com). Este repo solo entrega Hermes + Ollama local + chat en la terminal.
- Cuando algo falle, **leé el error real** (pedí que el humano te pegue lo que vio) y diagnosticá. No adivines.

---

## ✅ Lo que PODÉS ejecutar vos (agente)

Todo esto es **no interactivo** y seguro de correr en tu shell:

- **Detectar el sistema operativo** (`uname -s` en Mac/Linux, o saber que estás en Windows/PowerShell).
- **Chequear que Docker está corriendo:** `docker info`. Si falla, Docker no está abierto → frená (ver límites).
- **Chequear que el repo está completo** (que existan `docker-compose.yml`, `Dockerfile.hermes`, `.env.example`). Si el usuario no clonó bien, ayudalo a clonar.
- **Crear/leer/editar `.env`:**
  - Si no existe: `cp .env.example .env` (Windows: `Copy-Item .env.example .env`).
  - Cambiar el modelo: editar la línea `OLLAMA_MODEL=` (por defecto `qwen3:8b`, ~5GB). Otros: `qwen3:14b` (~9GB), `llama3.1:8b` (~4.9GB), `hermes3` (~4.7GB).
  - **En macOS** (Ollama nativo), poné `OPENAI_BASE_URL=http://host.docker.internal:11434/v1`. En Windows/Linux dejá el default (`http://ollama:11434/v1`).
- **Levantar Ollama (Windows/Linux, modo bundled):** `docker compose --profile bundled up -d ollama` (con GPU NVIDIA: agregá `-f docker-compose.gpu.yml`).
- **Descargar el modelo (Windows/Linux):** `docker compose --profile bundled run --rm pull-model`.
- **Construir la imagen de Hermes:** `docker compose build hermes` (baja e instala Hermes; tarda la primera vez).
- **Montar una carpeta del host (workspace, opcional):** descomentar `WORKSPACE_PATH=./workspace` en `.env` **Y** la línea del volumen `- ${WORKSPACE_PATH:-./workspace}:/workspace` bajo el servicio `hermes` en `docker-compose.yml`.
- **Leer logs y diagnosticar:** `docker compose --profile bundled ps`, `docker compose --profile bundled logs ollama`, leer la salida que el humano te pegue.

---

## 🛑 Lo que NO podés hacer — FRENÁ y pedíselo al humano

Cuando llegues a una de estas, **pará, explicá qué falta y pedile al humano que lo haga él**:

1. **Instalar Docker Desktop y (en Mac) la app de Ollama.** Son **instaladores gráficos** (ventanas, aceptar términos, contraseña). Vos no clickeás ventanas. Pasale el link y esperá:
   - Docker Desktop → https://www.docker.com/products/docker-desktop/
   - Ollama (Mac) → https://ollama.com/download/mac

2. **Configurar el modelo local y chatear (TUI interactivos):** `docker compose run --rm hermes "hermes model"` (elegir **"Custom endpoint"**, pegar la URL de Ollama, elegir el modelo — todo con flechas + ENTER) y `docker compose run --rm hermes` (chat). Son interfaces interactivas en la terminal → las corre el humano. **NO hay login ni cuenta:** el flujo es 100% local; Hermes apunta directo a Ollama.

### ⚠️ Lo más importante de todo

Los comandos **`hermes model` y `hermes chat`** (los `docker compose run --rm hermes ...`) son **INTERACTIVOS**: esperan ENTER, abren el selector con flechas y arrancan el chat.

Si vos los corrés en un shell **capturado / sin terminal real** (como el Bash tool típico de un agente), esas partes **no funcionan**: no vas a poder apretar ENTER, mover las flechas ni chatear.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fcori47/dios-ia-hermes-local](https://github.com/fcori47/dios-ia-hermes-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
