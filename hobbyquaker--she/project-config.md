---
trigger: always_on
description: **she** (smart-home-engine) is a Node.js CLI daemon (`she`) that loads user `.js` scripts into a sandboxed VM, connects to an MQTT broker, and exposes a web UI for editing scripts and browsing device state. It targets smart-home environments.
---

# GitHub Copilot Instructions

## Project Overview

**she** (smart-home-engine) is a Node.js CLI daemon (`she`) that loads user `.js` scripts into a sandboxed VM, connects to an MQTT broker, and exposes a web UI for editing scripts and browsing device state. It targets smart-home environments.

- **Binary**: `she` (npm package `she`, installs globally)
- **Entry point**: `src/index.js` (CommonJS, runs as a daemon)
- **Web frontend**: `web/` — Svelte 5 + Vite 6 + TypeScript, package `she-her`
- **Config**: `src/config.js` (yargs v17, `.parseSync()`); config.json loaded from `~/.she/config.json` by default
- **Active branch**: `main`

## Stack

| Concern | Library |
|---------|---------|
| MQTT client | mqtt v5 |
| File watching | chokidar v4 (`usePolling: true` required for WSL2/NTFS paths) |
| Scheduling | node-schedule v2 |
| Solar events | suncalc |
| Logging | pino v9 + pino-pretty v13, `colorize: true`, `sync: true` (same-thread stream, not worker-thread transport) |
| CLI args | yargs v17 |
| HTTP server | Express v5 |
| WebSocket | ws v8 |
| Frontend | Svelte 5 + Vite 6 + TypeScript |
| Matter | @matter/main v0.17 |
| Database | sheDB (built-in JSON document store, `src/web/shedb.js`) |
| Cache | ioredis v5 (optional Redis write-through) |
| Time series | @influxdata/influxdb-client v1 (optional) |
| Search | @elastic/elasticsearch v9 (optional) |

## Sandbox API Surface

Scripts run in a VM sandbox and receive a `she` object:

### MQTT (primary interface)
- `she.mqtt.sub(topic, [options], callback)` — subscribe
- `she.mqtt.pub(topic, payload, [options])` — publish
- `she.mqtt.get(topic)` → current value
- `she.mqtt.link(src, target, [transform])` — forward value changes
- `she.mqtt.getProp(topic, ...props)` — read state property (`val`, `ts`, `lc`)
- `she.mqtt.age(topic)` → seconds since last change

### Scheduling
- `she.schedule(pattern, [options], callback)` — cron string, Date, or suncalc event name (e.g. `'sunrise'`, `'sunset'`); options: `shift` (seconds offset), `random` (random delay in seconds)

### sheDB (document store)
- `she.db.get(id)` → document or undefined
- `she.db.set(id, doc)` — create/overwrite
- `she.db.extend(id, partial)` — deep merge
- `she.db.delete(id)`
- `she.db.prop(id, method, prop, val)` — nested property mutation (`method`: `'set'|'create'|'del'`)
- `she.db.sub(pattern, callback)` — subscribe to document changes (MQTT wildcard pattern)
- `she.db.query(filter, mapFn, [reduceFn])` → Array (ad-hoc synchronous query)

### Matter
- `she.matter.sub(nodeId, endpointId, clusterName, attrName, cb)` → listenerId
- `she.matter.unsub(listenerId)`
- `she.matter.get(nodeId, endpointId, clusterName, attrName)` → Promise\<value\>
- `she.matter.send(nodeId, endpointId, clusterName, command, [args])` → Promise\<result\>

### Stdlib helpers
- `she.mqtt.link(src, target, [transform])` — forward value changes; `target` is topic string or array
- `she.mqtt.or(srcs[], topicOrCb)` — publish 1 if any source truthy, else 0
- `she.mqtt.and(srcs[], topicOrCb)` — publish 1 if all sources truthy, else 0
- `she.mqtt.max(srcs[], topicOrCb)` — publish maximum of source values
- `she.mqtt.min(srcs[], topicOrCb)` — publish minimum of source values (0 if no values)
- `she.mqtt.timer(src, ms, topicOrCb)` — publish 1 for `ms` after `src` goes truthy, then 0
- All `topicOrCb` params accept a topic string or `callback(topic, val)`
- `she.getValue(topic)` / `she.setValue(topic, val)` / `she.getProp(topic, ...props)` — legacy MQTT helpers
- `she.now()` → ms since epoch
- `she.debug/info/warn/error(...args)` — structured logging (prefixed with script name)
- `she.global` — shared mutable object across all scripts
- `she.http.fetch(url, [opts])` → Promise — HTTP/HTTPS fetch; auto-parses JSON by Content-Type; throws on non-OK status
- `she.http.sub(path, callback)` — register a POST webhook at `/api/<scriptName><path>`; `callback(body, { params, query, headers })`
- `she.config.latitude` / `she.config.longitude` — read-only geographic coordinates from daemon config (frozen object)

### Variable system
Topics prefixed with `config.variablePrefix` (default `var`) are tracked in the `var::` store namespace and published retained.

## Web UI

Built with Svelte 5 + Vite 6, served as an SPA from `dist/web/`. Build: `npm run build:web`.

Tabs (in nav order): **Scripts** → **MQTT** → **Matter** → **DB** → **Logs** → **Config**

### HTTP API (all under `/she/*`, Bearer token auth via `apiKey`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/she/scripts` | List `.js` files `[{path, size, mtime}]` |
| GET | `/she/scripts/:path` | Read file `{path, content}` |
| PUT | `/she/scripts/:path` | Write file `{content}` |
| DELETE | `/she/scripts/:path` | Delete file |
| POST | `/she/scripts/:path/rename` | Rename `{newPath}` |
| GET | `/she/db/docs` | List document IDs |
| GET | `/she/db/docs/:id` | Get document |
| PUT | `/she/db/docs/:id` | Create/overwrite document |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hobbyquaker/she](https://github.com/hobbyquaker/she) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
