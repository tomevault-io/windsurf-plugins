---
trigger: always_on
description: > Última actualización: 2026-06-22 (v1.0.0 — five-CLI release). Actualizar al final de cada sesión.
---

# EcoRelay — Guía del proyecto

> Última actualización: 2026-06-22 (v1.0.0 — five-CLI release). Actualizar al final de cada sesión.

## Qué es

Mensajería entre sesiones de IA en la misma máquina o red. Claude Code, OpenCode, Copilot CLI, Codex CLI, **Antigravity CLI (`agy`)** y **Cursor CLI (`cursor-agent`)** hablan entre sí en lenguaje natural a través de un Hub central. 19 tools MCP. Mensajes persistentes, grupos, salas, federación LAN e internet.

## Cómo funciona (el flujo de un mensaje)

```
Sesión A (CC) dice "envía a backend-api que cambie el token"
    → tool relay_send(to="backend-api", text="...")
    → channel/tools/messaging.ts → envía JSON al Hub vía Unix socket
    → Hub (hub/handlers/messaging.ts) → busca "backend-api" en registry
    → Si online: envía por socket/WS directo + persiste en mailbox
    → Si offline: solo persiste en mailbox (relay_inbox lo recupera)
    → Sesión B (OC o CC) recibe el mensaje como notification
    → El agente receptor lo procesa y puede responder
```

CC se conecta al Hub por **Unix socket**. OC se conecta por **WebSocket** (puerto 19736). Ambos hablan al mismo Hub.

## Arquitectura: las piezas

### 1. Hub (`src/hub/`) — el cerebro

Proceso daemon standalone (`src/hub-daemon.ts`). Uno por máquina. Hace:

- **Registry** (`registry.ts`): quién está conectado, evicción de zombis, probe con ping.
- **Mailbox** (`mailbox.ts`): ring buffer de 500 msgs por peer, persiste en disco.
- **Groups** (`groups.ts`): grupos persistentes estilo WhatsApp con admin.
- **WS endpoint** (`ws-endpoint.ts`): WebSocket en :19736 para OC. Auth por token (`~/.eco-relay/hub-ws-token`, timingSafeEqual, 32 bytes random).
- **Handlers** (`handlers/`): un archivo por dominio (core, messaging, rooms, groups, federation).
- **Bridge** (`bridge.ts`): federación LAN hub-to-hub vía TCP.

Arranca automáticamente: la primera sesión (CC o OC) que abre lo spawnea. Se apaga solo en **10 segundos** sin peers (`idleExitMs`, hub/index.ts:63).

### 2. Channel CC (`src/channel/`) — el plugin de Claude Code

Un MCP server por sesión CC. Registra UNA sesión (la propia) con el Hub. NO hace `session.list()`. Estructura separada: tools/ (implementación), tool-schemas/ (definiciones), routing, reconnect, notifications.

**No se tocó en v0.8.** Si se rompe, CC↔CC deja de funcionar. Tratar como invariante.

### 3. Plugin OC (`src/opencode-plugin/ecorelay.ts`) — el plugin de OpenCode

Un solo archivo (~1400 líneas). Maneja múltiples sesiones OC (una por tab). Incluye:

- Conexión WS al Hub con auth por token
- 19 tools MCP (mismas que CC pero implementación propia)
- Push reactivo: `session.prompt` sin noReply → el agente receptor reacciona
- Auto-spawn del daemon (`spawnHubDaemon`, process.execPath)
- Validación de DAEMON_PATH + env allowlist (no hereda API keys)

### 4. Adaptador Codex CLI (`src/codex-adapter/`) — modular, 7 archivos

MCP server que conecta el Hub con Codex. A diferencia de CC/OC/Copilot, **Codex no tiene gancho de push in-process para terceros** (su daemon nativo es Unix-only; su named-pipe está firmado por OpenAI; no hay inyección vía MCP). Por eso necesita un **launcher** y un app-server aparte.

**Topología (Windows)** — `scripts/ecorelay-codex.cmd` → `ecorelay-codex-launch.ts` arranca DOS procesos:

1. `codex app-server --listen ws://127.0.0.1:PORT` (backend, oculto). **El adapter MCP corre AQUÍ** (los MCP servers de config.toml cargan en el backend, no en el cliente TUI).
2. `codex --remote ws://127.0.0.1:PORT` (TUI cliente). Comparte backend con #1 → el push del adapter llega al hilo del TUI.

El adapter empuja haciendo `turn/start` contra el app-server (al que se conecta como otro cliente WS).

**Gotchas críticos descubiertos en vivo (2026-06-17) — NO romper:**

- **El adapter descubre el puerto del app-server leyendo `~/.eco-relay/codex-appserver.pid`** ({pid, port} que escribe el launcher). NO depende de la env var: **Codex NO propaga el env del proceso al MCP child** — solo pasa lo que esté en `config.toml [mcp_servers.X.env]`. Poner `ECORELAY_CODEX_APP_SERVER` en el spawn NO llega al adapter.
- **El transporte MCP (`server.connect`) se registra ANTES** de conectar al app-server + `tracker.discover()` (que reintenta hasta 30s). Si discover bloquea el arranque → `startup_timeout_sec` (20s) mata el MCP. App-server connect + discovery van en BACKGROUND (IIFE).
- **El adapter se auto-mata al cerrar el padre** (`server.onclose` / `process.stdin` 'end'/'close' → `parent_gone_exiting`). Sin esto queda HUÉRFANO vivo conectado al Hub = peer zombie.
- **config.toml: rutas Windows con comillas SIMPLES** (literal TOML). Comillas dobles → `\U` de `\Users` = escape unicode → parseo roto. (Las entradas nativas tipo node_repl usan comillas simples por esto.)
- **Cold-start**: codex recién abierto no tiene hilo hasta que el usuario teclea → push retenido (no perdido) hasta que el poll (60s) descubre el hilo → `onThreadChanged` llama `notifyThreadAvailable` para vaciar el buffer. Compartido con OC.

**Sonda research** (`src/channel/codex-beta-ping.ts`): probe env-gated (`ECORELAY_CODEX_BETA_PING`) que usa `elicitInput` para investigar push MCP. No-op por defecto. Es del channel CC, de la fase de plan.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josortmel/EcoRelay](https://github.com/josortmel/EcoRelay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
