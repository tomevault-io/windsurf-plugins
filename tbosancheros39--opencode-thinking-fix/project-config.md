---
trigger: always_on
description: Plugin + proxy + watchdog that preserve reasoning content across multi-turn OpenCode conversations with DeepSeek, Kimi, GLM, MiMo, MiniMax, and OpenCode Go.
---

# opencode-thinking-fix

Plugin + proxy + watchdog that preserve reasoning content across multi-turn OpenCode conversations with DeepSeek, Kimi, GLM, MiMo, MiniMax, and OpenCode Go.

## Problem

Reasoning models emit chain-of-thought in provider-specific fields (`reasoning_content`, `reasoning`, or `thinking` blocks). OpenCode and other coding clients drop those fields when serializing subsequent turns. Providers require the reasoning to be passed back — some reject requests without it, others silently lose prior context. Either outcome degrades multi-turn coherence and tool-call accuracy. This repository preserves the fields and replays the actual reasoning text.

## Solution, Three Independent Layers

1. **Plugin** (`opencode-thinking-fix-universal.ts`): self-detection guard that injects `reasoning_content: ""` into assistant messages to prevent 400s. Works even if proxy is down.
2. **Proxy** (`proxy.js`): intercepts API traffic, caches real reasoning from SSE streams, injects it back on subsequent turns. One runtime dependency (`eventsource-parser`).
3. **Watchdog** (`watchdog.sh`): checks proxy health every 4 minutes, restarts if down.

## Architecture

- Port **3457**: model-based routing (model prefixes → upstream APIs)
- Port **3458**: fixed upstream to OpenCode Go (`https://opencode.ai/zen/go/v1`)
- Port **3459**: fixed upstream to OpenCode Zen (`https://opencode.ai/zen/v1`); set `REASONING_KEY=reasoning_content`
- Port **3462**: fixed upstream to OpenRouter (`https://openrouter.ai/api/v1`); set `REASONING_KEY=reasoning_content`

## File Layout

```
<project-root>/
├── plugins/opencode-thinking-fix-universal.ts   → copy to ~/.config/opencode/plugins/
├── proxy/core.js                                → pure proxy logic (required)
├── proxy/proxy.js                               → run with node
├── watchdog/watchdog.sh                         → auto-recovery script
├── systemd/
│   ├── reasoning-cache.service                  → port 3457 systemd unit
│   ├── reasoning-cache-go.service               → port 3458 systemd unit
│   └── reasoning-proxy-watchdog.service         → watchdog systemd unit
└── tests/
    ├── test-plugin.js                           → 12 plugin tests
    └── test-proxy.js                            → 127 proxy tests
```

## Prerequisites

- Node.js (any recent version, proxy uses `eventsource-parser` as its only dep)
- OpenCode v1.17.9+ (for plugin `.ts` compilation support)
- For direct providers: valid API keys in environment (`DEEPSEEK_API_KEY`, etc.)
- For OpenCode Go: `OPENCODE_GO_API_KEY` or `~/.local/share/opencode/auth.json`
- systemd user services (optional, requires D-Bus user session)
- curl (for health check verification)

## Installation

### Step 1: Install the Plugin

```bash
mkdir -p ~/.config/opencode/plugins
cp plugins/opencode-thinking-fix-universal.ts ~/.config/opencode/plugins/
```

Plugin auto-loads from `~/.config/opencode/plugins/`. No `opencode.json` config needed. OpenCode compiles `.ts` at startup.

### Step 2: Install and Start the Proxy

```bash
mkdir -p ~/reasoning-cache-proxy
cp proxy/core.js ~/reasoning-cache-proxy/
cp proxy/proxy.js ~/reasoning-cache-proxy/

# Start proxy for direct providers (port 3457)
node ~/reasoning-cache-proxy/proxy.js &

# OR via systemd:
cp systemd/reasoning-cache.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now reasoning-cache.service

# Start proxy for OpenCode Go (port 3458)
PORT=3458 UPSTREAM_URL=https://opencode.ai/zen/go/v1 node ~/reasoning-cache-proxy/proxy.js &

# OR via systemd:
cp systemd/reasoning-cache-go.service ~/.config/systemd/user/
systemctl --user enable --now reasoning-cache-go.service
```

### Windows installation

The plugin and proxy run natively on Windows. The Bash watchdog and systemd
units do not; use Task Scheduler or NSSM for automatic proxy restarts.

Open PowerShell and install Node.js LTS if it is not already installed:

```powershell
winget install OpenJS.NodeJS.LTS
node --version   # Node 18 or newer
npm --version
```

From the cloned project directory, install dependencies and copy the plugin:

```powershell
npm ci
$pluginDir = Join-Path $env:APPDATA 'OpenCode\plugins'
New-Item -ItemType Directory -Force $pluginDir | Out-Null
Copy-Item .\plugins\opencode-thinking-fix-universal.ts $pluginDir -Force
```

Create a private runtime directory and copy **both** proxy files. `core.js`
is required by `proxy.js`:

```powershell
$proxyDir = Join-Path $env:LOCALAPPDATA 'OpenCode\reasoning-cache-proxy'
New-Item -ItemType Directory -Force $proxyDir | Out-Null
Copy-Item .\proxy\core.js, .\proxy\proxy.js $proxyDir -Force
```

Start the proxy in the current PowerShell window:

```powershell
$env:PORT = '3457'
node (Join-Path $proxyDir 'proxy.js')
```

For OpenCode Go, use a second PowerShell window:

```powershell
$env:PORT = '3458'
$env:UPSTREAM_URL = 'https://opencode.ai/zen/go/v1'
node (Join-Path $proxyDir 'proxy.js')
```

The global OpenCode configuration is `%APPDATA%\OpenCode\opencode.json`; a
project configuration is `.opencode\opencode.json`. Set the provider base URL
to `http://127.0.0.1:3457/v1` (or port `3458` for OpenCode Go), then restart

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tbosancheros39/opencode-thinking-fix](https://github.com/tbosancheros39/opencode-thinking-fix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
